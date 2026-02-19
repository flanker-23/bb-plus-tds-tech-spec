# Payouts Codebase Understanding - TDS Integration Reference

> **Purpose:** Reference document for understanding the payouts codebase and BB+ TDS integration requirements.
> **Last Updated:** February 2026
> **Related Spec:** `TDS_Payout_Integration_Tech_Spec.html` (Version 6.0)
> **Public Docs:** `/payouts doc scraped/` folder

---

## 0. Public vs Internal API Context

### Public API: `POST /v1/payouts`

The **public** Razorpay Payouts API does NOT expose TDS fields:

```json
// Public API - documented at razorpay.com/docs/api/x/payouts/
POST /v1/payouts
{
  "account_number": "7878780080316316",
  "fund_account_id": "fa_00000000000001",
  "amount": 1000000,              // NET amount
  "currency": "INR",
  "mode": "IMPS",
  "purpose": "refund",
  "queue_if_low_balance": true,
  "reference_id": "...",
  "narration": "...",
  "notes": { }
  // NO tds field documented
}
```

### Internal API: `POST /v2/payouts`

The **internal** API (used by Dashboard, BFF) has the `tds` field:

```json
// Internal API - used by Dashboard/BFF
POST /v2/payouts
{
  "amount": 1000000,
  "fund_account_id": "fa_xxx",
  "mode": "IMPS",
  "purpose": "vendor_payment",
  "tds": {                        // INTERNAL FIELD
    "category_id": 14,
    "amount": 100000
  },
  "subtotal_amount": 1100000,
  "remitter_details": { },
  "attachments": [ ]
}
```

### Bulk Payouts

Per public docs, bulk payouts use **Dashboard CSV upload**:
- User downloads template
- Fills: fund_account_id, amount, mode, narration
- Uploads to Dashboard
- Dashboard creates batch → individual payouts

**BB+ will extend:** CSV template with `gross_amount`, `tds_category` columns

---

## Table of Contents

1. [Repository Structure](#1-repository-structure)
2. [V2 Payouts Endpoint Flow](#2-v2-payouts-endpoint-flow)
3. [Request DTOs](#3-request-dtos)
4. [Current TDS Implementation](#4-current-tds-implementation)
5. [Key Files for BB+ TDS Integration](#5-key-files-for-bb-tds-integration)
6. [Kafka Event Publishing](#6-kafka-event-publishing)
7. [Gap Analysis: Current vs BB+ Requirements](#7-gap-analysis-current-vs-bb-requirements)
8. [Implementation Notes](#8-implementation-notes)

---

## 1. Repository Structure

**Root:** `/Users/prashant.chauhan/go/src/github.com/razorpay/payouts`

```
payouts/
├── cmd/                           # Entry points
│   ├── api/main.go               # API service bootstrap
│   ├── kafkaConsumers/           # Kafka consumer services
│   └── workers/                  # Background workers
├── internal/                      # Core business logic
│   ├── app/                      # Domain layer
│   │   ├── dtos/                 # Data Transfer Objects
│   │   ├── payouts/              # Payout core logic
│   │   ├── payoutDetails/        # PayoutDetails model
│   │   └── interfaces/           # Interface definitions
│   ├── controllers/              # HTTP handlers
│   ├── routing/                  # API routes
│   ├── auth/                     # Authentication
│   └── provider/                 # Service container
├── pkg/                          # Shared packages
│   └── events/                   # Kafka event publishing
└── config/                       # Configuration files
```

---

## 2. V2 Payouts Endpoint Flow

### Request Path

```
POST /v2/payouts
       ↓
┌──────────────────────────────────────────────────────────────┐
│ Controller: FundAccountPayout()                              │
│ File: internal/controllers/payoutController.go:187           │
│ - Builds FundAccountPayoutRequestV2 from JSON                │
│ - Validates request                                          │
│ - Calls payoutService.CreateFundAccountPayoutV2()            │
└──────────────────────────────────────────────────────────────┘
       ↓
┌──────────────────────────────────────────────────────────────┐
│ Core: CreatePayoutToFundAccountV2()                          │
│ File: internal/app/payouts/core.go:469                       │
│ - Validates auth (internal app only)                         │
│ - Marshals V2 DTO → V1 DTO                                   │
│ - Sets merchantId from auth context                          │
│ - Pre-evaluates Splitz experiments                           │
│ - Fetches banking account & merchant config                  │
│ - Creates contact/fund account via CFA (if composite)        │
│ - Validates payout is allowed                                │
│ - Calls processor.CreatePayout()                             │
└──────────────────────────────────────────────────────────────┘
       ↓
┌──────────────────────────────────────────────────────────────┐
│ Processor: BasePayoutProcessor.CreatePayout()                │
│ File: internal/app/payouts/processor/base.go                 │
│ - Creates Payout model                                       │
│ - CheckIfPayoutDetailsIsNeeded() → determines if needed      │
│ - SetPayoutDetailsInPayout() → SETS TDS HERE                 │
│ - Validates payout                                           │
│ - Checks shield rules & workflows                            │
│ - Creates payout in DB                                       │
│ - Pushes to queue or processes sync                          │
└──────────────────────────────────────────────────────────────┘
       ↓
┌──────────────────────────────────────────────────────────────┐
│ On Status Update (PROCESSED/REVERSED)                        │
│ File: internal/app/payouts/fts_transfer_status_webhook.go    │
│ - Calls Core.ProcessTdsForPayout()                           │
│ - Publishes TDS to Kafka topic "add-tds-entry"               │
└──────────────────────────────────────────────────────────────┘
```

### Key Function: CreatePayoutToFundAccountV2

**Location:** `internal/app/payouts/core.go:469`

```go
func (c Core) CreatePayoutToFundAccountV2(ctx context.Context, input interface{}) (*Payout, errors.IError) {
    // 1. Auth validation
    if err = auth.IsAllowedInternalApp(ctx); err != nil { return nil, err }

    // 2. Marshal V2 → V1 DTO
    params := input.(*dtos.FundAccountPayoutRequestV2)
    jsonBytes, _ := json.Marshal(*params)
    fundAccountPayoutRequest := &dtos.FundAccountPayoutRequest{}
    json.Unmarshal(jsonBytes, fundAccountPayoutRequest)

    // 3. Set merchant ID from context
    merchantId, _ := auth.GetMerchantIdFromContext(ctx)
    params.MerchantID = merchantId
    fundAccountPayoutRequest.MerchantID = merchantId

    // 4. Fetch dependencies
    bankingAccount, merchantConfig, err := c.FetchPayoutDependencyBankingAccountAndMerchantConfig(ctx, *params)

    // 5. Composite payout handling (create contact + fund account)
    if !utils.IsEmpty(params.FundAccount) {
        fundAccount, err = c.ContactAndFundAccountCreationOnCFA(ctx, params, merchantConfig)
        params.FundAccountID = fundAccount.GetID()
    }

    // 6. Validate input
    err = c.ValidatePayoutCreateInput(ctx, params.Amount, merchantConfig)

    // ═══════════════════════════════════════════════════════════════════
    // BB+ TDS COMPUTE CALL INSERTION POINT (between lines 593-597)
    // ═══════════════════════════════════════════════════════════════════

    // 7. Create payout via processor
    payout, err := c.processorFactory.GetProcessor(...).CreatePayout(fundAccountPayoutRequest)

    return payout, err
}
```

---

## 3. Request DTOs

### FundAccountPayoutRequestV2

**File:** `internal/app/dtos/payout_create.go:19`

```go
type FundAccountPayoutRequestV2 struct {
    // Required fields
    Purpose       string `json:"purpose" binding:"required,max=30"`
    Amount        int64  `json:"amount" binding:"required"`
    Currency      string `json:"currency" binding:"required,oneof=INR MYR,len=3"`
    FundAccountID string `json:"fund_account_id" binding:"required_without=FundAccount"`

    // Optional - Composite payout
    FundAccount *FundAccountCompositePayout `json:"fund_account" binding:"omitempty,required_without=FundAccountID"`

    // Transfer details
    Mode       string       `json:"mode"`
    Notes      map[string]interface{} `json:"notes"`
    Narration  nulls.String `json:"narration" binding:"omitempty,alphanum,max=30"`

    // Reference & scheduling
    ReferenceID   nulls.String `json:"reference_id" binding:"omitempty,max=40"`
    BatchID       nulls.String `json:"batch_id" binding:"omitempty"`
    ScheduledAt   nulls.Int64  `json:"scheduled_at" binding:"omitempty"`

    // Workflow control
    QueueIFLowBalance                bool `json:"queue_if_low_balance"`
    SkipWorkflow                     bool `json:"skip_workflow" binding:"omitempty"`
    EnableWorkflowForInternalContact bool `json:"enable_workflow_for_internal_contact" binding:"omitempty"`

    // TDS & attachments (BB+ relevant)
    TDS             TDSDetails          `json:"tds" binding:"omitempty"`
    Attachments     []AttachmentDetails `json:"attachments" binding:"omitempty,dive"`
    RemitterDetails RemitterDetails     `json:"remitter_details" binding:"omitempty,dive"`
    SubtotalAmount  nulls.Float64       `json:"subtotal_amount" binding:"omitempty"`

    // Internal fields
    MerchantID    string       `json:"merchant_id"`       // Set from auth context
    AccountNumber string       `json:"account_number"`    // Merchant's banking account
    BalanceID     string       `json:"-"`                 // Internal use
    Origin        string       `json:"origin"`
    FeeType       nulls.String `json:"fee_type"`
    PayoutLinkID  string       `json:"payout_link_id"`
    IdempotencyKey nulls.String `json:"idempotency_key"`
    SourceDetails []SourceDetail `json:"source_details" binding:"omitempty,dive"`
}
```

### TDSDetails (Current)

**File:** `internal/app/dtos/payoutDetails.go:7`

```go
type TDSDetails struct {
    TDSCategoryID nulls.UInt32  `json:"category_id" binding:"required"`
    TDSAmount     nulls.Float64 `json:"amount" binding:"required"`
}

// Getters
func (tds *TDSDetails) GetTDSCategoryID() nulls.UInt32
func (tds *TDSDetails) GetTDSAmount() nulls.Float64

// Setters
func (tds *TDSDetails) SetTDSCategoryID(id nulls.UInt32)
func (tds *TDSDetails) SetTDSAmount(amount nulls.Float64)
```

### RemitterDetails

**File:** `internal/app/dtos/payoutDetails.go:18`

```go
type RemitterDetails struct {
    MerchantName    string `json:"merchant_name" binding:"omitempty"`
    MerchantPan     string `json:"merchant_pan" binding:"omitempty"`
    MerchantAddress string `json:"merchant_address" binding:"omitempty"`
}
```

### AttachmentDetails

**File:** `internal/app/dtos/payoutDetails.go:12`

```go
type AttachmentDetails struct {
    FileID   string `json:"file_id" binding:"required"`
    FileName string `json:"file_name" binding:"required"`
    FileHash string `json:"file_hash" binding:"omitempty"`
}
```

---

## 4. Current TDS Implementation

### Storage: PayoutDetails Model

**File:** `internal/app/payoutDetails/model.go:22`

```go
type PayoutDetails struct {
    spine.Model
    QueueIfLowBalanceFlag int64
    PayoutID              string
    TDSCategoryID         nulls.UInt32                              // DB column
    TaxPaymentID          nulls.String                              // DB column
    AdditionalInfo        datatypes.JSONType[AdditionalInfoStruct]  // JSON column
    BeneficiaryBankCode   string
}
```

### Storage: AdditionalInfoStruct

**File:** `internal/app/payoutDetails/additional_info_model.go:14`

```go
type AdditionalInfoStruct struct {
    SubtotalAmount *nulls.Float64  `json:"subtotal_amount,omitempty"`
    TDSAmount      *nulls.Float64  `json:"tds_amount,omitempty"`
    Attachments    []Attachment    `json:"attachments,omitempty"`
    MerchantDetail *MerchantDetail `json:"merchant_detail,omitempty"`
}
```

### Where TDS is Set

**File:** `internal/app/payouts/processor/base.go:404`

```go
func SetPayoutDetailsInPayout(ctx context.Context, params *dtos.FundAccountPayoutRequest, payout *payouts.Payout) {
    payoutDetailsObj := &payoutDetails.PayoutDetails{
        QueueIfLowBalanceFlag: ...,
        PayoutID: payout.GetID(),
    }

    // Set TDS fields
    payoutDetailsObj.SetTDSCategoryID(params.TDS.TDSCategoryID)           // line 423
    payoutDetailsObj.SetAdditionalInfo(
        params.SubtotalAmount,
        params.TDS.TDSAmount,      // TDS amount goes into additional_info JSON
        attachmentsInterfaceSlice,
        &params.RemitterDetails,
    )                                                                       // line 424

    payout.PayoutDetails = *payoutDetailsObj
}
```

### Check if PayoutDetails Needed

**File:** `internal/app/payouts/processor/base.go:434`

```go
func CheckIfPayoutDetailsIsNeeded(params *dtos.FundAccountPayoutRequest, beneficiaryAccountType string) bool {
    isQueueIfLowBalanceSet := !utils.IsEmpty(params.QueueIFLowBalance)
    isTDSSet := !utils.IsEmpty(params.TDS)                    // ← TDS triggers PayoutDetails
    isAttachmentsSet := !utils.IsEmpty(params.Attachments)
    isSubtotalAmountSet := !utils.IsEmpty(params.SubtotalAmount)
    isRemitterDetailsSet := !utils.IsEmpty(params.RemitterDetails)
    isBeneficiaryOfBankAccountType := ...

    return isQueueIfLowBalanceSet || isTDSSet || isAttachmentsSet || ...
}
```

---

## 5. Key Files for BB+ TDS Integration

| File | Purpose | Changes Needed |
|------|---------|----------------|
| `internal/app/dtos/payoutDetails.go` | TDSDetails struct | Add BB+ compute input fields |
| `internal/app/dtos/payout_create.go` | V2 Request DTO | Relax `Amount` binding to `omitempty` |
| `internal/app/payoutDetails/additional_info_model.go` | AdditionalInfoStruct | Add `BBPlusTDSInput`, `BBPlusTDSComputed` |
| `internal/app/payouts/core.go` | Core payout logic | Add compute call block, extend TdsData |
| `internal/app/payouts/processor/base.go` | Payout processor | Store BB+ data in SetPayoutDetailsInPayout |
| `internal/app/payouts/constants.go` | Constants | Add `TdsTopicBBPlus` |
| **NEW:** `internal/app/payouts/tds_compute_client.go` | S2S client | HTTP client for Tax-Compliance compute |

---

## 6. Kafka Event Publishing

### Current TDS Message Structure

**File:** `internal/app/payouts/core.go:7839-7856`

```go
type TdsData struct {
    EntityID      string         `json:"entity_id"`        // payout ID
    EntityType    string         `json:"entity_type"`      // "payout"
    EntityStatus  string         `json:"entity_status"`    // "processed"/"reversed"
    MerchantID    string         `json:"merchant_id"`
    TdsCategoryID nulls.UInt32   `json:"tds_category_id"`
    TdsAmount     *nulls.Float64 `json:"tds_amount"`
}

type TdsMessage struct {
    Data       TdsData    `json:"data"`
    Attributes Attributes `json:"attributes"`
}

type Attributes struct {
    Mode string `json:"mode"`  // "LIVE"
}
```

### Current Topic

**File:** `internal/app/payouts/constants.go:40`

```go
const TdsTopic = "add-tds-entry"
```

### ProcessTdsForPayout Function

**File:** `internal/app/payouts/core.go:7858`

```go
func (c Core) ProcessTdsForPayout(ctx context.Context, payout *Payout, oldStatus string) errors.IError {
    // Only publish when payout reaches terminal state
    if MoneyTransferredStatus[payout.GetStatus()] == true {
        payoutDetails := payout.GetPayoutDetails(ctx)
        tdsCategoryId := payoutDetails.GetTDSDetailsPublic().GetTDSCategoryID()
        tdsAmount := payoutDetails.GetAdditionalInfo().GetTDSAmount()

        data := TdsData{
            EntityID:      payout.GetID(),
            EntityType:    payout.EntityName(),
            EntityStatus:  payout.Status,
            MerchantID:    payout.MerchantID,
            TdsCategoryID: tdsCategoryId,
            TdsAmount:     &tdsAmount,
        }

        message := TdsMessage{
            Data:       data,
            Attributes: Attributes{Mode: appConstants.LIVE},
        }

        eventClient := provider.GetEventClient(ctx)
        eventClient.PublishEvent(message, TdsTopic)
    }
    return nil
}
```

### MoneyTransferredStatus

**File:** `internal/app/payouts/status.go:111-114`

```go
var MoneyTransferredStatus = map[string]bool{
    appConstants.StateProcessed: true,
    appConstants.StateReversed:  true,
}
```

---

## 7. Gap Analysis: Current vs BB+ Requirements

| Aspect | Current Implementation | BB+ Requirement |
|--------|------------------------|-----------------|
| **TDS Fields** | `category_id`, `amount` (pre-computed) | `category`, `source`, `is_ldc_applicable`, `contact_id` (compute inputs) |
| **Amount Handling** | Caller sends net amount | Caller sends gross; Payouts derives net after compute |
| **TDS Compute** | None - caller provides values | Payouts calls Tax-Compliance `/v1/tds/compute` (S2S) |
| **Storage** | Basic: `tds_category_id` + `tds_amount` | Extended: `BBPlusTDSInput` + `BBPlusTDSComputed` |
| **Kafka Message** | Minimal TDS data | Full compute result for `tds_payment` creation |
| **Kafka Topic** | `add-tds-entry` (Vendor-Payments) | New: `add-tds-entry-bb-plus` (Tax-Compliance) |
| **S2S Client** | None to Tax-Compliance | New HTTP client with timeout/circuit breaker |

---

## 8. Implementation Notes

### BB+ Detection Logic

Per Tech Spec Section 11.2.1:

```go
// IsBBPlusComputeFlow returns true when the caller sent BB+ compute inputs.
func (t *TDSDetails) IsBBPlusComputeFlow(grossAmount int64) bool {
    return t.Category != "" && grossAmount > 0
}
```

### Input Field Mapping (Reuse Existing Fields)

Per Tech Spec Section 11.0:

| Compute API Input | Extract From | Notes |
|-------------------|--------------|-------|
| `merchant_id` | `auth.GetMerchantIdFromContext(ctx)` | Never from request body |
| `contact_id` | `fundAccountCache.GetContactId()` or `tds.contact_id` | Fund account has contact |
| `category` | `tds.category` | New field in TDSDetails |
| `vendor_payment_amount` | `amount` (top-level) | Caller sends gross |
| `deduction_date` | `time.Now().Unix()` | Auto-generated |
| `source` | `tds.source` | New field in TDSDetails |
| `is_ldc_applicable` | `tds.is_ldc_applicable` | New field in TDSDetails |

### Compute Call Insertion Point

**Location:** `internal/app/payouts/core.go` - inside `CreatePayoutToFundAccountV2()`

Insert **AFTER:** `ValidatePayoutCreateInput()` (line ~593)
Insert **BEFORE:** `processorFactory.GetProcessor(...).CreatePayout()` (line ~598)

### Topic Routing Logic

```go
// In ProcessTdsForPayout()
if source == "bb_plus" {
    eventClient.PublishEvent(message, TdsTopicBBPlus)  // "add-tds-entry-bb-plus"
} else {
    eventClient.PublishEvent(message, TdsTopic)        // "add-tds-entry" (existing)
}
```

---

## Appendix: Related Interfaces

### IPayoutDetails

**File:** `internal/app/interfaces/IPayoutDetails.go`

```go
type IPayoutDetails interface {
    GetPayoutID() string
    GetQueueIfLowBalanceFlag() bool
    GetSubtotalAmount() nulls.Float64
    GetTaxPaymentID() nulls.String
    GetAttachmentsPublic() []IPayoutDetailsAttachment
    GetTDSDetailsPublic() IPayoutDetailsTDS
    GetAdditionalInfo() IPayoutDetailsAdditionalInfo
    SetTDSCategoryID(id nulls.UInt32)
    SetTaxPaymentID(id nulls.String)
    SetAdditionalInfo(...)
}
```

### IPayoutDetailsTDS

```go
type IPayoutDetailsTDS interface {
    GetTDSCategoryID() nulls.UInt32
    GetTDSAmount() nulls.Float64
    SetTDSCategoryID(id nulls.UInt32)
    SetTDSAmount(amt nulls.Float64)
}
```

### IPayoutDetailsAdditionalInfo

```go
type IPayoutDetailsAdditionalInfo interface {
    GetSubtotalAmount() nulls.Float64
    GetTDSAmount() nulls.Float64
    GetAttachments() []IPayoutDetailsAttachment
    GetRemitterDetails() IPayoutDetailsRemitterDetails
    SetSubtotalAmount(amt nulls.Float64)
    SetTDSAmount(amt nulls.Float64)
    SetAttachments(attachments []IPayoutDetailsAttachment)
}
```

---

## 9. Deep Dive: TDS Field Complete Flow

### 9.1 Database Schema: payout_details Table

**Migration File:** `internal/database/migrations/20210902153752_create_payout_details_table.go`

```sql
CREATE TABLE payout_details (
    id char(14) PRIMARY KEY,
    payout_id char(14) NOT NULL,                    -- FK to payouts table
    queue_if_low_balance_flag tinyint DEFAULT '0',
    tds_category_id int unsigned DEFAULT NULL,      -- TDS category (e.g., 14 for 194J)
    tax_payment_id varchar(30) DEFAULT NULL,        -- Reference to tax_payments table
    additional_info json DEFAULT NULL,              -- JSON blob for extended data
    created_at int NOT NULL,
    updated_at int NOT NULL
);
```

**Key Columns for TDS:**
- `tds_category_id` - Direct column storing the TDS category ID
- `additional_info` - JSON column storing `tds_amount` and other extended data
- `tax_payment_id` - Reference to Tax-Compliance's `tds_payments` entity (currently unused for BB+)

### 9.2 Entity Relationships

```
┌─────────────────────┐         1:1         ┌─────────────────────┐
│       payouts       │◄────────────────────│   payout_details    │
├─────────────────────┤                     ├─────────────────────┤
│ id (PK)             │                     │ id (PK)             │
│ merchant_id         │                     │ payout_id (FK)      │
│ amount              │                     │ tds_category_id     │
│ status              │                     │ tax_payment_id      │
│ ...                 │                     │ additional_info     │
└─────────────────────┘                     └─────────────────────┘
                                                     │
                                            JSON Column Structure:
                                            {
                                              "subtotal_amount": 10000.00,
                                              "tds_amount": 1000.00,
                                              "attachments": [...],
                                              "merchant_detail": {...}
                                            }
```

### 9.3 TDS Data Flow: Request → Storage → Kafka

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: REQUEST ARRIVAL                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  POST /v2/payouts                                                            │
│  {                                                                           │
│    "amount": 900000,           // Net amount (₹9,000) - caller computed      │
│    "fund_account_id": "fa_xxx",                                              │
│    "tds": {                                                                  │
│      "category_id": 14,        // Pre-computed: 194J category                │
│      "amount": 100000          // Pre-computed: TDS amount (₹1,000)          │
│    },                                                                        │
│    "subtotal_amount": 1000000  // Gross amount (₹10,000)                     │
│  }                                                                           │
│                                                                              │
│  File: internal/app/dtos/payout_create.go:19                                 │
│  Struct: FundAccountPayoutRequestV2                                          │
│  Field: TDS TDSDetails `json:"tds" binding:"omitempty"`                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 2: CONTROLLER VALIDATION                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  File: internal/controllers/payoutController.go:187                          │
│  Function: FundAccountPayout()                                               │
│                                                                              │
│  createParams := dtos.GetRequestBuilder(enum.PayoutFundAccountRequestV2)     │
│  err := validator.GetValidationError(createParams.Build(ctx))                │
│                                                                              │
│  // TDS fields validated via gin binding tags:                               │
│  // - category_id: binding:"required" (when tds is present)                  │
│  // - amount: binding:"required" (when tds is present)                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 3: CORE PROCESSING                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  File: internal/app/payouts/core.go:469                                      │
│  Function: CreatePayoutToFundAccountV2()                                     │
│                                                                              │
│  // V2 DTO marshaled to V1 DTO (TDS fields preserved)                        │
│  jsonBytes, _ := json.Marshal(*params)                                       │
│  fundAccountPayoutRequest := &dtos.FundAccountPayoutRequest{}                │
│  json.Unmarshal(jsonBytes, fundAccountPayoutRequest)                         │
│                                                                              │
│  // TDS flows through unchanged to processor                                 │
│  payout, err := c.processorFactory.GetProcessor(...).                        │
│      CreatePayout(fundAccountPayoutRequest)                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 4: PROCESSOR - PAYOUT CREATION                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  File: internal/app/payouts/processor/base.go:83                             │
│  Function: CreatePayout()                                                    │
│                                                                              │
│  // Step 1: Create Payout model (line 94-111)                                │
│  payout := &payouts.Payout{                                                  │
│      MerchantID:    params.MerchantID,                                       │
│      Amount:        params.Amount,        // Net amount: 900000              │
│      Status:        "create_request_submitted",                              │
│      ...                                                                     │
│  }                                                                           │
│                                                                              │
│  // Step 2: Check if PayoutDetails needed (line 125)                         │
│  if CheckIfPayoutDetailsIsNeeded(params, payout.FundAccount.GetAccountType())│
│                                                                              │
│  // Step 3: Set PayoutDetails with TDS (line 126, 159)                       │
│  SetPayoutDetailsInPayout(b.ctx, params, payout)                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 5: SET PAYOUT DETAILS (TDS STORAGE)                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  File: internal/app/payouts/processor/base.go:404                            │
│  Function: SetPayoutDetailsInPayout()                                        │
│                                                                              │
│  payoutDetailsObj := &payoutDetails.PayoutDetails{                           │
│      PayoutID: payout.GetID(),                    // Links to payout         │
│  }                                                                           │
│                                                                              │
│  // Line 423: Set TDS Category ID (goes to DB column)                        │
│  payoutDetailsObj.SetTDSCategoryID(params.TDS.TDSCategoryID)                 │
│                                                                              │
│  // Line 424: Set Additional Info (goes to JSON column)                      │
│  payoutDetailsObj.SetAdditionalInfo(                                         │
│      params.SubtotalAmount,           // subtotal_amount in JSON             │
│      params.TDS.TDSAmount,            // tds_amount in JSON                  │
│      attachmentsInterfaceSlice,       // attachments in JSON                 │
│      &params.RemitterDetails,         // merchant_detail in JSON             │
│  )                                                                           │
│                                                                              │
│  payout.PayoutDetails = *payoutDetailsObj                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 6: DATABASE PERSISTENCE                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Two rows created in single transaction:                                     │
│                                                                              │
│  TABLE: payouts                                                              │
│  ┌────────────────────────────────────────────────────────────┐              │
│  │ id: "pout_ABC123"                                          │              │
│  │ merchant_id: "merch_XYZ"                                   │              │
│  │ amount: 900000                      (net amount)           │              │
│  │ status: "created"                                          │              │
│  └────────────────────────────────────────────────────────────┘              │
│                                                                              │
│  TABLE: payout_details                                                       │
│  ┌────────────────────────────────────────────────────────────┐              │
│  │ id: "poutdet_DEF456"                                       │              │
│  │ payout_id: "pout_ABC123"                                   │              │
│  │ tds_category_id: 14                 (194J category)        │              │
│  │ additional_info: {                                         │              │
│  │   "subtotal_amount": 1000000,       (gross)                │              │
│  │   "tds_amount": 100000,             (TDS deducted)         │              │
│  │   "merchant_detail": {              (remitter info)        │              │
│  │     "merchant_name": "Acme Corp",                          │              │
│  │     "merchant_pan": "ABCDE1234F"                           │              │
│  │   }                                                        │              │
│  │ }                                                          │              │
│  └────────────────────────────────────────────────────────────┘              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       │ (Async - on status change)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 7: STATUS UPDATE WEBHOOK                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  File: internal/app/payouts/fts_transfer_status_webhook.go:305               │
│  Function: UpdateStatusWithTransferStatusWebhook()                           │
│                                                                              │
│  // FTS (Fund Transfer Service) calls webhook when transfer completes        │
│                                                                              │
│  switch status {                                                             │
│  case appConstants.PROCESSED:         // Line 330                            │
│      oldStatus := payout.GetStatus()                                         │
│      payoutResponse, err = c.UpdateProcessedStatusWithTransferStatusWebhook()│
│      c.ProcessTdsForPayout(ctx, payout, oldStatus)    // ← TDS PUBLISHED     │
│                                                                              │
│  case appConstants.REVERSED:          // Line 336                            │
│      oldStatus := payout.GetStatus()                                         │
│      payoutResponse, err = c.UpdateReversedStatusWithTransferStatusWebhook() │
│      c.ProcessTdsForPayout(ctx, payout, oldStatus)    // ← TDS PUBLISHED     │
│  }                                                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 8: KAFKA MESSAGE PUBLISHING                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  File: internal/app/payouts/core.go:7858                                     │
│  Function: ProcessTdsForPayout()                                             │
│                                                                              │
│  // Only publish for terminal states                                         │
│  if MoneyTransferredStatus[payout.GetStatus()] == true {                     │
│                                                                              │
│      // Step 1: Load PayoutDetails (lazy load if needed)                     │
│      payoutDetails := payout.GetPayoutDetails(ctx)                           │
│                                                                              │
│      // Step 2: Extract TDS data                                             │
│      tdsCategoryId := payoutDetails.GetTDSDetailsPublic().GetTDSCategoryID() │
│      tdsAmount := payoutDetails.GetAdditionalInfo().GetTDSAmount()           │
│                                                                              │
│      // Step 3: Build Kafka message                                          │
│      data := TdsData{                                                        │
│          EntityID:      payout.GetID(),           // "pout_ABC123"           │
│          EntityType:    payout.EntityName(),      // "payout"                │
│          EntityStatus:  payout.Status,            // "processed"             │
│          MerchantID:    payout.MerchantID,        // "merch_XYZ"             │
│          TdsCategoryID: tdsCategoryId,            // 14                      │
│          TdsAmount:     &tdsAmount,               // 100000                  │
│      }                                                                       │
│                                                                              │
│      message := TdsMessage{                                                  │
│          Data:       data,                                                   │
│          Attributes: Attributes{Mode: "LIVE"},                               │
│      }                                                                       │
│                                                                              │
│      // Step 4: Publish to Kafka                                             │
│      eventClient := provider.GetEventClient(ctx)                             │
│      eventClient.PublishEvent(message, TdsTopic)  // "add-tds-entry"         │
│  }                                                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 9: KAFKA MESSAGE STRUCTURE                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Topic: "add-tds-entry"                                                      │
│                                                                              │
│  Message JSON:                                                               │
│  {                                                                           │
│    "data": {                                                                 │
│      "entity_id": "pout_ABC123",                                             │
│      "entity_type": "payout",                                                │
│      "entity_status": "processed",                                           │
│      "merchant_id": "merch_XYZ",                                             │
│      "tds_category_id": 14,                                                  │
│      "tds_amount": 100000                                                    │
│    },                                                                        │
│    "attributes": {                                                           │
│      "mode": "LIVE"                                                          │
│    }                                                                         │
│  }                                                                           │
│                                                                              │
│  Consumer: Vendor-Payments service (creates tds_payment record)              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 9.4 Key Functions Summary

| Function | File:Line | Purpose |
|----------|-----------|---------|
| `FundAccountPayout()` | `payoutController.go:187` | HTTP handler, validates request |
| `CreatePayoutToFundAccountV2()` | `core.go:469` | Core orchestration |
| `CheckIfPayoutDetailsIsNeeded()` | `processor/base.go:434` | Determines if PayoutDetails needed |
| `SetPayoutDetailsInPayout()` | `processor/base.go:404` | Sets TDS in PayoutDetails |
| `SetTDSCategoryID()` | `payoutDetails/model.go:76` | Sets category_id column |
| `SetAdditionalInfo()` | `payoutDetails/model.go:84` | Sets JSON with tds_amount |
| `UpdateStatusWithTransferStatusWebhook()` | `fts_transfer_status_webhook.go:305` | Handles status changes |
| `ProcessTdsForPayout()` | `core.go:7858` | Publishes TDS to Kafka |
| `PublishEvent()` | `pkg/events/events.go:37` | Sends to Kafka topic |

### 9.5 Model Relationships in Code

```go
// Payout model has PayoutDetails as embedded relation
// File: internal/app/payouts/model.go:90
type Payout struct {
    ...
    PayoutDetails payoutDetails.PayoutDetails `gorm:"association_autoupdate:false"`
    ...
}

// PayoutDetails is linked via payout_id
// File: internal/app/payoutDetails/model.go:22
type PayoutDetails struct {
    PayoutID       string                                  // Foreign key
    TDSCategoryID  nulls.UInt32                            // Direct column
    AdditionalInfo datatypes.JSONType[AdditionalInfoStruct] // JSON column
}

// TDS public interface combines both storage locations
// File: internal/app/payoutDetails/model.go:183
func (pd *PayoutDetails) GetTDSDetailsPublic() interfaces.IPayoutDetailsTDS {
    return &TDS{
        TDSCategoryID: pd.TDSCategoryID,                    // From column
        TDSAmount:     pd.GetAdditionalInfo().GetTDSAmount(), // From JSON
    }
}
```

### 9.6 Current Limitations (Why BB+ Changes Needed)

| Limitation | Impact |
|------------|--------|
| **Pre-computed TDS only** | Caller must compute TDS externally; Payouts doesn't validate |
| **No compute inputs stored** | Cannot audit what inputs generated the TDS amount |
| **No category code** | Only numeric ID stored, not human-readable "194J" |
| **No breakdown** | Only total TDS stored, not basic_tax/surcharge/cess breakdown |
| **Minimal Kafka message** | Consumer can't create full `tds_payment` record |
| **Single topic** | Can't differentiate BB+ vs Vendor-Payments flows |

---

## Questions & Open Items

1. **S2S Auth:** What authentication mechanism for Payouts → Tax-Compliance compute call?
2. **Timeout:** Recommended timeout for compute call (spec suggests 2-5s)?
3. **Circuit Breaker:** Strategy when Tax-Compliance is down?
4. **Backward Compatibility:** Is `IsBBPlusComputeFlow()` sufficient to distinguish from Vendor-Payments?
5. **LDC Validation:** What if LDC certificate expires between compute and payout creation?
6. **Kafka Consumer:** Who consumes `add-tds-entry` currently? Need to confirm Vendor-Payments owns it.

---

## 10. Vendor-Payments Repository - TDS Consumer

### 10.1 Repository Overview

**Root:** `/Users/prashant.chauhan/go/src/github.com/razorpay/vendor-payments`

```
vendor-payments/
├── cmd/
│   ├── vendor-payments/main.go    # API server (Twirp + gRPC)
│   ├── worker/main.go             # Kafka consumer (legacy)
│   ├── workerv2/main.go           # Kafka consumer (new)
│   └── ap-worker/main.go          # Accounting-Payouts worker
├── internal/
│   ├── taxpayments/               # TDS/Tax payment business logic (core.go: 145KB)
│   ├── initiatetds/               # Metro message handling for TDS initiation
│   ├── tdscategory/               # TDS category management
│   ├── icici/                     # ICICI Bank integration for TDS execution
│   ├── records/                   # Record management (credit/debit entries)
│   └── tasks/                     # Kafka consumer tasks
├── generated_endpoints/
│   └── proto/tax-payments/        # gRPC/Twirp service definitions
└── config/                        # Configuration files
```

### 10.2 Kafka Topic: `add-tds-entry`

**Consumer Configuration:**

| Setting | Value |
|---------|-------|
| Topic | `add-tds-entry` |
| Consumer Group | `vendor-payments` |
| Handler | `initiatetds.GetCore().HandleInitiateTdsJob()` |
| Target | `taxpayments.GetCore().InitiateTds()` |
| Idempotency | Uses `message_id` as idempotency key |

**Task Registration:** `/internal/tasks/initiatetds.go`
```go
func InitInitiateTdsKafka() {
    // Registers consumer for "add-tds-entry" topic
}
```

### 10.3 Message Format (Payouts → Vendor-Payments)

**Incoming Kafka Message:**
```json
{
  "message": {
    "data": "{\"merchant_id\":\"merch_XYZ\",\"entity_id\":\"pout_ABC123\",\"entity_type\":\"payout\",\"entity_status\":\"processed\",\"tds_amount\":100000,\"tds_category_id\":14}",
    "message_id": "unique_idempotency_key",
    "publish_time": "2026-02-19T10:30:00Z"
  },
  "subscription": "add-tds-entry-subscription"
}
```

**Parsed TdsEntryRequest:**
```go
type TdsEntryRequest struct {
    MerchantId    string  `json:"merchant_id"`
    EntityId      string  `json:"entity_id"`       // Payout ID
    EntityType    string  `json:"entity_type"`     // "payout"
    EntityStatus  string  `json:"entity_status"`   // "processed"/"reversed"
    TdsAmount     float64 `json:"tds_amount"`
    TdsCategoryId int64   `json:"tds_category_id"`
}
```

### 10.4 InitiateTds Flow

**Location:** `/internal/taxpayments/core.go:3866`

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ VENDOR-PAYMENTS: InitiateTds() FLOW                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. IDEMPOTENCY CHECK                                                        │
│     ├─ GetRecordByIKey(idempotency_key)                                      │
│     └─ If record exists → return nil (duplicate, already processed)         │
│                                                                              │
│  2. PARSE & VALIDATE                                                         │
│     ├─ json.Unmarshal(data, &tdsEntryRequest)                                │
│     └─ tdsEntryRequest.validate(ctx)                                         │
│                                                                              │
│  3. DETERMINE RECORD TYPE & AMOUNT                                           │
│     ├─ entity.GetTdsDeducted() → existing TDS on entity                      │
│     ├─ entity.GetTdsToBeDeducted() → new TDS amount                          │
│     ├─ If tdsDeducted < tdsToBeDeducted → RecordType = "credit"              │
│     └─ Else → RecordType = "debit"                                           │
│                                                                              │
│  4. ADD TAX PAYMENT ENTRY                                                    │
│     ├─ Find existing TaxPayment bucket OR create new one                     │
│     │   └─ Bucket key: (merchant_id, deduction_month, tds_category_id)       │
│     ├─ Create Record entry (credit/debit)                                    │
│     └─ Update TaxPayment aggregated amount                                   │
│                                                                              │
│  5. UPDATE PAYOUT (callback to Payouts service)                              │
│     └─ TaxPaymentAPICaller().UpdateTaxPaymentIdOnPayout(                     │
│            ctx, merchantId, payoutId, taxPayment.Id)                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 10.5 Database Schema (Vendor-Payments)

**Core Tables:**

```sql
-- Aggregated TDS bucket (per merchant, month, category)
TABLE tax_payments (
    id varchar(30) PRIMARY KEY,
    merchant_id varchar(14) NOT NULL,
    tax_type varchar(10),           -- "tds", "gst"
    deduction_month varchar(6),     -- "202602" (YYYYMM)
    status varchar(20),             -- "unpaid", "paid", "processing"
    due_date int,
    tax_type_id varchar(30),        -- FK to tds table
    created_at int,
    updated_at int
);

-- Individual TDS entries (credit/debit)
TABLE records (
    id varchar(30) PRIMARY KEY,
    added_to_tax_payment_id varchar(30),  -- FK to tax_payments
    entity_type varchar(20),              -- "payout", "vendor_payment", etc.
    entity_id varchar(30),                -- payout_id, invoice_id, etc.
    amount bigint,                        -- Amount in paise
    record_type varchar(10),              -- "credit" or "debit"
    idempotency_key varchar(50),          -- For duplicate detection
    created_at int
);

-- TDS details for a tax payment
TABLE tds (
    id varchar(30) PRIMARY KEY,
    tax_payment_id varchar(30),           -- FK to tax_payments
    tds_category_id int,                  -- FK to tds_categories
    challan_file_id varchar(30),
    cin varchar(50),                      -- Challan Identification Number
    crn varchar(50),                      -- Challan Reference Number
    tax_amount bigint
);

-- TDS category master data
TABLE tds_categories (
    id int PRIMARY KEY,
    code varchar(10),                     -- "194C", "194J", "194A", etc.
    name varchar(100),                    -- "Payment to Contractors"
    slab decimal(5,2),                    -- Rate: 1.00, 2.00, 10.00
    extern_goi_code varchar(10)           -- GOI code for filing
);
```

**Entity Types in Records:**

| Entity Type | Source |
|-------------|--------|
| `payout` | TDS from Payouts service (Kafka) |
| `vendor_payment` | TDS from invoice |
| `vendor_advance` | TDS from advance |
| `manual_tds` | Manually created TDS |
| `penalty` | Late fee penalty |

### 10.6 TaxPayment Aggregation Logic

```
Multiple TDS entries aggregate into single TaxPayment bucket:

Bucket Key = (merchant_id, deduction_month, tds_category_id)

Example:
┌────────────────────────────────────────────────────────────────┐
│ tax_payment_id: "tp_ABC123"                                    │
│ merchant_id: "merch_XYZ"                                       │
│ deduction_month: "202602"                                      │
│ tds_category_id: 14 (194J)                                     │
│ status: "unpaid"                                               │
│ total_amount: ₹25,000                                          │
├────────────────────────────────────────────────────────────────┤
│ Records:                                                       │
│ ├─ record_1: payout pout_001, credit, ₹10,000                  │
│ ├─ record_2: payout pout_002, credit, ₹8,000                   │
│ ├─ record_3: vendor_payment inv_001, credit, ₹5,000            │
│ └─ record_4: payout pout_003, credit, ₹2,000                   │
└────────────────────────────────────────────────────────────────┘
```

### 10.7 TaxPayment Status Lifecycle

```
                    ┌─────────┐
                    │ unpaid  │ ← Initial state
                    └────┬────┘
                         │ PayTaxPayment() called
                         ▼
                  ┌────────────┐
                  │ processing │
                  └──────┬─────┘
           ┌─────────────┼─────────────┐
           ▼             ▼             ▼
      ┌────────┐   ┌──────────┐   ┌────────────┐
      │  paid  │   │  failed  │   │ cancelled  │
      └────────┘   └──────────┘   └────────────┘
```

### 10.8 ICICI Bank Integration

**For actual TDS payment to Government of India:**

```
TaxPayment (status: unpaid)
       ↓ PayTaxPayment() API call
       ↓
ICICI Integration (internal/icici/)
       ├─ DoPayment() → ICICI TDS API
       ├─ Verify() → ICICI Verification API
       └─ GetChallan() → Download challan
       ↓
TaxPayment (status: paid)
       ↓
Challan stored (CIN, CRN numbers)
```

### 10.9 Key Files Summary

| File | Purpose |
|------|---------|
| `/internal/taxpayments/core.go` | Core business logic (InitiateTds, AddTaxPaymentEntry) |
| `/internal/taxpayments/model.go` | Data models (TdsEntryRequest, TaxPayment, Record) |
| `/internal/taxpayments/repo.go` | Database operations |
| `/internal/tasks/initiatetds.go` | Kafka consumer for `add-tds-entry` |
| `/internal/initiatetds/core.go` | Metro message handler |
| `/internal/icici/` | ICICI Bank payment integration |
| `/generated_endpoints/proto/tax-payments/service.proto` | gRPC service definition |

### 10.10 What Vendor-Payments Does NOT Have (BB+ Gaps)

| Missing | Impact |
|---------|--------|
| **No compute endpoint** | Can't compute TDS - only receives pre-computed values |
| **No breakdown storage** | Can't store basic_tax/surcharge/cess breakdown |
| **No LDC handling** | No Lower Deduction Certificate support |
| **Minimal Kafka fields** | Only receives `tds_amount` + `tds_category_id` |
| **No BB+ topic** | Uses same topic as other flows |

---

## 11. End-to-End TDS Flow: Payouts → Vendor-Payments

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                           COMPLETE TDS FLOW                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  PAYOUTS SERVICE                                                              │
│  ════════════════                                                             │
│  1. POST /v2/payouts                                                          │
│     └─ Request: { amount: 900000, tds: { category_id: 14, amount: 100000 } } │
│                                                                               │
│  2. CreatePayoutToFundAccountV2() → CreatePayout()                            │
│     └─ SetPayoutDetailsInPayout() stores TDS in payout_details table          │
│                                                                               │
│  3. Payout processed by FTS (Fund Transfer Service)                           │
│     └─ Status changes to PROCESSED                                            │
│                                                                               │
│  4. ProcessTdsForPayout() triggered                                           │
│     └─ Publishes to Kafka topic: "add-tds-entry"                              │
│        Message: {                                                             │
│          entity_id: "pout_ABC123",                                            │
│          entity_type: "payout",                                               │
│          entity_status: "processed",                                          │
│          merchant_id: "merch_XYZ",                                            │
│          tds_category_id: 14,                                                 │
│          tds_amount: 100000                                                   │
│        }                                                                      │
│                                                                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  KAFKA                                                                        │
│  ═════                                                                        │
│  Topic: "add-tds-entry"                                                       │
│  Consumer Group: "vendor-payments"                                            │
│                                                                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  VENDOR-PAYMENTS SERVICE                                                      │
│  ═══════════════════════                                                      │
│  1. WorkerV2 consumes Kafka message                                           │
│     └─ initiatetds.HandleInitiateTdsJob()                                     │
│                                                                               │
│  2. taxpayments.InitiateTds()                                                 │
│     ├─ Idempotency check (message_id)                                         │
│     ├─ Parse TdsEntryRequest                                                  │
│     ├─ Determine credit/debit                                                 │
│     └─ AddTaxPaymentEntry()                                                   │
│                                                                               │
│  3. Database writes:                                                          │
│     ├─ tax_payments: aggregate bucket (merchant + month + category)           │
│     ├─ records: individual entry (payout_id, amount, credit)                  │
│     └─ tds: TDS details (category_id)                                         │
│                                                                               │
│  4. Callback to Payouts:                                                      │
│     └─ UpdateTaxPaymentIdOnPayout(pout_ABC123, tp_XYZ789)                     │
│        └─ Stores tax_payment_id in payout_details.tax_payment_id              │
│                                                                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  RESULT                                                                       │
│  ══════                                                                       │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐   │
│  │    PAYOUTS      │        │     KAFKA       │        │ VENDOR-PAYMENTS │   │
│  ├─────────────────┤        ├─────────────────┤        ├─────────────────┤   │
│  │ payout_details: │        │                 │        │ tax_payments:   │   │
│  │  tds_category_id│───────▶│ add-tds-entry   │───────▶│  id: tp_XYZ789  │   │
│  │  tds_amount     │        │                 │        │  amount: 100000 │   │
│  │  tax_payment_id │◀───────│                 │◀───────│  status: unpaid │   │
│  │   = tp_XYZ789   │        │                 │        │                 │   │
│  └─────────────────┘        └─────────────────┘        └─────────────────┘   │
│                                                                               │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

---

## 12. Tax-Compliance Repository - TDS Management Service

### 12.1 Repository Overview

**Root:** `/Users/prashant.chauhan/go/src/github.com/razorpay/tax-compliance`

```
tax-compliance/
├── cmd/
│   ├── compliance/main.go           # Primary HTTP/gRPC server
│   ├── compliance_migration/        # Database migration tool
│   └── compliance_worker/           # Background task worker
├── internal/compliance/
│   ├── auth/                        # Authentication
│   ├── dto/                         # Data transfer objects
│   ├── model/                       # Database models
│   ├── repo/                        # Repository layer
│   ├── service/                     # Business logic
│   │   ├── tds_payments/           # TDS payment service
│   │   ├── tds_setup/              # TDS setup service
│   │   ├── ldc_certificates/       # LDC certificate service
│   │   └── tds_filing/             # TDS filing service
│   └── validator/                   # Request validation
├── pkg/client/                      # External service clients
│   ├── apimonolith/                # API Monolith client
│   ├── quicko/                     # Quicko service client
│   └── xpayroll/                   # xPayroll client
├── proto/                           # Protocol buffer definitions
└── migrations/                      # Database migrations
```

### 12.2 CRITICAL FINDING: No `/v1/tds/compute` Endpoint

**The Tax-Compliance service does NOT have a TDS compute endpoint.**

Current state:
- ❌ No `/v1/tds/compute` endpoint
- ❌ No slab-based rate calculation
- ❌ No automated TDS deduction logic
- ❌ No category → rate mapping
- ✅ LDC certificates stored but NOT auto-applied
- ✅ TDS payment records created with client-provided amounts

**Implication:** The compute endpoint mentioned in the Tech Spec needs to be **built from scratch**.

### 12.3 Existing TDS Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/tds-payments` | POST | Create TDS payment (amounts provided) |
| `/v1/tds-payments/{id}` | GET | Get TDS payment |
| `/v1/batches` | POST | Create batch for filing |
| `/v1/batches/{id}` | GET | Get batch details |
| `/internal/v1/tds/ldc_certificates` | POST | Create LDC certificate |
| `/internal/v1/tds/ldc_certificates/query` | POST | Query LDC certificates |
| `/internal/v1/compliance_settings/merchants/{id}` | GET/POST | Compliance settings |

### 12.4 Database Models

**TDSPayment Model:**
```go
type TDSPayment struct {
    ID                   string
    MerchantID           *string
    SourceTransactionID  string              // Payout ID
    TransactionSource    string              // "BB_PLUS" or "PAYROLL"
    TAN                  string
    PAN                  string
    Category             string              // TDS category (14-char ID)

    // TDS Component Amounts (in paise)
    BasicTaxAmount       int64
    InterestAmount       int64
    SurchargeAmount      int64
    CessAmount           int64
    PenaltyAmount        int64
    TotalTDSAmount       int64

    // LDC Support
    VendorPaymentAmount  *int64              // Gross amount
    LDCApplicable        bool
    LDCRate              *float64
    LDCertificateID      *string

    // Status
    Status               TDSPaymentStatus    // pending/success/failed
    DeducteeID           string
    ChallanPaymentID     *string
    BatchID              *string
}
```

**LDCCertificate Model:**
```go
type LDCCertificate struct {
    ID                string
    MerchantID        string
    ContactID         string
    TDSCategoryID     string
    CertificateNumber string
    LDCRate           int32               // Percentage (e.g., 5 = 5%)
    Amount            *uint64
    ValidFrom         int64               // Unix timestamp
    ValidTo           int64               // Unix timestamp
    VendorPAN         *string
    FileID            string
}
```

**Deductee Model:**
```go
type Deductee struct {
    ID             string
    MerchantID     *string
    Name           string
    PAN            string
    ContactID      *string              // Link to Contacts service
    ReferenceID    string
    ReferenceType  string               // "people" or "contact"
}
```

### 12.5 Transaction Source Enum

```go
const (
    TransactionSourceBBPlus  = "BB_PLUS"    // BB+ system (Payouts)
    TransactionSourcePayroll = "PAYROLL"     // xPayroll system
)
```

### 12.6 Create TDS Payment API Contract

**Proto Definition:** `/proto/tax_compliance/compliance/tds_payments/v1/tds_payments.proto`

```protobuf
message CreateTDSPaymentRequest {
  string entity_id = 1;                    // Payout/transaction ID
  TransactionSource source = 2;            // BB_PLUS or PAYROLL
  int64 organization_id = 3;
  string merchant_id = 4;
  string tan = 5;                          // TAN of deductor
  string pan = 6;                          // PAN of deductee
  string name = 7;                         // Deductee name
  PaymentDetails payment_details = 8;
  DeducteeDetails deductee_details = 9;
}

message PaymentDetails {
  string deduction_date = 1;
  string source_transaction_id = 2;        // Payout ID
  int64 major_head = 3;                    // Tax major head
  int64 minor_head = 4;                    // Tax minor head
  string category = 5;                     // TDS category (14-char)
  int64 basic_tax_amount = 6;              // In paise (CLIENT PROVIDED)
  int64 interest = 7;                      // In paise
  int64 surcharge = 8;                     // In paise
  int64 cess = 9;                          // In paise
  int64 penalty = 10;                      // In paise
  int64 total_tds_amount = 11;             // Total (computed as sum)
}
```

**Note:** All amounts are **client-provided**, not computed by Tax-Compliance.

### 12.7 LDC Certificate Handling

**Create LDC:**
```protobuf
message CreateLDCCertificateRequest {
  string merchant_id = 1;
  string contact_id = 2;
  string tds_category_id = 3;
  string certificate_number = 4;
  int32 ldc_rate = 5;                      // e.g., 5 for 5%
  optional uint64 amount = 6;
  int64 valid_from = 7;                    // Unix timestamp
  int64 valid_to = 8;                      // Unix timestamp
}
```

**Query LDC:**
```protobuf
message LDCCertificatesQueryRequest {
  string merchant_id = 1;
  string contact_id = 2;
  optional string tds_category_id = 3;
  optional int64 valid_on = 4;             // Check validity at timestamp
}
```

**Current Limitation:** LDC certificates are stored but **not automatically applied** to reduce TDS rates. This logic needs to be built.

### 12.8 Key Files

| File | Purpose |
|------|---------|
| `/internal/compliance/service/tds_payments/tds_payments.go` | TDS payment service |
| `/internal/compliance/service/ldc_certificates/service.go` | LDC certificate service |
| `/internal/compliance/model/tds_payment.go` | TDSPayment model |
| `/internal/compliance/model/ldc_certificate.go` | LDCCertificate model |
| `/internal/compliance/model/constants.go` | BB_PLUS constant |
| `/proto/tax_compliance/compliance/tds_payments/v1/tds_payments.proto` | API contract |

### 12.9 What Needs to be Built for BB+ Compute

| Component | Status | Description |
|-----------|--------|-------------|
| `/v1/tds/compute` endpoint | ❌ Missing | Compute TDS from gross amount + category |
| TDS Category → Rate mapping | ❌ Missing | Lookup rate from category code (194J → 10%) |
| Slab management | ❌ Missing | Handle different rates per category |
| LDC auto-application | ❌ Missing | Auto-reduce rate if valid LDC exists |
| Compute response | ❌ Missing | Return breakdown (basic_tax, surcharge, cess) |

---

## 13. Complete System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         BB+ TDS INTEGRATION ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌──────────────────┐                      ┌──────────────────┐                 │
│  │   BB+ FRONTEND   │                      │  TAX-COMPLIANCE  │                 │
│  │   (Dashboard)    │                      │     SERVICE      │                 │
│  └────────┬─────────┘                      └────────┬─────────┘                 │
│           │                                         │                            │
│           │ POST /v2/payouts                        │ ❌ /v1/tds/compute         │
│           │ {                                       │    (NEEDS TO BE BUILT)     │
│           │   amount: 10000 (gross)                 │                            │
│           │   tds: {                                │ ✅ /v1/tds-payments        │
│           │     category: "194J",                   │    (stores TDS records)    │
│           │     is_ldc_applicable: true             │                            │
│           │   }                                     │ ✅ /v1/ldc_certificates    │
│           │ }                                       │    (stores LDC certs)      │
│           ▼                                         │                            │
│  ┌──────────────────┐                               │                            │
│  │     PAYOUTS      │ ─────────S2S CALL────────────▶│                            │
│  │     SERVICE      │     (compute TDS)             │                            │
│  │                  │◀────────RESPONSE──────────────│                            │
│  │  - Store gross   │   {                           │                            │
│  │  - Compute net   │     tds_amount: 1000,         │                            │
│  │  - Create payout │     basic_tax: 900,           │                            │
│  │    with net amt  │     cess: 100,                │                            │
│  │                  │     effective_rate: 10%       │                            │
│  └────────┬─────────┘   }                           │                            │
│           │                                         │                            │
│           │ Payout PROCESSED                        │                            │
│           ▼                                         │                            │
│  ┌──────────────────┐                               │                            │
│  │      KAFKA       │                               │                            │
│  │  add-tds-entry   │ ─────────CONSUMED BY─────────▶│                            │
│  │  (or bb-plus)    │                               │                            │
│  └──────────────────┘                      ┌────────┴─────────┐                 │
│           │                                │  (Option A)      │                 │
│           │                                │  Tax-Compliance  │                 │
│           │                                │  consumes topic  │                 │
│           ▼                                └──────────────────┘                 │
│  ┌──────────────────┐                                                           │
│  │ VENDOR-PAYMENTS  │◄─────────OR (Option B)────────                            │
│  │    SERVICE       │         Vendor-Payments                                   │
│  │                  │         continues to consume                              │
│  │  - Aggregates    │                                                           │
│  │  - Creates       │                                                           │
│  │    tax_payment   │                                                           │
│  │  - Pays to GOI   │                                                           │
│  └──────────────────┘                                                           │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 14. Summary: Three Services & Their Roles

| Service | Role | TDS Responsibility |
|---------|------|-------------------|
| **Payouts** | Payout creation & processing | Stores TDS, publishes to Kafka |
| **Vendor-Payments** | TDS aggregation & GOI payment | Aggregates TDS, pays to government |
| **Tax-Compliance** | TDS management & filing | TDS records, LDC, filing (NO compute yet) |

---

## Questions & Open Items

1. **Compute Endpoint:** `/v1/tds/compute` needs to be built in Tax-Compliance
2. **S2S Auth:** Basic Auth exists in Tax-Compliance, need Payouts credentials
3. **Timeout:** Recommend 2-3s with circuit breaker
4. **LDC Auto-Apply:** Build logic to auto-apply LDC rate reduction
5. **Category → Rate Mapping:** Need master data for TDS rates per category
6. **Kafka Topic:** Two options:
   - Option A: New topic `add-tds-entry-bb-plus` → Tax-Compliance
   - Option B: Enhance existing `add-tds-entry` → Vendor-Payments forwards to Tax-Compliance
7. **Deductee Creation:** Tax-Compliance needs contact_id → deductee mapping

---

*End of Document*
