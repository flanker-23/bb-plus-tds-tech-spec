# BB+ TDS Integration - Architecture Recommendation

> **Goal:** Minimal invasion to Payouts, domain separation, works for all channels (UI, Bulk, API)

---

## 0. Public API Context (Important)

### Current Public API: `POST /v1/payouts`

Based on the public Razorpay documentation, the payout API does **NOT** expose a `tds` field publicly:

```bash
# Current documented public API
curl -u <KEY>:<SECRET> \
-X POST https://api.razorpay.com/v1/payouts \
-H "Content-Type: application/json" \
-d '{
  "account_number": "7878780080316316",
  "fund_account_id": "fa_00000000000001",
  "amount": 1000000,
  "currency": "INR",
  "mode": "IMPS",
  "purpose": "refund",
  "queue_if_low_balance": true,
  "reference_id": "Acme Transaction ID 12345",
  "narration": "Acme Corp Fund Transfer",
  "notes": { ... }
}'
```

**Key observations:**
- `tds` field is **internal** (used by Vendor-Payments, not publicly documented)
- Public entity does not return TDS fields
- Bulk payouts use **Dashboard CSV upload** (not API)
- Internal API is `/v2/payouts` (used by Dashboard/BFF)

### Implication for BB+ TDS

| Aspect | Public API (`/v1/payouts`) | Internal API (`/v2/payouts`) |
|--------|---------------------------|------------------------------|
| Consumer | External merchants | Dashboard, BFF, Internal services |
| TDS field | Not documented | Exists (Vendor-Payments) |
| BB+ TDS | Should NOT be exposed publicly (initially) | Extend for BB+ |
| Amount | Merchant sends net amount | Can send gross (if BB+ flow) |

**Recommendation:** BB+ TDS is a **Dashboard/BB+ feature first**. Use internal `/v2/payouts` API. Public API exposure can be a future enhancement.

---

## 1. Analysis of Tech Spec Approaches

### 1.1 Approach 1 (Spec Recommended): Payouts as Orchestrator

**Flow:**
```
UI/FE → Payouts (with TDS inputs) → S2S call to Tax-Compliance → Compute → Create Payout
```

**Problems:**

| Issue | Impact | Severity |
|-------|--------|----------|
| **S2S call in critical path** | Adds 50-200ms latency to every BB+ payout | High |
| **Failure coupling** | If Tax-Compliance down, payouts fail | High |
| **Domain leakage** | Payouts understands TDS computation logic | Medium |
| **Complex changes** | New client, structs, amount overwriting | Medium |
| **Amount mutation** | `params.Amount` overwritten from gross to net | High Risk |
| **Testing complexity** | Need to mock Tax-Compliance in all payout tests | Medium |

### 1.2 Approach 2 (Spec Alternative): Tax-Compliance as Orchestrator

**Flow:**
```
UI/FE → Tax-Compliance → Create TDS Payment → Call Payouts → Return combined response
```

**Problems:**

| Issue | Impact | Severity |
|-------|--------|----------|
| **Domain leakage** | Tax-Compliance becomes payout orchestrator | High |
| **Complex for bulk** | Two-phase workflow, new bulk TDS API | High |
| **Tight coupling** | Tax-Compliance depends on Payouts API internals | High |
| **Failure handling** | TDS Payment created but payout fails = orphan | Medium |

---

## 2. Recommended Approach: Pre-Compute Pattern (Minimal Invasion)

### 2.1 Core Principle

**"Keep Payouts dumb about TDS computation"**

- Payouts should NOT call Tax-Compliance during payout creation
- Payouts should receive **pre-computed** TDS values (same as Vendor-Payments today)
- The only new responsibility: route to correct Kafka topic based on `source`

### 2.2 Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    PRE-COMPUTE PATTERN (RECOMMENDED)                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  PHASE 1: COMPUTE (Before Payout)                                               │
│  ════════════════════════════════                                               │
│                                                                                  │
│  ┌──────────┐     POST /v1/tds/compute      ┌────────────────┐                  │
│  │  UI/FE   │ ────────────────────────────▶ │ TAX-COMPLIANCE │                  │
│  │  or BFF  │                               │                │                  │
│  │  or Bulk │ ◀──────────────────────────── │  Computes TDS  │                  │
│  │  Service │     { tds_amount, breakdown } │  Checks LDC    │                  │
│  └────┬─────┘                               └────────────────┘                  │
│       │                                                                          │
│       │ Now has: gross_amount, net_amount, tds_amount, category_id, breakdown   │
│       │                                                                          │
│  PHASE 2: CREATE PAYOUT (Payouts unchanged)                                     │
│  ══════════════════════════════════════════                                     │
│       │                                                                          │
│       │     POST /v2/payouts                                                     │
│       │     {                                                                    │
│       │       "amount": 900000,              // NET amount (computed)            │
│       │       "tds": {                                                           │
│       │         "category_id": 14,           // From compute                     │
│       │         "amount": 100000,            // From compute                     │
│       │         "source": "bb_plus",         // NEW: routing indicator           │
│       │         "gross_amount": 1000000,     // NEW: for audit                   │
│       │         "breakdown": { ... }         // NEW: for Kafka message           │
│       │       }                                                                  │
│       │     }                                                                    │
│       ▼                                                                          │
│  ┌──────────────────┐                                                           │
│  │     PAYOUTS      │  No S2S call!                                             │
│  │                  │  No amount mutation!                                       │
│  │  - Validate      │  Just stores what it receives                             │
│  │  - Create payout │                                                           │
│  │  - Store TDS     │                                                           │
│  └────────┬─────────┘                                                           │
│           │                                                                      │
│  PHASE 3: ASYNC TDS ENTITY (On terminal state)                                  │
│  ═════════════════════════════════════════════                                  │
│           │                                                                      │
│           │ Payout reaches PROCESSED/REVERSED                                    │
│           ▼                                                                      │
│  ┌──────────────────┐                                                           │
│  │ ProcessTdsFor    │                                                           │
│  │ Payout()         │                                                           │
│  │                  │                                                           │
│  │ if source ==     │     ┌──────────────────────┐                              │
│  │   "bb_plus":     │────▶│ add-tds-entry-bb-plus│──▶ Tax-Compliance            │
│  │ else:            │     └──────────────────────┘                              │
│  │   (existing)     │────▶│ add-tds-entry        │──▶ Vendor-Payments           │
│  └──────────────────┘     └──────────────────────┘                              │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Changes Required

| Service | Change | Effort | Risk |
|---------|--------|--------|------|
| **Payouts** | Extend `TDSDetails` with `source`, `gross_amount`, `breakdown` | Low | Low |
| **Payouts** | Topic routing in `ProcessTdsForPayout()` | Low | Low |
| **Payouts** | Store BB+ fields in `additional_info` | Low | Low |
| **Tax-Compliance** | Build `/v1/tds/compute` endpoint | Medium | None to Payouts |
| **Tax-Compliance** | Consume `add-tds-entry-bb-plus` topic | Medium | None to Payouts |
| **UI/BFF** | Call compute before payout | Medium | None to Payouts |
| **Bulk Service** | Call compute per row before creating payouts | Medium | None to Payouts |

---

## 3. Detailed Payouts Changes (Minimal)

### 3.1 Extend TDSDetails (Only Add Fields, No Logic Change)

**File:** `internal/app/dtos/payoutDetails.go`

```go
type TDSDetails struct {
    // ── Existing fields (unchanged, already optional) ──
    TDSCategoryID nulls.UInt32  `json:"category_id" binding:"omitempty"`
    TDSAmount     nulls.Float64 `json:"amount" binding:"omitempty"`

    // ── NEW: BB+ fields (all optional, pre-computed by caller) ──
    Source      string         `json:"source" binding:"omitempty"`       // "bb_plus" or empty
    GrossAmount nulls.Int64    `json:"gross_amount" binding:"omitempty"` // For audit trail
    Breakdown   *TDSBreakdown  `json:"breakdown" binding:"omitempty"`    // Full compute result
}

// TDSBreakdown - stored for audit and Kafka message
type TDSBreakdown struct {
    BasicTaxAmount  float64 `json:"basic_tax_amount"`
    Surcharge       float64 `json:"surcharge"`
    Cess            float64 `json:"cess"`
    EffectiveRate   float64 `json:"effective_rate"`   // Percentage
    IsLdcApplied    bool    `json:"is_ldc_applied"`
    LdcRate         float64 `json:"ldc_rate,omitempty"`
    CategoryCode    string  `json:"category_code"`    // "194J"
    MajorHead       int     `json:"major_head"`
    MinorHead       int     `json:"minor_head"`
}

// IsBBPlusFlow returns true if this is a BB+ TDS payout
func (t *TDSDetails) IsBBPlusFlow() bool {
    return t.Source == "bb_plus"
}
```

### 3.2 Extend AdditionalInfoStruct (Storage Only)

**File:** `internal/app/payoutDetails/additional_info_model.go`

```go
type AdditionalInfoStruct struct {
    // ── Existing fields (unchanged) ──
    SubtotalAmount *nulls.Float64  `json:"subtotal_amount,omitempty"`
    TDSAmount      *nulls.Float64  `json:"tds_amount,omitempty"`
    Attachments    []Attachment    `json:"attachments,omitempty"`
    MerchantDetail *MerchantDetail `json:"merchant_detail,omitempty"`

    // ── NEW: BB+ fields (just storage, no logic) ──
    TDSSource      string         `json:"tds_source,omitempty"`      // "bb_plus"
    TDSGrossAmount *nulls.Int64   `json:"tds_gross_amount,omitempty"`
    TDSBreakdown   *TDSBreakdown  `json:"tds_breakdown,omitempty"`
}
```

### 3.3 Topic Routing (Only Change to Core Logic)

**File:** `internal/app/payouts/core.go` - `ProcessTdsForPayout()`

```go
const (
    TdsTopic       = "add-tds-entry"          // Existing - Vendor-Payments
    TdsTopicBBPlus = "add-tds-entry-bb-plus"  // NEW - Tax-Compliance
)

func (c Core) ProcessTdsForPayout(ctx context.Context, payout *Payout, oldStatus string) errors.IError {
    if MoneyTransferredStatus[payout.GetStatus()] == true {
        payoutDetails := payout.GetPayoutDetails(ctx)
        additionalInfo := payoutDetails.GetAdditionalInfo()

        // Determine topic based on source
        topic := TdsTopic  // Default: Vendor-Payments
        if additionalInfo.GetTDSSource() == "bb_plus" {
            topic = TdsTopicBBPlus  // BB+: Tax-Compliance
        }

        // Build message (extend for BB+ if needed)
        message := c.buildTdsMessage(ctx, payout, payoutDetails, additionalInfo)

        eventClient := provider.GetEventClient(ctx)
        eventClient.PublishEvent(message, topic)
    }
    return nil
}
```

### 3.4 Store BB+ Fields in SetPayoutDetailsInPayout

**File:** `internal/app/payouts/processor/base.go`

```go
func SetPayoutDetailsInPayout(ctx context.Context, params *dtos.FundAccountPayoutRequest, payout *payouts.Payout) {
    // ... existing code ...

    payoutDetailsObj.SetTDSCategoryID(params.TDS.TDSCategoryID)
    payoutDetailsObj.SetAdditionalInfo(
        params.SubtotalAmount,
        params.TDS.TDSAmount,
        attachmentsInterfaceSlice,
        &params.RemitterDetails,
        // NEW: BB+ fields (added to SetAdditionalInfo signature)
        params.TDS.Source,
        params.TDS.GrossAmount,
        params.TDS.Breakdown,
    )

    // ... rest unchanged ...
}
```

---

## 4. What Each Service Does (Domain Separation)

| Service | Responsibility | Does NOT Do |
|---------|----------------|-------------|
| **Tax-Compliance** | Compute TDS, LDC lookup, rate management, TDS Payment records, filing | Create payouts |
| **Payouts** | Create payouts, store TDS data, route to Kafka | Compute TDS |
| **Vendor-Payments** | Aggregate TDS, pay to GOI (existing flow) | BB+ TDS |
| **UI/BFF** | Orchestrate: compute → display → confirm → create payout | Compute TDS, create TDS records |
| **Bulk Service** | Parse CSV, compute per row, create payouts | Store TDS records |

---

## 5. Flow by Channel

### 5.1 UI Single Payout

```
1. User enters vendor details + amount (₹10,000)
2. User selects TDS category (194J)
3. UI calls: POST /v1/tds/compute → Tax-Compliance
4. Tax-Compliance returns: { tds_amount: 1000, net: 9000, breakdown: {...} }
5. UI displays preview: "Net payout: ₹9,000, TDS: ₹1,000"
6. User confirms
7. UI calls: POST /v2/payouts → Payouts
   {
     amount: 900000,  // NET
     tds: { category_id: 14, amount: 100000, source: "bb_plus", gross_amount: 1000000, breakdown: {...} }
   }
8. Payouts creates payout, stores TDS
9. On PROCESSED: Payouts publishes to "add-tds-entry-bb-plus"
10. Tax-Compliance consumes, creates TDS Payment record
```

### 5.2 Bulk Payouts (Dashboard CSV Upload)

Per public docs, bulk payouts use **Dashboard CSV upload** with templates.

**Current bulk flow:**
1. User downloads CSV template from Dashboard
2. User fills: fund_account_id, amount, mode, narration, etc.
3. User uploads CSV
4. Dashboard validates and creates batch
5. User confirms with OTP
6. Batch creates individual payouts

**BB+ TDS bulk flow:**
```
1. User downloads BB+ TDS CSV template (new template)
   - Columns: fund_account_id, gross_amount, tds_category, mode, narration, etc.

2. User fills CSV with gross amounts and TDS categories

3. User uploads CSV to Dashboard

4. Dashboard/BFF processes CSV:
   For each row:
   a. Call POST /v1/tds/compute → Tax-Compliance
   b. Get: net_amount, tds_amount, breakdown

5. Dashboard generates PREVIEW file showing:
   - Gross amount | TDS Category | TDS Amount | Net Payout
   - User downloads and reviews

6. User confirms with OTP

7. Dashboard creates batch, for each row:
   a. Call POST /v2/payouts with pre-computed TDS

8. Each payout follows same flow as single payout
```

**Bulk changes required:**

| Component | Change |
|-----------|--------|
| CSV Template | Add: `gross_amount`, `tds_category` columns |
| Dashboard | Add TDS compute step during upload |
| Preview | Show computed TDS breakdown |
| Batch Creation | Pass pre-computed TDS to /v2/payouts |

**No Payouts service changes for bulk** - uses same /v2/payouts with pre-computed TDS.

### 5.3 API Integration (Merchant)

```
1. Merchant calls: POST /v1/tds/compute (optional, for preview)
2. Merchant calls: POST /v2/payouts
   - Option A: With compute inputs → Fail (we don't support inline compute)
   - Option B: With pre-computed values → Works (same as today)

For BB+ API merchants:
1. Merchant MUST call compute first
2. Then call payouts with computed values
```

---

## 6. Comparison: Spec Approach 1 vs Recommended

| Aspect | Spec Approach 1 | Recommended (Pre-Compute) |
|--------|-----------------|---------------------------|
| **S2S call in payout creation** | Yes (blocking) | No |
| **Latency added to payout API** | 50-200ms | 0ms |
| **Payouts fails if Tax-Compliance down** | Yes | No |
| **Amount mutation in Payouts** | Yes (gross → net) | No |
| **Payouts code changes** | High (client, compute logic) | Low (just storage + routing) |
| **Domain leakage** | Payouts knows TDS logic | None |
| **Testing complexity** | High (mock Tax-Compliance) | Low |
| **Works for Bulk** | Yes (but complex) | Yes (simple) |
| **Works for API** | Yes | Yes |

---

## 7. Kafka Message Enhancement

### 7.1 Existing Message (Vendor-Payments)

```json
{
  "data": {
    "entity_id": "pout_ABC123",
    "entity_type": "payout",
    "entity_status": "processed",
    "merchant_id": "merch_XYZ",
    "tds_category_id": 14,
    "tds_amount": 100000
  },
  "attributes": { "mode": "LIVE" }
}
```

### 7.2 BB+ Message (Tax-Compliance)

```json
{
  "data": {
    "entity_id": "pout_ABC123",
    "entity_type": "payout",
    "entity_status": "processed",
    "merchant_id": "merch_XYZ",
    "tds_category_id": 14,
    "tds_amount": 100000,
    "source": "bb_plus",
    "gross_amount": 1000000,
    "breakdown": {
      "basic_tax_amount": 90000,
      "surcharge": 0,
      "cess": 10000,
      "effective_rate": 10.0,
      "is_ldc_applied": false,
      "category_code": "194J",
      "major_head": 21,
      "minor_head": 200
    }
  },
  "attributes": { "mode": "LIVE" }
}
```

---

## 8. Tax-Compliance Changes Required

### 8.1 Build `/v1/tds/compute` Endpoint

```protobuf
// Request
message ComputeTDSRequest {
  string merchant_id = 1;
  string contact_id = 2;
  string category = 3;           // "194J"
  int64 vendor_payment_amount = 4;  // Gross amount in paise
  bool is_ldc_applicable = 5;
}

// Response
message ComputeTDSResponse {
  int64 tds_amount = 1;          // Total TDS in paise
  int64 net_amount = 2;          // Net payout amount in paise
  int64 basic_tax_amount = 3;
  int64 surcharge = 4;
  int64 cess = 5;
  float effective_rate = 6;      // Percentage
  bool is_ldc_applied = 7;
  float ldc_rate = 8;
  int32 category_id = 9;         // Numeric category ID
  string category_code = 10;     // "194J"
  int32 major_head = 11;
  int32 minor_head = 12;
}
```

### 8.2 Consume `add-tds-entry-bb-plus` Topic

- Create Kafka consumer
- Parse BB+ message format
- Create TDS Payment record with breakdown
- Idempotency via `entity_id` (payout_id)

---

## 9. Summary

### Payouts Changes (Minimal)

1. **Add fields to `TDSDetails`**: `source`, `gross_amount`, `breakdown`
2. **Add fields to `AdditionalInfoStruct`**: same BB+ fields
3. **Add constant**: `TdsTopicBBPlus`
4. **Modify `ProcessTdsForPayout()`**: Route based on source
5. **Modify `SetPayoutDetailsInPayout()`**: Store BB+ fields

**No changes to:**
- `CreatePayoutToFundAccountV2()` core logic
- Amount handling
- Validation logic
- Processor flow
- Any S2S calls

### Tax-Compliance Changes

1. Build `/v1/tds/compute` endpoint
2. Implement TDS rate lookup (category → rate)
3. Implement LDC application logic
4. Build Kafka consumer for `add-tds-entry-bb-plus`
5. Create TDS Payment records from Kafka

### UI/BFF Changes

1. Call `/v1/tds/compute` before displaying preview
2. Pass computed values to `/v2/payouts`

### Bulk Service Changes

1. Call `/v1/tds/compute` per row during parsing
2. Generate preview with computed values
3. Pass computed values when creating payouts

---

## 10. Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Compute and payout are separate calls | Compute result has short TTL; re-compute if stale |
| LDC expires between compute and payout | Kafka consumer validates LDC; flags if expired |
| Merchant doesn't call compute first | Validate: if source=bb_plus, require breakdown |
| Rate changes between compute and payout | Store compute timestamp; audit trail exists |

---

*End of Recommendation*
