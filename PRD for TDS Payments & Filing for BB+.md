# TDS Payment & Filing for BB+

Author: [Shubham](mailto:shubham.sinha@razorpay.com)  
Date: Oct 23, 2025

# 1\. How will we announce this offering to the world? 

## 1.1. How would you post this on the Product bulletin in slack?

🚀 **New Launch: Automated TDS Payment & Filing in BB+ Vendor Payouts**

Hey team\! 👋

We’re excited to announce that **BB+** now supports **Automated TDS Payment and Filing**, making TDS compliance effortless for businesses dealing with Payouts.

### **The Problem**

Today, managing TDS in the payouts flow is tedious and manual:

* Businesses must calculate and deduct TDS while making payout  
* Track and pay TDS before the 7th each month  
* Download and reconcile challans for quarterly filings  
* Manually generate and file Form 26Q

This repetitive process increases errors, delays, and compliance risks.

### **Our Solution**

With this launch, BB+ automates TDS **calculation, deduction, payment, and filing**—all through the Compliance Service APIs.

 ✅ **At payout:** TDS is auto-calculated and deducted  
 ✅ **Post-payment:** Razorpay automates payment to the government and 26Q filing  
 ✅ **Real-time visibility:** CIN(Challan reference number) and filing acknowledgements appear on the BB+ dashboard

This launch aligns with Razorpay’s mission to **power and streamline all forms of business money movement**, helping businesses focus on growth while we handle their compliance.

💬 *“You take care of your business — we’ll take care of your compliance.”*

## 1.2. What would a 1-page press-release look like?

#### **Razorpay Payout product Launches Automated TDS Payment & Filing to Simplify Tax Compliance for Businesses**

**Bangalore, India – \[Insert Date\]** – Razorpay today announced the launch of **Automated TDS Payment and Filing** for customers who are using Payouts product, a new feature designed to simplify one of the most complex aspects of financial compliance for Indian businesses.

With this release, BB+ customers can now automate the entire TDS lifecycle — from deduction to payment and filing — through seamless integration with Razorpay’s **Compliance Service APIs**.

Traditionally, businesses have had to manually calculate, deduct, and file TDS for every vendor transaction — a process prone to delays and errors. With this launch, BB+ removes that friction entirely. The system automatically deducts TDS at the time of payout, moves the funds securely to the Razorpay Compliance account, and ensures timely payment and filing with the government portal.

Once payment and filing are complete, customers receive real-time updates on their BB+ dashboard, including **CIN**, **BRN**, and the **Filing Acknowledgement PDF**, providing full transparency and compliance confidence.

“Our mission has always been to help businesses focus on growth while we handle the complexity of financial operations. Automating TDS compliance is a major step forward in that journey,” said \[Insert Spokesperson Name\], \[Insert Title\], Razorpay.

This new offering is available for all BB+ customers from \[Insert Launch Date\]. Businesses can enable the feature directly from their BB+ dashboard.

For more information, visit \[Insert Razorpay Product Page link\].

## 1.3. How will we enable Sales & Solution on this offering? (including How-to-use)

**Training Components:** 

* Feature demo & SOP walkthrough for Ops, Sales & Solution teams.

## 1.4. What would a merchant facing tech docs in razorpay.com/docs look like?

## \<WIP\>

## 1.5. Where would merchant go to to learn more (Knowledge-base)

**Knowledge Base Articles:**

* In the Razorpay Doc section, we will add an article for Automated TDS Payment and Filing for BB+ customers.

# 2\. What problem are we solving

## 2.1. Describe the problem in detail

TDS management within vendor payout flows represents a significant operational burden for businesses in India. Every time a business makes a vendor payment, they must navigate a complex, multi-step compliance process that is highly manual, error-prone, and time-consuming.

**The Current Manual Process:**

Businesses today face a four-stage compliance workflow for TDS:

1. **Manual TDS Calculation & Deduction:** When making vendor payments, organizations must manually calculate the applicable TDS rate based on the payment category (194J, 194C, etc.), deduct the correct amount before payment, and record the transaction for each payout. This requires continuous tracking across potentially hundreds of vendor transactions monthly.  
2. **Payment Tracking & Government Remittance:** After deduction, businesses need to collate all TDS transactions and ensure payment to the government before the 7th of the following month. Missing this deadline results in interest charges and penalties(1.5%/Month). Organizations must maintain detailed records to avoid reconciliation issues later.  
3. **Challan Management :** Once payments are made to the government, businesses must manually download and maintain challans (payment receipts) as proof of payment. These challans are critical for quarterly filing, and misplacing them creates significant reconciliation challenges and compliance risks.  
4. **Quarterly TDS Filing (Form 26Q):** Every quarter, businesses must reconcile payout data, challan data, and deduction details, generate a Form 26Q file, and manually file it on the government portal. Errors in this process often lead to rework, corrections, notices from the tax department, and additional compliance overhead.  
5. **Form 16A :** Every quarter Contractors & Vendors ask for Form 16A from merchants              Every business once completed the TDS filing for the quarter has to share form 16A to vendors showing how much TDS was deducted. Since there is no automated solution, orgs will have to download the forms 16A for each of the vendors from the portal and then share with them separately.

**Root Causes of the Problem:**

The fundamental issue is that TDS compliance is treated as a separate, disconnected process from the core business activity of making vendor payments. This separation forces businesses to:

* Maintain parallel tracking systems for payments and compliance  
* Manually reconcile data across multiple sources  
* Context-switch between operational payout activities and regulatory compliance  
* Rely heavily on CAs (Chartered Accountants) or dedicated tax teams for execution

**Impact & Severity:**

According to industry research, tax teams at Indian companies spend approximately 70% of their time on tax compliance, with TDS being a significant contributor. This burden stems from managing frequent TDS transaction cycles, detailed reconciliations, complex reporting requirements, and responding to regulatory notices.

From our conversations with merchants, 6-7 out of 10 businesses report that they either have faced penalties/interest charges due to TDS compliance delays, or they maintain regular follow-ups with their CAs specifically to ensure TDS compliance is completed on time. This constant vigilance indicates the anxiety and operational overhead this process creates.

The volume of TDS involved amplifies this problem \- the government collects approximately ₹4 lakh crore in TDS annually, with ₹1-1.5 lakh crore coming solely from vendor/contractor payouts. This represents a massive volume of recurring transactions that businesses must manage compliantly every single month.

## 2.2. Who are we solving the problem for?

**Primary Customers:**  
We are building this solution for all businesses using Razorpay’s BB+ product to manage their vendor and contractor payments. These businesses typically handle recurring B2B payments where TDS compliance is mandatory.

**Target Profile:** Companies that face complexity in TDS management due to one or more of the following factors:

* Multiple TDS categories across different payment types  
* High transaction volumes requiring frequent TDS deductions  
* Payments to numerous vendors, contractors, or entities  
* Recurring monthly payment cycles

70-80%([Link](https://docs.google.com/spreadsheets/d/1ZEqR6Z7WJj-MTS-RiOcFmLqkB_j_HH-j_LfvmCsxZFQ/edit?gid=1461121806#gid=1461121806) to the analysis) of our BB+ customer base falls into the below mentioned sectors, making them critical for driving revenue, increasing payout volumes, and reducing churn:

| Industry | TDS Payment Scenarios | Key Pain Points |
| :---- | :---- | :---- |
| E-commerce | Vendor/contractor payments Affiliate commissions Agency payments Seller payouts Foreign vendor payments Warehouse/office rentals | High volume of transactions across multiple TDS categories (194C, 194H, 194J, 195\) makes tracking, calculation, deduction, payment, and filing extremely complex |
| D2C Brands | Vendor/contractor payments Affiliate commissions Agency payments Foreign vendor payments Warehouse/office rentals | Similar to e-commerce: high volumes with multiple TDS categories creating operational burden |
| IT & Software | Contractor payments (194J) NRI payments (195) ESOP/equity payouts (192) Warehouse/office rentals | Large contractor base with high payment frequency NRI payments require separate tracking and documentation ESOP events involve high-value, complex TDS calculations for multiple employees |
| EdTech | Contract teacher payments Vendor payments Rental payments (194I) | Monthly recurring payments to large teacher base makes the process repetitive and operationally intensive |
| Co-working Spaces | Rental payments (194IB) Contractor/vendor payments | High monthly transaction volume across numerous landlords and service providers |
| Logistics |  Rider/contractor payments Vendor payments ESOP/equity payouts | High volume combined with workforce attrition makes TDS management and documentation difficult |

We can also target new sectors where our presence is currently low, but TDS challenges exist, making them strong opportunities for acquisition. I have listed those sectors along with their key use cases for reference.

| Sector | TDS Payment Scenarios | Key Pain Points |
| :---- | :---- | :---- |
| Quick Commerce/Grocery | Gig worker payments Warehouse/dark store rentals | Extremely high transaction volumes with workforce churn create compliance tracking challenges |
| Manufacturing & Real Estate | Contractor payments Machinery/equipment purchases Foreign vendor payments | Mix of domestic (194C) and international (195) TDS requirements |
| NBFC, Lending & Insurance | Interest payments to lenders/investors (194A) Agent commissions (194H, 194D) Contractor payments Rental payments | Regulated entities with strict compliance requirements and high penalty exposure |
| Agencies & Marketing Firms | Contractor/vendor payments Commission payments | Multiple payment types requiring different TDS treatments |

Apart from these, we are also solving this for all organizations that use both BB+ and Payroll to manage their contractor payments.

## 2.3. How many such customers exist?

Each month, the Indian government collects substantial TDS across a range of payment categories. For FY 2024–25, total TDS collections are estimated at approximately ₹4L crore for the year—translating to an average monthly collection of around ₹33,000 crore. Out of this, non-salary TDS accounts for 40-50%, and within that, 80-90% comes from vendor/contractor payments, which equals approximately ₹10-15K crore per month.

Listed below are all industries along with their TAM, SAM, SOM, and Razorpay share:

| Industry | TAM | SAM  | SOM | Razorpay Share (10% of SOM) | Monthly NR Potential | Confidence | TDS % out of total payouts | Comment |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **E-commerce** | 20000 \-25000 companies | 8000 \- 10000 (40%) | 1600 \- 2000 (20%) | 160 \- 200 orgs | 2.4 \- 3L | HIGH | 40-60% | E-commerce market $125B in 2024, projected $345B by 2030 [IBEF](https://www.ibef.org/industry/ecommerce). Have taken TAM based on sellers/platforms requiring TDS compliance |
| **D2C Brands** | 1500-2000 orgs | 900-1400(60%) | 180-280(20%) | 18-28 orgs | 27K \- 42K | HIGH | 25-40% | India has 800+ D2C brands with $80B market size in 2024, 11,000 total D2C companies but only \~800 funded [Statista](https://www.statista.com/topics/9783/d2c-market-in-india/)[D2CStory](https://www.d2cstory.com/blog-posts/the-complete-guide-to-indian-d2c-brands-market-size-growth-and-opportunities-in-2025/) |
| **IT & Software** | 30000-35000 companies | 12000-14000 (40%) | 2400-2800 (20%) | 240-280 orgs | 3.6-4.2L | HIGH | 50-70% | IT professional services market $64.3B by 2030 [Grand View Research](https://www.grandviewresearch.com/horizon/outlook/it-professional-services-market/india). TAM based on software services firms |
| **EdTech** | 5000 \- 7000 orgs | 2000-2800 (40%) | 400-560 (20%) | 40-56 orgs | 60-84K | MEDIUM | 40-60% |  |
| **Co-working Spaces** | 800-1000 orgs | 400-500 (50%) | 80-100 (20%) | 8-10 orgs | 12-15K  | MEDIUM | 30-45% |  |
| **Logistics** | 12000 \- 15000 companies | 4800-6000 (40%) | 960-1200 (20%) | 96-120 orgs | 1.4L \- 1.8L | MEDIUM | 30-40% | Logistics firms with high gig worker payments requiring TDS |
| **Quick Commerce** | 15-20 major players | 10-15 (50-75%) | 5-10 (50%) | 2-5 orgs | 5-7.5K | HIGH | 35-50% | Major players: Zepto, Blinkit, Swiggy Instamart, Dunzo, Meesho, etc. Limited player count but high transaction volumes. They have a huge number of transactions, so monthly fees will also be high and money movement through us will also be huge |
| **Manufacturing & Real Estate** | 35000-40000 companies | 14000-16000 (40%) | 2800-3200 (20%) | 280-320 orgs | 4.2-4.8L | MEDIUM | 20-30% |  |
| **NBFC/Lending/Insurance** | 10000-12000 orgs | 5000-6000 (50%) | 1000-1,200 (20%) | 100-120 orgs | 1.5-1.8L | HIGH | 25-40% |  |
| **Agencies/Marketing/Consulting** | 20000-25,000 firms | 8000-10000 (40%) | 1600-2,000 (20%) | 160-200 orgs | 2.4-3L | MEDIUM |  | \~10,000 consulting firms across major cities [Scribd](https://www.scribd.com/document/422179146/The-Market-Size-of-SME-Consulting-in-India) plus marketing agencies |

**Overall Market Summary :** Listing down the overall summary

| Metric | Number |
| :---- | :---- |
| **TAM** | 160000 \- 170000 org |
| **SAM**  | 65000-70000 orgs |
| **SOM**  | 13000-14000 orgs |
| **Razorpay Share** | 1400-1500 orgs |
| **Monthly NR Potential** | 20L \- 22L |
| **Annual NR Potential** | 2.4-2.8 cr |

**Note:** The above NR numbers are mostly generated by TDS fees that we will charge. Apart from that, there will be increased acquisition due to this feature, which will further add to the NR. Apart from the NR mentioned above, we will also generate AMB from these organizations. Based on an assumption that an average organization will have around 10-15L of TDS transactions, this translates to \~₹200 Cr of money movement through us.

**Existing Customer Base Analysis:**

Beyond the market opportunity mentioned above, we have strong insights from our existing customer base and pipeline:

**Current TDS Volume (I2P Flow):**

* Around 70-80 organizations process TDS payments through Razorpay every month, with a total monthly TDS value of approximately 26 lakhs (3.1 cr annually)

**Impact of Migrating These Orgs from Payroll to BB+**

* Around \~400 orgs with 14K FTEs currently use Payroll for vendor payments. If we don’t migrate them, they are likely to churn. Migration helps us retain them. When they were on Payroll, we were providing them Payment and Filing support. Since they are now migrating them to BB+ dashboard these will become one of our early customers when we launch TDS Payment and Filing in BB+.   
  * Out of 400 orgs, in phase 1 we are migrating around 177 orgs,   
    * AMB impact of these orgs \- 8 cr.

**Validated Demand from existing customers:**

* We have recently conducted a survey for BB+ customers to check if they have TDS use cases and whether it is a problem for them or not. From our survey of 150 businesses, approximately 60-70% reported active TDS use cases related to vendor/contractor payments and expressed strong interest in our Automated TDS Payment and Filing feature  
* Apart from the above survey, around 90 organizations are currently processing vendor payouts through Razorpay's Payroll and BB+ products but are not yet using our TDS services—primarily because TDS filing wasn't previously available within BB+. These represent high-conversion prospects.  
* All \~80 I2P users currently using TDS Payment can immediately adopt our TDS Filing solution, as the payment infrastructure is already in place  
* All BB+ customers making vendor payments are potential users of this feature, as TDS compliance is mandatory for most business-to-business transactions in India  
* According to the Sales team (Gagan's team), they receive 5-10 inbound requests every month specifically asking about TDS automation capabilities, indicating consistent organic demand

## 2.4. Have we spoken to any customers?

Yes, we have conducted detailed discussions with multiple customers to understand their current TDS process and issues they face and gather feedback on the proposed solution.  
Below are some key insights from these conversations:

**kidebaj.com (Marathi Kida Creative Media LLP)**

Business Profile: Kidebaj is a Marathi content and apparel brand. They operate their own channel for Marathi-language content and also sell Marathi-themed merchandise through their e-commerce store.

**Current Challenges:**

* They calculate TDS manually before initiating payouts through Razorpay.  
* After that, they share the data manually with their CA, who handles payment and filing separately.  
* The customer mentioned that coordinating with the CA is frustrating and time-consuming, as they have to send data offline and keep following up regularly. If they have any more data requirements then again they will call us and we then check and then make the payment. Sometimes they do the calculation wrong and deduct the wrong amount. In that case orgs again have to then work with vendors to get it sorted.

**JOGOHEALTH PRIVATE LIMITED**

**Business Profile:** JogoHealth operates a chain of healthcare clinics and manages payroll and compliance in-house.

**Current Challenges:**

* The person we spoke with is a CA who manages payroll and compliance for the company.  
* They use Razorpay Payroll and appreciate its end-to-end automated compliance capabilities.  
* They are also using our I2P flow for payouts and mentioned that while we already support TDS payment, the lack of TDS filing support forces them to handle Form 26Q manually outside the system.  
  * As per the POC, like how we do it in Payroll, similarly we should calculate, deduct the TDS and then make the payment and give us the confirmation. Otherwise only for this flow he has to manually handle all these data and do the filing.  
* The CA specifically requested that Razorpay extend its compliance automation to include TDS filing, so everything can be managed seamlessly within one platform.

Apart from these Aryan has done a detailed research on this by talking to multiple customers. Here is the [doc link](https://docs.google.com/document/d/12_lwEHWsNy9u0QEAXRSrVu9y6CVZjhsa0LS8yA5FjLA/edit?tab=t.0) for reference. Adding the summary of his research below for reference:

* Almost 80–85% of customers said that it should be a must-have feature in the software.  
* Most of the firms either do it manually, rely on CA partners, or have an in-house accounting team, creating coordination overhead. Alternatively, they handle things manually, which is not only a big, mundane, and time-consuming task but also leads to errors.  
* According to them, it will not only save time but also help avoid misses, and they won’t need to get into complex compliance procedures.  
* One customer mentioned that if Razorpay can pull information from other accounting tools and only handle the payment, it would be very helpful, as they do not want to maintain multiple ledgers in two different software.

## 2.5. Any quantification of the problem?

We have quantified the problem through market research and customer feedback.

**Time & Resource Impact:**  
Most of the businesses we have talked to always say that time is a very big issue for them. They also mention that since compliance is a tedious use case, as the volume increases, they need more resources to handle the same.

As per the report from Deloitte, companies spend around 70% of their time on compliance payments, with TDS being the biggest issue. \[[Here’s the link to that report](https://timesofindia.indiatimes.com/large-companies-spend-overwhelming-amount-of-time-on-tax-compliance-deloitte-survey/articleshow/100095999.cms)\]

Based on customer interviews and industry research, here’s the quantified burden of manual TDS compliance:

**Monthly Time Investment per Business:**

* TDS calculation and deduction tracking: 2–3 hours/month  
* Payment coordination with CA: 1–2 hours/month  
* Challan download and organization: 1 hour/month  
* Data reconciliation for filing: 3–4 hours/quarter 

Total: 5–7 hours per month spent on TDS compliance activities. This also depends on the number of transactions. The count might go up if the transaction is high.For a business with a finance employee earning 40K–60K/month, this represents 7K–11K in opportunity cost monthly, or 80K–1.2L annually in internal resource time.

**External CA Costs:**

* Monthly TDS payment handling: ₹1,000–3,000/month  
* Quarterly TDS filing (Form 26Q): ₹5,000–15,000/quarter  
* Annual CA cost for TDS compliance: ₹15,000–20,000

Combined annual burden per business: ₹1L–1.5L (internal time \+ external costs)

**Penalty & Interest Charges:**  
From our customer conversations, we have identified that customers often fear missing deadlines and incurring penalty charges. Some customers said they do regular follow-ups with CAs to ensure payments are completed on time.

* Around 40–50% of businesses report having faced penalties or interest charges for TDS compliance delays at least once.  
* Interest charges: 1.5% per month on unpaid TDS amounts

**Market-Level Impact:**

* **TDS Collection Volume:**  
  * Total TDS collected annually in India: ₹4 lakh crore (across all sections)  
  * TDS from vendor payments (194J, 194C, 194I, 194H): ₹1–1.5 lakh crore annually  
  * Number of businesses filing TDS returns: 3–4 million annually

# 3\. Is there any alternate solution to this problem? (optional)

Currently, the process of managing TDS in vendor payouts flow is manual and broken.

* Merchants must calculate the TDS amount manually and deduct it upfront before making vendor payments.  
* TDS challan payments are then handled either manually by the merchant or through third-party CA firms.  
* Most available software solutions only generate the Form 26Q text file required for TDS filing. Merchants must download this file and either:  
  * Upload and file it manually on the government portal, or  
  * Hire a CA or engage a third-party service provider to complete the filing on their behalf.

# 4\. Is this solved by any of our competitors?

| Provider | Automated TDS Payment | TDS Filing Automation | Managed/Manual |
| :---- | :---- | :---- | :---- |
| RazorpayX | Yes | Yes | Platform-based |
| Cashfree | No | No | Manual/self |
| PayU | No | No | Manual/self |
| Volopay | No | No | Manual/self |
| Decentro | No | No | Manual/self |
| IDFC Bank | No | No | Platform-based |
| HDFC Bank | No | No | Platform-based |

Checked with a few banks as well. They have Payout Flow and TDS Payment Flow, but these are not interlinked. Filing support is also not provided by banks.  
Organisations still have to calculate and deduct TDS based on the applicable category, consolidate the data, and then make the payment. In addition, for filing, they need to track challans and each transaction, and then proceed with filing for their orgs.

No competitor provides comprehensive automated TDS Payment and Filing functionality. Most offer partial solutions or managed services, creating a significant competitive opportunity.

# 5\. How will we solve this problem?

## 5.1. Our proposed solution and the various phases/ milestones involved

[**Link**](https://docs.google.com/spreadsheets/d/1UYfukEuFjYNRCjgRbmIUVdXu6j7LhdI1ZttLeZuZjV4/edit?gid=0#gid=0) to detailed user Stories

**Overview**

The Compliance team will be exposing Compliance Service APIs that enable other product teams to automate TDS payment and filing processes.

As part of the vendor payout flow, BB+ will integrate its existing workflows with the Compliance APIs to support TDS payment and filing. This integration will ensure that customers no longer need to perform any manual steps related to TDS compliance.

The Compliance team will provide an API that allows BB+ to send transaction-level TDS payment and filing details. Using this data, the Compliance Service will:

1. Generate the CBDT file and share it with the Ops team for TDS payment.  
2. Prepare the TDS filing file (Form 26Q) and process the filing accordingly.

Once the payment and filing are completed, the Compliance Service will trigger a webhook to notify BB+ with the following details:

* CIN & BRN   
* Filing acknowledgement PDF

BB+ will then display this information to customers on their dashboard, providing complete visibility into their TDS payment and filing status.

**TDS Payment and Filing Setting Enablement for BB+ Customers**

In this section, we will discuss how customers can enable these settings from the dashboard.

#### **TDS Payment :** 

Merchants can enable the setting from the **Settings** section of the BB+ dashboard.  
BB+ already supports TDS Payment, and this setting is currently available on the dashboard. We will continue to use the same setting.

From the date the setting is enabled, the BB+ team must deduct TDS, call our Compliance API, and share details at each transaction level.

If the setting is enabled in the middle of the month, TDS deduction will start from that point onward.

For I2P customers for whom the TDS setting is already live, and we have committed that by the end of the month we will check if their setting is enabled — even if they have enabled it mid-month — we will consider all the transactions for that month.

Note : 

1. For TDS payment we need the TAN number of the merchant and we also need to ask the merchant from which bank they want us to deduct the TDS amount.  
2. Apart from that when they are enabling the payment setting, we also need to inform customers that the TDS amount will be deducted by the 4th, so we need to make sure the TDS amount is available in their account by the 4th.

#### 

#### **TDS Filing :** 

This is a new capability being introduced for BB+ customers. BB+ must expose a new setting on the dashboard to allow organizations to enable this themselves.

Once customers enable the **TDS Payment** setting, the **TDS Filing** option will also be shown.

In BB+, once the TDS Filing setting is enabled, organizations must select the period from which the filing should be applicable:

* **From the start of the next quarter:** Data will be considered only from the beginning of the next quarter.  
   *Example:* If the user enables this setting on 25th June, data will be considered from 1st July, and we will file for Q2 (July–September).  
* **From the current date:** Transactions will be considered from the date the user enables the setting.

After selecting the transaction period, Orgs will also have to provide the IT portal credentials and Traces credentials. IT Portal and Traces credentials are used for TDS filing and for generating Form 16A respectively. This will be a mandatory step for the organization. In case Orgs disable the setting in future, we will still store the credentials entered for one month and then delete it from our db. Next time when they enable the setting that time they will have to select the setting again.

Once the details are filled and saved, we will do credential verification and in case the credentials are not correct, we will send an email notification to customers informing them about this and asking them to enter the correct credentials from the settings page. 

Note : Credentials verification happens after some time(after 15-30 minutes) and not instantly.

A disclaimer must be shown on the dashboard stating that if there is data prior to enabling the setting, they may receive notices from the government portal. Razorpay recommends that organizations enable the setting from the start of a quarter.  We also need to inform customers when they are enabling settings, that if they have done any transactions outside Razorpay, they can use the import functionality to import the data and do the filing correctly.

#### **New Organizations**

New organizations can enable these settings via the dashboard. Once enabled, the Compliance team will handle the filing for these organizations as well.

### **Important Notes**

* Only TDS Filing support is not possible. Organizations can either enable only **TDS Payment** or both **Payment and Filing**.  
* Merchants can enable or disable the **Filing** setting at any time. The Compliance service will check the filing status on the **last day of the quarter**, lock that status, and proceed (or not proceed) with filing based on it.  
* If organizations want us to handle or not handle their TDS filing, they must enable or disable the setting before the last day of the quarter.  
* For **TDS Payment**, if the setting is disabled, **Filing** will automatically be disabled as well. However, if TDS has already been collected, we will still make the payment to the government. This message will be clearly displayed to customers to avoid confusion.  
* If merchants enable the **TDS Filing** setting from the BB+ dashboard, we will consider all their transactions done via RazorpayX, whether payouts are made via BB+ or through Payroll for contractor payments, to ensure this does not become a blocker for such organizations.  
* If the filing has already been done outside Razorpay, we will not handle the revised filing for these merchants.  
* What If orgs TDS money was deducted but they don't want us to make the payment:  
  * In this case, they will have to reach out to our support team who will then inform our Ops team about it and Ops will then move the transactions to deferred and will not do their payments.  
  * In initial phases, these settings will be behind a feature flag

Currently, we have an existing settings page for the Vendor Payouts TDS flow where we have the TDS payment settings. Until we merge the data for both the Payout and Vendor flows, we will have two different dashboards. Listing down all the use cases and how they will be handled:

1. Org has only Vendor Payout flow enabled and not the new TDS flow: In this case, they will see only the old settings dashboard and not the new one. No change here.  
   1. \`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`\`												Org has both Vendor Payout flow enabled and the new TDS flow enabled: In this case, they will see two dashboards : both the new and the old settings dashboards. This will be applicable only until the data is merged from both the sources.  
2. Once the data is merged for both the Vendor Payout flow and the new TDS flow, we will have only one settings dashboard, which will be the new one and old one will be removed. Listing down all the cases after the data is merged:  
   1. If an org was already using the Vendor Payout flow for TDS payment but is not using TDS in the Payouts flow: We will migrate these settings to the new one. TDS payment will be enabled by default with all the configurations prefilled based on what the org had previously configured.  
   2. If an org is using both the Vendor Payout flow and TDS in the Payouts flow: In this case, we will enable the TDS payment setting and migrate the existing vendor settings and configurations for existing Vendor Payout flow customers. For new customers, they will need to enable the TDS payment setting and configure it (invoice date/payment date).  
   3. If an org is only using TDS in the Payouts flow: In this case, the org will enable the setting, but the Vendor Payout flow configuration will not be set. We will inform the customer to configure it so that they can use it in the Vendor Payout flow in the future.

**How the setting will be checked by Compliance team** 

To BB+ customers, we will always present this as an automated solution — they do not need to log in to the dashboard or trigger anything for TDS Payment and Filing. They only need to enable the setting once, and we will take care of both payment and filing before the due date.

**Compliance Service Requirements:**

* Compliance service will need a trigger to identify the current setting configuration for each org.  
* Compliance must fetch the filing setting from the BB+ team.

Important Notes:

* BB+ must share both:  
  1. The setting status (enabled/disabled).  
  2. The timeframe chosen by the user (current month vs. next quarter).  
* Based on this input, Compliance will decide whether filing should start in the current quarter or the next.  
* For TDS Payment, if we already have the data for the current month, Compliance will process the payment.  
* Only for TDS Filing, this setting check is mandatory.

**TDS Payment \- Money movement from user current account to Razorpay Compliance account**

Once the payment setting is enabled on the partner’s dashboard, BB+ must deduct TDS as per government regulations and transfer the deducted TDS amount to the Compliance account.

BB+ will use the Payout API to move the TDS amount from the customer’s account to the Compliance account. Once the amount is transferred, BB+ will receive a reference Payout ID, which must be shared with us as proof of payment when calling our Payment API.

If the Payout ID is not available, compliance will not be able to process that information on their side.

In the I2P flow, for the existing customers, we can follow the same deduction approach that we currently use — deducting TDS on the 4th and then making the payment. However, instead of deducting the amount on the 4th, we should move it to the 1st, as we now have enough time to inform customers and collect funds accordingly.

TDS deduction will always happen every month. We will start sending comms to customers informing them about this and they need to make sure that they add the money by 4th. Also we will continue recovering this amount till 4th and post that customers will have to manually handle the TDS payment.

If we are unable to deduct the payment due to low balance, automated communication must be sent to customers informing them about it so they can load funds, enabling us to handle their tax payments. We need to send three email reminders for this.

**Money deduction flow**

We will inform customers to load the TDS amount by  4th of every month. Starting from the 2nd of the month, we will begin deducting amounts from customers to ensure there is sufficient time to process all payments smoothly. Money deduction will happen in bulk, which means that we will check what the TDS amount needed for that month is and if it is available then we will deduct the amount from the selected customer account. If not, we will send a reminder email.

Customers whose funds we have not collected will start receiving reminders in advance, informing them that the deadline to load funds is the 4th of the month. While deductions may begin from the 2nd, the final cutoff for ensuring sufficient balance remains the 4th.

If an organization is unable to load the required funds by the 4th, they will have to complete the TDS payment manually outside the system.

In cases where customers approach us after the 4th and request assistance with the payment, there should be a way for our Ops team to process these transactions manually. In such cases, customers must first transfer the required amount to our escrow account, after which Ops can move the transactions and complete the payment on their behalf.

**Money Reversal cases**

1. TDS deduction from the customer account is not done yet and payout reversal happens.  
   1. In this case, as soon as the reversal happens, BB+ needs to inform the compliance service about it and Compliance will not deduct the amount from the customer account and transaction IDs will not move into the Compliance team bucket.  
2. TDS deduction was done from the customer account, money was not paid to the government and payout reversal happened.  
   1. In this case also, as soon as the payout reversal happens, BB+ needs to inform the compliance service about it. Compliance will then have to make sure that the TDS payment to the government is not done for the corresponding payout and the TDS money is refunded back to the customer account.  
3. TDS deduction was done from the customer account, money was paid to the government and payout reversal happened.  
   1. In this case, BB+ will inform the Compliance service and then the Compliance team needs to inform Ops/Support who will then have to manually recover the amount from the customer account.

**LDC handling** 

Currently, LDC (Lower Deduction Certificate) handling is available in the Vendor Payments flow. When orgs add vendors, they select the applicable TDS category and upload the LDC certificate. This certificate allows orgs to deduct TDS at a lower interest rate approved by the government instead of the standard rate.

When we take the TDS flow live in the Payouts flow, we need to provide the same LDC functionality in the Add Contacts flow as well. This ensures that orgs that do not use the Vendor Payments flow can still avail the LDC feature.

In the Add Contact Page, we will give an option to add LDC details where merchants will have to add LDC details for each vendor.Each Vendor can have multiple certificates for multiple categories. We need to collect the following information from merchants : LDC certificate, Certificate number, TDS category for which it is applicable, Reduced rate, validity period and Maximum amount limit.

For vendors where an LDC is applicable, once the organization selects the TDS category, there will be an option to select whether LDC is applicable or not. If yes they the system should calculate based on the reduced interest rate as defined by the government instead of the standard rate. The TDS calculation should also be clearly shown to the organization using this lower rate. Additionally, when sharing transaction data with the Compliance Service, the LDC certificate must be included for all such transactions, as it is required for TDS filing.

Notes: 

* LDC will be provided to a vendor for a specific category. It is possible that one merchant can have multiple certificates for different TDS categories.   
* Every certificate issued will expire by the end of the financial year. 

**Mandatory PAN number for Vendors**

For TDS filing, the PAN number of the vendor/deductee is mandatory. Therefore, it is important for us to capture it during the payment flow.

Listing how this will be handled(Required only when TDS Payment setting is enabled):

1. If an organization has enabled the TDS filing setting, then while making payouts to a vendor, we will check whether the PAN is available for that vendor or not.  
   1. If yes, we will show all TDS categories and allow customers to proceed with the payment.  
   2. If not, we will not allow them to select TDS categories with an interest rate. They can only select the No TDS option and proceed.  
2. To select other TDS categories, we will show a note to customers stating that the PAN is not available for this vendor. Please add the PAN to view and select other TDS categories.  
3. From our side, we will check the Type of the vendor selected by the customer in the payout flow. If the Type is Vendor, then in the note we will mention that the customer should go to Vendor Onboarding and update the vendor’s PAN number. If the Type is not Vendor, then we will ask them to update the PAN from the Contacts page.  
4. Similarly, we will also add a link to the respective page (Vendor Onboarding or Contacts) based on the Type, for easy access.

**TDS Selection on the dashboard by the user** 

Currently, on the BB+ dashboard, the TDS option is available only in the I2P flow, where users can select the TDS category while creating an invoice. Based on this selection, BB+ handles the TDS payment automatically if the user's TDS setting is enabled.

We will now extend this same approach to other flows such as Single Payout, Bulk Payout, and Payout via Links.

Below is the detailed experience for each flow:

#### **Single Payout Flow**

In the Single Payout flow, we will now provide a TDS category dropdown to customers, similar to what already exists in the normal payout flow.The flow remains largely the same, except that in the amount entry step, customers will now see a TDS category dropdown along with the corresponding interest rates. Based on the user’s selection, we will calculate and display the total amount, vendor payout, and the TDS amount to be paid to the government portal.

**Updated Flow:**

1. User selects the vendor Contact  
2. We check if the TDS Payment setting is enabled or not.   
3. Selects the Destination Account of the vendor  
4. Enters the Total Amount  
5. We check if the PAN number is available or not for the selected vendor.  
6. If yes we show all the TDS categories. In No then we show only the No TDS category option in the drop down and we show a note telling that Please add PAN for the selected vendor. [Refer this](#bookmark=id.fvlbt7737b0q) for more details  
7. Selects the TDS Category on the same page  
8. Customers can select whether the LDC is applicable or not. If yes, they need to ensure that an LDC certificate has been added against the selected vendor.  
   1. If an LDC is present, we will calculate the TDS amount based on the rate mentioned in the certificate.  
   2. If there is no LDC, we will calculate TDS using the normal TDS category rate and show a message to the customer stating that the LDC certificate was not found. The customer can either add the certificate and then make the payout or proceed with the existing option.  
   3. When the customer selects LDC applicable or not we will show a Note : Please make sure that it is within the threshold limit.  
9. Reviews the Amount Bifurcation and fills in extra details  
10. Enters OTP  
11. Makes the Payment

Once the payment is completed, the total amount is split into two parts:

* The vendor amount credited to the Vendor Account  
* The TDS amount credited to the Compliance Account(1st \- 4th of month)

#### 

# **TDS in Bulk Payout Flow**

Pending Points: 

1. Approval Flow Summary \- pending & Processed

Bulk payouts flow is used by BB+ merchants to make payouts in bulk. Our plan is to add support for TDS payment in the bulk flow as well.

## **Template Changes**

If the TDS Payment setting is enabled, the template that we give to customers to download should include TDS fields. If the TDS Payment setting is disabled, we should give the existing template with no TDS fields.

In the updated template, we will provide the following additional options:

### **Beneficiary Templates**

| Field | Description |
| :---: | :---: |
| TDS Category | A dropdown option which will show a list of all the TDS categories(with one liner) that we support. |
| Is LDC Applicable | Dropdown option (Y/N) |
| LDC Certificate | If LDC is applicable, merchants need to enter the LDC certificate number. |
| LDC Rate | LDC rate (in %) that will be applicable for that payout — should be entered by the user. |
| PAN Number | PAN number of the vendor — required for TDS payment. |

### **Fund ID Templates**

| Field | Description |
| :---- | :---- |
| TDS Category | A dropdown option which will show a list of all the TDS categories that we support. |
| Is LDC Applicable | Dropdown option (Y/N) — A note will be shown that whatever LDC was defined against the vendor in the system will be applied; otherwise, the rate based on the selected category will be calculated. |

**Note:** For all Fund ID templates, we don't need to give the following columns: LDC Certificate, LDC Rate, and PAN Number of the vendor.

We also show a Note to customers telling that TDS updated calculation will be shown as soon as you import the file.

### **Notes to Show When Downloading the Template (when TDS Payment setting is enabled)**

* If you are selecting any TDS category except "No TDS", then PAN of the vendor is mandatory.  
* If you are selecting LDC Applicable as "Y", then LDC Certificate and LDC Rate are required.  
* In case of Fund ID templates, please update the PAN and LDC details before making the TDS payouts; otherwise, the payout may fail.  
* Once the file is imported in the system we calculate the TDS amount based on the category, LDC and applicable rate. Request you to review the file before proceeding with the payment. 

## **Validation While Importing the File**

All existing validations performed in Bulk Import will continue in this flow. On top of that, we will perform additional validations for the TDS flow. Validations will differ based on the template type. Templates can be divided into two major cohorts:

1. Fund Account Template — where payment is made to an already added fund account.  
2. Beneficiary Template — where fund account ID is not present and beneficiary details need to be entered.

### **Beneficiary Template Validations**

* Ensure that merchants have selected only those TDS categories that were available in the dropdown.  
* If they have selected a TDS category with a value other than "No TDS", then they must provide the PAN of the vendor.  
* If they have selected LDC as applicable, then they must provide the LDC Certificate number and LDC Rate.

### **Fund ID Template Validations**

* Ensure that merchants have selected only those TDS categories that were available in the dropdown.  
* When a customer imports the file and a TDS category is selected, we also check if the PAN for that Fund ID is available in our system. If not, we throw an error.  
* If they have selected LDC Applicable as "Yes", we check if the LDC Rate and Certificate are mentioned against that contact in the Contacts tab. If not, we throw an error that LDC details are not mentioned against the vendor.

**Note:** In case of Fund ID templates, if LDC is applicable, whatever LDC rate is defined against that contact under the Contacts tab will be applied automatically.

## **Error Handling in Bulk Flow**

Errors will be handled in the same way as they are currently. There will be one additional column added against each entry, and in that column, we will show the error applicable for that particular row.

If there are multiple errors for a single row, they will be shown in the same field separated by commas.

**Bulk Payout list view \- Approval and Processed flow**

1. In the bulk payout list view, we will have to add one more column, which is the TDS amount that was deducted and paid to the government from the customer account. Beside total Amount we can add a tooltip which says that this amount was paid to your beneficiary.  
2. Apart from that, in the right side panel and in the Approval pop-up, we have to show a summary where we can display all the following information to the customer: Total Number of Payouts, Number of Payouts for which TDS is applicable, Total Payout Amount, Total Amount paid to the beneficiary, and Total TDS Amount.  
3. Also, in the right panel, we will provide the Excel file with the TDS calculated amount (if applicable), so that whenever customers want, they can verify the data. 

## **Final User Journey**

1. User clicks on Bulk Payout flow.  
2. Option to download the template or drag and drop the template.  
3. User clicks on Download the template.  
4. We ask them to select the payment method and the type of file.  
5. If TDS is applicable then the downloaded template will have TDS options. If not then the existing one with no TDS option will be downloaded.  
6. Once selected, we show the Notes/Info to the customer.  
7. Customer downloads the file, adds the data, and re-uploads the file.  
8. We perform the import validation.  
   * If there are any errors, we give the error file to the customer and ask them to download the error file and re-upload the corrected file.  
   * In the error file, we list down all the errors against each row.  
9. Once the file is processed without any errors, there will be a preview option where we show some rows and TDS calculation to the user.  
10. Below the preview option, we will show Total Number of Payouts, Number of Payouts for which TDS is applicable, Total Payout Amount, Total Amount paid to beneficiary and we also have to show Total TDS Amount.  
11. Apart from this we will also give a Download report button with the calculated TDS amount so that customers can download the file, check and validate the amount if needed.   
12. After reviewing the information, they will click on next where we will take a confirmation if TDS Payout is applicable asking customers hope they validated the tax amounts.  
13. In the next screen, we ask the user to enter other information like Purpose, Debit From, etc.  
    * Under Debit From, we have to add a note telling that this is only for Payout purpose. For TDS, whatever account the user has selected in Settings — from that account it will be deducted.  
    * Also show a note that TDS will be deducted from 1st–4th of the next month.  
14. Then complete the payment.  
15. After payment is completed, we need to add these contacts under our Contacts tab so that the customer can use them for future payouts.

#### 

#### **Payout via Links – TDS**

When merchants create Payout Links, they can now select the TDS category during link creation. Based on this selection, the system will automatically calculate and display the TDS deduction and net payout to the customer when they access the link.

**Flow:**

1. Merchant selects the vendor to whom the payout will be made.  
2. Merchant enters the phone number and email ID of the vendor. (If the phone number is already available, it will be prefilled but editable.)  
3. Merchant enters the amount and selects the TDS category from the dropdown on the amount entry page.  
4. Merchant enters additional details, verifies via OTP, and sends the payout link to the vendor.

When the vendor opens the payout link, all details remain the same as before, except that the system will now also display:

* The TDS amount to be deducted  
* The net amount the vendor will receive

After the vendor confirms the details, the payout is initiated.Once the payment is completed, the total amount is split into two parts:

* The vendor amount credited to the Vendor Account  
* The TDS amount credited to the Compliance Account

**Data Transfer \- Compliance API call**

TDS Payment API details \- [Tech Specs link](https://docs.google.com/document/d/1YtXCh2wxjELNi8Dh4tIE8S2canbqd2jpbipVEBM28nI/edit?tab=t.0#heading=h.p315c7rwre7v)

Once the TDS payment setting is enabled and TDS payment is moved to the compliance account, then for each transaction, BB+ will have to call the API and share the information with us. The compliance service will store it at their level and use it to prepare the CBDT and TDS Filing file. 

If Both TDS payment and Filing is enabled, then only we need the Deductee details else we don't need those details. But BB+ team should share both Payment and Filing details with us.

**Filing** 

Apart from deductee details mentioned above, BB+ also needs to capture IT portal credentials and share it with us. BB+ team needs to capture and store this in Credentials Encryption Service provided by Compliance team \<Credentials Encryption Service details to be added here by Compliance Tech Team\>

Compliance will fetch the credentials from that service and proceed with the filing. Whenever users update their credentials in the dashboard, BB+ team will have to update the same in our Credentials Service. Compliance will fetch IT portal cred directly from the credentials service. 

**Data that needs to be shared with Compliance service**

| Entity | Mandatory | Description |
| :---- | :---- | :---- |
| source | M | Originating system of the payment request (BB\_PLUS or PAYROLL). |
| organization\_id | O | Organization\_id from payroll |
| merchant\_id | O | Merchant id from payout |
| TAN | M | Tan of a merchant |
| PAN | M | PAN of a merchant. (Not mandatory) |
| Name | M | Merchant name |
| Ppayment\_details.deduction\_date  | M | Date on which merchant has made a transaction. In payroll’s case it can be transaction date or payroll month |
| payment\_details.source\_transaction\_id | M | payout\_id |
| payment\_details.major\_head | M | Mode of payment |
| payment\_details.minor\_head | M | Mode of payment |
| payment\_details.category | M | Category will define if its salary payment or non salary and also if non salary then which type of vendor payment |
| payment\_details.basic\_tax\_amount | M | Base TDS amount without any penalty and charges |
| payment\_details.interest | M | Interest occurred on TDS payment |
| payment\_details.surcharge | M | Surchange associated with TDS payment |
| payment\_details.cess | M | Cess applicable to the TDS payment |
| payment\_details.penalty | M | If there is any penalty for TDS payment |
| payment\_details.total\_tds\_amount | M | Total amount need to paid for the TDS payment |
| deductee\_details.deductee\_name | O | Name of the vendor |
| deductee\_details.deductee\_pan | O | Pan of the vendor |
| deductee\_details.contac\_id | O | If deductee\_pan and name is mentioned then contact\_id of vendor need to be shared which can be used to consume the data  |
| deductee\_details.people\_id | O | If deductee\_pan and name is mentioned then people\_id of payroll user need to be shared. Not applicable for BB+ source. |

**Note:** For filing all the details are mandatory even those marked as O so we have to share all the info with compliance service. For more details please refer to the Compliance tech spec 

**Import flow for Orgs**

For TDS Filing, we will provide an Import option that merchants can use to upload transaction information carried out outside the Razorpay system, along with the corresponding TDS payment details.

The import functionality will be available only for TDS filing and not for making payments.

To import historical data, we will provide a [downloadable template](https://docs.google.com/spreadsheets/d/1HXUuKikoWM9r9OaeBjqyXuNaKpXaFXe1Dwt-M_C_2TE/edit?gid=9191253#gid=9191253) that customers can download and use to upload data strictly in the specified format.

**Listing down the user journey for reference:**

* The user selects Import Data from the TDS Filing Dashboard.  
* We inform the user about the overall process and what is expected from them.  
* The user is then asked to select the date range for which they want to import data.  
* An option is provided to download the template so that users can download the file and add data in the prescribed format only.  
* Once the data is added, the user can return and upload the file into the system.  
* In case of any errors, we show an uploaded failed status and provide an error file that the user can download, fix, and reupload the new one.

**How the error handling will be done:**

* We will add an additional column in the Excel file that mentions the **reason for the error**.  
* The user can view the reason in this column, correct the issue, and reupload the updated file.

There will be certain validations performed during data import to ensure the accuracy of the information being added to the system:

* The TDS amount should be correct based on the payout amount and the selected TDS category.  
* If LDC is applicable for a transaction, the merchant must mention the applicable rate, and the TDS amount should be calculated based on the entered rate.  
  * In this case, the LDC certificate will be mandatory.  
* The transaction data added in the file must fall within the date range selected by the user during import.

**List of errors that can occur:**

* TDS amount is incorrect based on the selected category.  
* PAN format is invalid.  
* Some data falls outside the selected date range.

Once the file is successfully imported into the system, the corresponding contacts will also be added under the Contacts tab. Unique contacts will be identified based on PAN number.

In the TDS Filing Dashboard, where transaction-level data is displayed, the imported transactions should also be shown, clearly indicating that these transactions were imported from outside the system.

**Data Transfer \-  Issue in the API call**

Every time the BB+ team calls our Payment API, we will send a response indicating the status of the data transfer.

**Possible States:**

1. **Success** – Data transfer is completed successfully.  
2. **Failure** – Data transfer failed due to an issue (error message will be shared by the Compliance team)  
3. **Pending** – Data transfer is still in progress.

**Failure Handling:**

* If there is an issue at BB+ end, they will receive a Failure status along with the relevant error message.  
* The BB+ team must fix the issue and re-share the data with us. In case of failure, there will mostly be 2 types of errors:  
  * System error: This might occur due to technical issues. In such cases, the BB+ team needs to retry and send the data back to us again.  
  * Data-related error: These errors occur only when some data in the request body is missing. Example: If PAN was missing at the time of payment, we will show the error to users on the dashboard. They can then fix the error, and once corrected, BB+ can retry and send the transaction to us again.  
* For data-related errors, we will show the error messages to merchants on the BB+ dashboard so that they can fix the issue on their side. These will mostly be data-missing errors. Once the data is updated by the merchants, the BB+ team can share the corrected data with us by calling the compliance API again.  
* Once the data is successfully received, we will initiate the payment at our end.

**CBDT file shared & payments done by Ops team**

Once the data is shared with the Compliance team, they will make sure that Payment is done before the due date. 

We will share the file with the Ops team that they can download from the Ops dashboard. We will be creating a new dashboard from which ops can download the file and do the payment. The process for Ops will be the same as they are doing it right now but the dashboard page will be different because the data source is different right now.

Once the Ops team does the payment, they will upload the success response along with the CRN number back to the dashboard.

Compliance will process the information like CRN number and BRN number and share the same with BB+ team via webhook.

The cutoff date for BB+ to send the data will be 4th of every month.

**Payment & Filing Status Check**

There will be 2 separate webhooks for Payment and Filing status. BB+ team will have to use both webhooks to show the status on the dashboard to their users.

In BB+ there will be 2 dashboards, One to show status for TDS Payment and other for TDS filing.

**TDS Payment confirmation**

Once the TDS payment is done by the Ops team, they will upload the CIN & BRN numbers on the Ops dashboard. The Compliance team will then process these details from the Ops dashboard and share the success status with the BB+ team. The BB+ team will have to display the same to customers on their TDS Payment dashboard as proof.

Currently, compliance will only share CIN and BRN numbers with customers, but in the future, compliance will also share the challan PDF with them. 

In the existing Tax Payment dashboard, users currently have the “Mark as Paid” option, which allows them to make TDS payments offline and we skip it. Going forward, we will remove this option. We will clearly communicate to customers that if they choose to handle TDS payment and filing through Razorpay, we will take care of all their payments and filings. They should not make any TDS payments or filings offline once these settings are enabled.

**Note:** Razorpay has to make sure that TDS payment is done before the due date.. If we have the data before the cutoff date and there is no issue in the merchant data, then it's Razorpay's responsibility to complete the payment by the due date. Same for Filing also. But in case there is some issue in the customer data then we will be sending alerts to customers informing about the same and if they dont fix it by the due date then its customer responsibility.

**Filing data on BB+ dashboard**

There will be a new dashboard where we will show the filing data to customers, we will also show the filing status and acknowledgement pdf once the filing is done.

BB+ team will also have to show the filing data to customers on the dashboard so that they can check or verify the data in case of any issue or for reference purposes.

Once the filing data is processed by the Compliance team, the Compliance team will send a .txt file to the BB+ team with all the filing data for that organization. The BB+ team will have to fetch the data from the .txt file and display it to customers on their dashboard.

Attaching a screenshot of what we are currently showing on the Payroll dashboard. The same information can be displayed on the BB+ dashboard.

![][image1]

![][image2]

In BB+, we can show a similar dashboard where, if a user wants to view the filing data, they can click on the Filing Data field. This will redirect them to a new dashboard with a detailed view, as shown above.

**Filing success confirmation**

Once the filing is completed successfully, the Compliance team will send the success response to BB+ via webhook along with the Acknowledgement PDF. The BB+ team will have to download this and display it on the dashboard so that users can view and download it themselves, similar to how it is shown in the above dashboard.

**Error while doing filing**

Listing below how we will handle errors in case of TDS filing. The errors that we receive during TDS filing will be shown both to the users and also to the Ops team.

* For users, the error will be shown on the BB+ dashboard.  
* For the Ops team, the error will be shown on the Ops dashboard.

Adding more details below on how we are going to handle it:

There are 2 types of errors that we get during TDS filing:

1. System Error: This might occur due to technical issues. In such cases, we will show the error to the users that there is some technical error. The compliance team will retry again from their side.  
2. Data-related Error: These errors occur when the data provided at the transaction level is incorrect and validation fails.  
   * *Example:* Invalid PAN.  
   * Errors can occur in both merchant and vendor data.

Whenever Compliance services encounter any error while filing the TDS return, they will share a .txt file with the BB+ team highlighting the error. The BB+ team will consume this file and display the error message to their customers on the dashboard so that they can review and fix the issue. Customers can then correct the data-related error on their side.

We will have to send alerts & reminders to customers informing them about these issues so that they come and update the details before the deadline

**TDS Filing Errors and how customers will have to solve that**

| Error List | Solution |
| :---- | :---- |
| Missing PAN number of Org & Vendor | Add the vendor Pan in the dashboard |
| Invalid PAN format of Org & Vendor | Update the vendor Pan in the dashboard |
| Invalid Credentials | Update the IT Portal credentials in the dashboard |
| Missing/invalid vendor names | Update the Vendor name from the dashboard |

**How Compliance will get the updated data**

Once the data is updated by the customer on the dashboard, Compliance service will have to get the updated data so that they can reinitiate the filing for those orgs. 

**How Ops will see the status, filing data, and errors**

We will be creating a new dashboard for the Ops team within the Payroll dashboard where they can check all pending orgs along with the errors encountered.

Ops can use this information to get the data fixed from customers or ask customers to update the data themselves. These corrections should ideally be done by customers on their dashboard to avoid any issues. Once the data is updated on the dashboard, the cron set by the compliance team will automatically fetch the updated data.

In case customers are not able to fix the data via the dashboard, the Ops team themselves can reach out to customers and update the data in the dashboard.

**Note:** To support manual filing in cases where an issue is not getting resolved or a customer has escalated, we will need to handle filing for such orgs manually. This may also be required if: The customer has other data that also needs to be filed or our services or Quicko’s services are down.

In such cases, we will have to provide the filing data to the Ops team on their dashboard so that they can complete the filing manually for those orgs.

**TDS Payment Dashboard**

This dashboard will be used by the Ops team to make TDS payments for BB+ users.

In the payment dashboard, we will show all BB+ org data that has opted for TDS payment, along with their TDS details. The dashboard will also provide options for the Ops team to download the CBDT file, complete the payment, and update the payment status along with challan details.

There will be 4 dashboard states for TDS payment: **Pending, In Progress, Success, and Deferred**.

### **Pending Dashboard**

In this dashboard, we will show all orgs that are eligible for TDS payments. Once we receive the TDS amount from the customer’s account, those transactions will move to the Pending state, which means Ops can proceed with making the TDS payments.

This dashboard will have a list-view table showing all orgs along with their payment details. One org can have multiple rows based on the number of TDS categories it has. For each TDS category, there will be a separate row in the list.

The table will display the following details:

* Org ID / Merchant ID – Unique identifier used in BB+ for identifying merchants  
* TIDs – Transaction IDs of that organization and of that TDS Category  
* Name of the merchant  
* TAN of the merchant  
* Address of the merchant  
* Type of TDS (Salary / Non-salary)  
* Purpose – TDS category  
* Payout month  
* TDS amount  
* Interest

In this dashboard, we will provide 4 filters to help the Ops team make payments faster:

* Chunk size – By default, it will be 200, but Ops can select a chunk size of their choice  
* Assessment year  
* Choose the bank for which the CBDT file needs to be generated – Axis / IDFC  
* Option to select “Calculate Interest payment also”  
  * If selected while generating the file, wherever TDS interest is applicable, it will be calculated and added to the CBDT file

There will be a **Generate CBDT File** option using which Ops can generate CBDT files for making TDS payments. Ops can enter values using the above filters and then click on **Generate CBDT File**.

Once they click on **Generate CBDT File**, based on the selected chunk size, that number of entries will be added to the CBDT file. A unique batch with a unique identifier will be generated.

The CBDT file will be downloaded in the system for the Ops team to make the payment, and the rows for which the CBDT file was generated will automatically move to the **In Progress** state. Ops do not need to move them manually.

In the Pending screen, against each row, there should also be an option to move the entire row to the **Deferred** state. When moving to the Deferred state, Ops will be asked to enter the reason for deferring it.

### **In Progress Dashboard**

Once a CBDT file is generated in a chunk, those transactions will automatically move to the **In Progress** state.

In the In Progress dashboard, we will show batch-wise data, allowing the Ops team to view all transactions that are part of a batch.

In this dashboard, the Ops team will have multiple action buttons. These include:

* **Upload payment confirmation Excel**

  * This will be an Excel import that Ops receives from Axis Bank, containing payment and challan details.  
  * Ops must ensure that the correct file is uploaded against the correct batch.  
  * Once the file is uploaded, the system will perform amount validation, TAN validation, and category validation to ensure that the uploaded file data is correct.  
  * If there is a mismatch, the system will throw an error message, and Ops will need to fix the issues and re-upload the file.  
  * Once the file is successfully uploaded, the selected batch will move to the **Success** state.  
* **Move to Deferred state**  
  * This CTA can be used to move an entire batch or an individual transaction within a batch to the Deferred state.  
  * This option will be available at both the batch level and individual transaction level.  
  * On clicking this option, the assigned batch (unique ID) will be deleted, and the selected batch (along with all corresponding transactions) or individual transaction will move to the Deferred state.  
  * While moving to the Deferred state, the Ops team must mention the reason for deferring it.  
* **Regenerate CBDT file**  
  * If, for a batch, Ops has moved some transactions to the Deferred state, they can regenerate the CBDT file and download it again at the batch level.

### **Deferred State**

All transactions that are manually moved by the Ops team will appear in this dashboard.

In the list, we will also show the reason why the transaction was moved to the Deferred state.

For payout reversal cases where we receive a message from the BB+ team not to proceed with payment before paying to government, those transactions will also move to the Deferred state with the reason mentioned as requested by the BB+ team.

For all transactions in the Deferred state, there should be an option to reverse the money back to the customer’s account.

**Note:**  
In cases where money reversal is identified before deducting TDS, the transaction should be marked as **Failed** and does not need to be shown in the Ops dashboard. It can simply be shown with a **Failed** status on the TDS payment dashboard.

**Failed / Cancelled State** 

All transactions that are manually cancelled by the Ops team will be moved to this dashboard.

Also if a payout reversal occurs after the Ops team has already made the payment to the government, those transactions should also move to the Failed state with the error reason: Payout Reversal. For such cases, an alert must be sent to the Ops team’s email ID so they can initiate recovery of the amount from the customer.

**Command to move transactions to deferred or cancelled state**

There should be a command for the Ops team where they can move a particular transaction to Deferred and cancelled state. 

If a transaction is moved to a cancelled state by the ops team then the TDS amount deducted from the customer will be refunded back to the customer. When cancelling the transaction Ops also need to mention the reason for refund and the same reason will be reflected in the Failed/cancelled Dashboard.

## **TDS Filing Dashboard**

Similar to the TDS Payment Dashboard, we will build TDS Filing Dashboard so that the Ops team can perform TDS filing for BB+ customers.

For TDS filing, there will be 3 dashboards created for the Ops team: Pending, In Progress, Success

### **Pending Dashboard**

In this dashboard, we will show a list of all eligible orgs for which the Ops team needs to perform TDS filing. This will be a list table view displaying the following details:

* Org ID / Merchant ID – Unique identifier used in BB+ for identifying merchants  
* Name of the merchant  
* TAN of the merchant  
* Address of the merchant  
* Quarter  
* Financial year

In this dashboard, we will provide 4 filters to help the Ops team make filing faster:

* Org ID / Merchant ID – To search for a particular org or a group of orgs. This will filter out the relevant orgs so that Ops can select them from the list and click on the File TDS CTA.  
* Quarter filter – Select one or multiple quarters  
* Financial year  
* Chunk size – By default, this will be set to 50

There will be a couple of CTAs for the Ops team to handle filing in this dashboard:

1. #### **File TDS**

* This CTA will be available at both the org level and the overall level.  
* If Ops want to trigger filing for a single org, they can search for that org and click the File TDS CTA beside it.  
* If Ops want to trigger filing in bulk, they can use the overall-level File TDS CTA.  
* Some example scenarios and how they will be handled:  
* Ops want to trigger filing for all orgs in Q3 2026\.  
  * They can select the quarter, financial year, and chunk size (default 50\) and click File TDS.  
  * The system will pick 50 orgs at a time, execute filing, and then move to the next batch of 50\.  
* Ops want to trigger filing for only 100 selected orgs in Q3 2026\.  
  * They can select the quarter, financial year, chunk size, search for those orgs, select them, and click File TDS.  
  * The system will pick only those 100 orgs and execute filing for them.

2. #### **Download Filing Data**

* This CTA will be available against each org.  
* Clicking this option will download the filing data for that org.  
* This is intended for cases where automation is not working and Ops need to perform filing manually.  
* This option should be used only when manual filing is required.  
* On clicking this CTA, we will ask for a confirmation from the Ops team.  
* Once the data is downloaded, the org will automatically move to the In Progress state.

### **In Progress Dashboard**

During automated filing, if filing fails for any org due to any reason, those orgs will be moved to the In Progress dashboard along with the failure reasons. This allows the Ops team to fix the issues and retry.

Along with error reasons, we will add one more column to indicate whether the org was moved to the In Progress state by:

* Automation, or  
* Manual action by the Ops team

The same filters available in the Pending dashboard will also be available here. Additionally, there will be a Reason filter so that Ops can filter orgs based on specific error reasons.

There will be around 4 CTAs in this dashboard for the Ops team:

1. #### **Download .txt File**

* This CTA will be available at the org level.  
* When Ops click this for any org, the corresponding `.txt` file will be downloaded.

2. #### **Move to Pending**

* If there is an issue that cannot be fixed and Ops decide to proceed with manual filing, they can move that org back to the Pending state.  
* After moving to Pending, Ops can download the filing data and perform manual filing.  
* This option will be available against each org.

3. #### **Upload Acknowledgement (Bulk or Org Level)**

* Once filing is completed, Ops need to upload acknowledgements in bulk.  
* We will provide a CTA where Ops can upload a ZIP file containing all acknowledgement PDFs.  
* The system will identify acknowledgements based on TAN number and map them to the respective orgs.  
* This option will also be available at the org level, allowing Ops to upload an acknowledgement for a single org if required.

4. #### **Retry**

* Filing errors will be visible to both Ops and BB+ customers.  
* If customers fix the issue themselves and inform the BB+ team, the system will automatically retry filing. On success, the org will be removed from this dashboard and moved to the Success dashboard.  
* If Ops become aware that the issue has been fixed but the system has not auto-retried, Ops can manually click Retry.  
* Retry will be available at both the org level and the overall level.

### **Success Dashboard**

All successfully completed filings will move to this dashboard.

The same filters provided in the Pending dashboard will also be available here.

The same list view used in the Pending dashboard, with the same columns, will be used in the Success dashboard as well.

Against each org, there will be 3 CTAs:

* Ack PDF – To download the acknowledgement PDF  
* FVU file – To download the FVU file  
* .txt file – To download the .txt file

**List of emails that will be sent to customers** 

1. Reminder emails sent before the Last day of the month \- 4th of the month, asking customers to load the required TDS amount into their wallet.  
2. Email sent to customers whose funds were not received by the 4th, informing them that they must complete the TDS payment manually outside the system.  
3. Confirmation email notifying customers that the TDS payment has been successfully completed.  
4. Alert emails informing customers of any errors encountered during TDS filing, along with steps on how to resolve them.  
5. Reminder emails asking customers to fix pending TDS filing errors.  
6. Email informing customers that their credentials are invalid and requesting them to update the same on the dashboard.  
7. Confirmation email notifying customers that the TDS filing has been successfully completed.

**Versions wise Implementation plan**

We have checked the overlap data between major Payouts flow like Single Bulk & APIs. I2P flow data is pending, there is some instrument gap. Here's the summary of the data : 

| *Final* | COUNTA of merchant\_name | COUNTA of merchant\_name |
| :---- | ----- | ----- |
| Only Dashboard(Single Payout) | 873 | 40.12% |
| Only API & Dashboard | 669 | 30.74% |
| Only API | 314 | 14.43% |
| All API, Bulk & Dashboard | 225 | 10.34% |
| Only Bulk & Dashboard | 89 | 4.09% |
| Only API & Bulk | 5 | 0.23% |
| Only Bulk | 1 | 0.05% |
| **Grand Total** | **2176** | **100.00%** |

Based on the above data and also based on the customer conversations, we should go ahead with the below options:

Phase  1 : Single, Bulk payouts & I2P \- TDS Payment & Filing

* Version 1 \- Single & Bulk flow \- TDS Payment & Filing  
  * Version 1.1 \- Single Payout flow \- TDS Payment  
  * Version 1.2 \- Single & Bulk flow \- TDS Filing  
* Version 2 \- Bulk Payout flow \- \- TDS Payment & Filing

Phase 2 : APIs & Payout Links  \- TDS Payment & Filing

Phase 3 : Form 16A for all the orgs

## 

## 5.2. How will we drive adoption?

\<WIP\>

## 5.3. How will we extend this launch (in the future) 

Once we have added support for TDS payment and filing in all the payouts flow, we will then plan to add support for Form 16A also so that after TDS filing is done, merchants can automatically send Form 16A with all their vendors.  
Apart from that we are also planning to have one composite API that can handle both payout and TDS in the same API rather than having two different APIs.

## 5.4. Non-Goals/ Out of Scope

Composite APIs for Payouts and TDS

# 6\. How will we measure impact?

## 6.1. What will success look like for this project?

* 50-80 organizations enable TDS Payment setting within first 3 months  
* Out of the orgs who have opted for TDS payment, around 40-50% of the orgs should enable TDS filing through us .  
* More than 90% of these orgs payment & filing should be successful without any error in the first attempt.   
* NPS score should be above 40\.

Medium to Long term goal : 

* 20-25% lower churn rate among TDS-enabled customers vs. non-enabled customers \- This is because we are handling the complex filing process for them now.  
* 20-30% more engagement among TDS-enabled customers vs. non-enabled customers \- This is because earlier people were doing transactions outside Razorpay but now because TDS is handled by razorpay they are bound to do all other transactions through razorpay. 

## 6.2. Which OKR will this influence if outcomes are achieved?

* Increase NR from X to Y  
* Reduce churn by 10%  
* Increase acquisition by 15%  
* Increase engagement by 15%  
  * From engagement I meant the number of Payouts org made through us will increase.

## 6.3. How would we know if the solution is working?

These indicators will help us understand if the solution is delivering the intended value and progressing toward our plan or not :

#### **1\. Month-over-Month Growth in Organizations Using TDS Services**

**What we'll track:**

* Number of organizations with TDS Payment enabled (total and new adds per month)  
* Number of organizations with TDS Filing enabled (total and new adds per month)  
* Conversion rate: No Selection → TDS Payment → TDS Filing

**Success Signals:**

* **Month 1:** 15-20 new organizations enable TDS Payment  
* **Month 2-3:** 20%+ MoM growth in Filing enablement  
* **Month 4+:** Sustained 15-20% MoM growth as word-of-mouth and sales efforts compound

#### **2\. Successful First-Time Experience Rate**

**What we'll track:**

1. % of organizations that successfully complete first TDS payment without errors  
2. % of organizations that successfully complete first quarterly filing without errors

**Success Signals:**

* **Payment:** \>99% success rate on first payment attempt  
* **Filing:** \>99% success rate on first filing

#### **3\. Month-over-Month Net Revenue Growth from TDS Services**

**What we'll track:**

1. Total NR from TDS Payment processing fees  
2. Total NR from TDS Filing services  
3. NR per enabled organization  
4. MoM growth rate in TDS-attributed revenue

**Success Signals:**

* First Quarter launch \- 50K-1L NR from Filing  
* Next Quarter 2-3x growth in Filing NR 

#### **4\. Payout Volume Increase for Filing-Enabled Customers**

**What we'll track:**

1. Average monthly payout count per organization (before vs. after enabling Filing)  
2. % of Filing-enabled orgs showing 10%+ payout increase  
3. Time-to-impact: Months between Filing enablement and observable payout increase

**Success Signals:**

* 10% increase in average monthly payouts for filing-enabled orgs  
* Increase maintained or grows in subsequent quarters

#### **5\. Customer Retention and Churn Impact**

**What we'll track:**

1. Churn rate comparison: TDS-enabled vs. non-enabled customers  
2. Retention rate at 30 days, 90 days, 180 days, 365 days post-enablement

**Success Signals:**

* 15% lower churn rate for TDS-enabled customers

## 6.5. Any NFRs alongside the functional scope?

Document any non-functional requirements around scale, latency, availability, cost, security, and compliance that would be essential to achieve the outcome.

# 9\. Appendix (Optional)

## 9.1. Other Solutions Evaluated

This section should describe other solutions/approaches considered to solve this problem.

### 9.1.1. Alternate Solution 1

First alternate solution evaluated

### 9.1.2. Alternate Solution 2

Second alternate solution evaluated

## 10\. Are we making any assumptions?

This section should highlight any assumptions made in problem design or hypothesis building.

Open Points :   
Split of different segments among merchants (currently onboarded \+ potential) \- Added under section 2.2

TDS Category list \+ deciding its source of truth \- Compliance will have to maintain the source. For now we can use the same list which is used in I2P flow. Compliance Product will keep a track of all the changes and make sure the list is updated in case of new change.

At the time of payment challan \+ Filing challan (what's the difference) \- Payment challan is a receipt given by government telling that you have done the payment of X amount on this date. Filing challan is more detailed where government tells about all the amounts paid, how many deductees were there etc.

Payout count wise distribution of Dashboard \+ API Payouts (secondary group by merchants to identify the TDS use cases merchant's distribution) \- I have added this data under the overlap analysis sheet(Version wise part) but not able to do detail analysis on this

Possibility of

* Filing Credentials Validation while entering creds vs knowing at the time of filing (which can be discovered even 5 months later) \- Validation will be done when customers enter their credentials not instantly but after some time(\<1 day)  
* Automated TDS payment at the time of payout, instead of moving to escrow \- No this is not possible.. All the payments happen through our escrow and that is why we need to move money to our account before making the payment. This can be done once we have APIs from the government/Bank.

Do we want to show all TDS payments (BB+ & Payroll transactions) under the same dashboard?  
Why or Why not? 

\================================================================================  
    OPEN QUESTIONS  
\================================================================================

\--------------------------------------------------------------------------------  
1\. TDS CATEGORY SELECTION & LOWER DEDUCTION CERTIFICATE (LDC) HANDLING  
\--------------------------------------------------------------------------------

1a. Default TDS Category:  
\-------------------------  
Will there be a default TDS category pre-populated based on the vendor/contact   
profile? If so, where will this default be configured \- at the contact level   
during KYC or elsewhere?

1b. Lower Deduction Certificate (LDC) Support:  
\----------------------------------------------  
How will the system handle Lower Deduction Certificate (LDC) use cases?   
Specifically:

\- When a merchant selects the LDC option, will they be required to upload the   
  certificate at the time of each payout?

\- If certificate upload is required per transaction, this could become   
  repetitive and cumbersome. Should we instead capture and store LDC documents   
  at a centralized location (e.g., during contact KYC)?

\- What is the proposed UX flow for LDC scenarios?

\--------------------------------------------------------------------------------  
2\. COMMUNICATION WITH COMPLIANCE SERVICE  
\--------------------------------------------------------------------------------

2a. Payout Status Notification Trigger:  
\---------------------------------------  
At what stage of the payout lifecycle should BB+ inform the Compliance Service?

\- Should we notify Compliance Service only upon successful payout completion?

\- Or should we also notify on payout failure?

\- If we notify on failure, what action is expected from Compliance Service,   
  given that the merchant may retry or we may auto-retry for internal errors?

2b. Error Handling Strategy:  
\----------------------------  
What should be the error handling behavior for internal vs. external errors?

\- For internal errors (system/infrastructure failures): Should BB+ automatically   
  retry before notifying Compliance Service?

\- For external errors (e.g., bank rejection, insufficient funds): Should we   
  prompt the merchant to initiate a new payout, and only notify Compliance   
  Service upon eventual success?

\--------------------------------------------------------------------------------  
3\. TDS DEDUCTION TIMING  
\--------------------------------------------------------------------------------

When should the TDS amount be deducted and transferred to the Compliance account?

\- Option A (Instant): Deduct TDS immediately at the time of payout initiation

\- Option B (Scheduled): Deduct TDS at a scheduled time (e.g., end of day, or on   
  a specific date like the 1st/4th of the month as mentioned for I2P flow)

What is the recommended approach for each payout flow (Single, Bulk, Payout   
Links, I2P)? Are there any scenarios where one approach is preferred over   
the other?

\--------------------------------------------------------------------------------  
4\. EXTERNAL DATA IMPORT FOR TDS FILING \- SERVICE OWNERSHIP  
\--------------------------------------------------------------------------------

Which service (BB+ or Compliance Service) should own the responsibility of   
importing external TDS data for filing?

Context:   
When a merchant enables TDS Filing mid-month or mid-quarter, they may have TDS   
transactions processed outside Razorpay that need to be included in the   
quarterly filing. The PRD mentions an import functionality for new organizations.

Clarification Needed:

\- Should the import functionality reside with the Compliance Service (since they   
  house all TDS data and handle filing)?

\- Or should BB+ expose this import feature and pass the data to Compliance Service?

\- What validations should be performed on imported external data?

\--------------------------------------------------------------------------------  
5\. DATA REQUIREMENTS FOR TDS PAYMENT & FILING  
\--------------------------------------------------------------------------------

Can we get a comprehensive list of all data fields required for TDS payment and   
filing to ensure data sufficiency?

Specifically:

\- What transaction-level information must BB+ capture and store?

\- What deductor (merchant) details are required beyond what's already captured?

\- What deductee (vendor) details are mandatory for successful filing?

\- What are all possible 'data insufficiency' error scenarios that can occur   
  during filing?

This will help us ensure that BB+ collects all required information upfront   
during payout creation, minimizing filing errors and rework.

\--------------------------------------------------------------------------------  
6\. DATA DISCREPANCY RESOLUTION, REVISED FILING & DASHBOARD OWNERSHIP  
\--------------------------------------------------------------------------------

In case of data discrepancy or insufficiency discovered at the time of TDS   
filing, which service should own the resolution process?

Clarification Needed:

\- Notification Responsibility: Should BB+ or Compliance Service notify the   
  merchant about data insufficiency?

\- Data Correction Storage: Once the merchant updates/corrects the data, where   
  should the updated data be stored \- in BB+ or Compliance Service?

\- Revised Filing: If a revised filing is required due to data corrections,   
  which service handles this?

\- Data Sync: How do we ensure data consistency between BB+ and Compliance   
  Service after corrections?

Dashboard View Ownership:

\- Which service will power the Payout TDS Deductions dashboard view?

\- Recommendation: Compliance Service should own this view since it serves as   
  the source of truth for all TDS payment records and has complete details   
  against each payout payment. BB+ only has a limited view of payout-related   
  TDS deductions.

\- Need a final call on dashboard ownership for payout TDS deduction views.

\================================================================================  
                              END OF QUESTIONS  
\================================================================================

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>