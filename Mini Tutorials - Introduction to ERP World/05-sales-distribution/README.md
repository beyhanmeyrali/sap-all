# 💰 SD: Sales & Distribution - Getting Products to Customers

*Become Diego: The Deal Closer*

## Diego Martinez - "The Sales Virtuoso": From First Call to Billion-Dollar Relationships

Meet **Diego Martinez**, the Sales & Distribution mastermind who transforms customer inquiries into delivered excellence and lifetime partnerships. His journey from nervous phone calls to closing multi-million dollar deals will teach you that SD isn't just about selling products - it's about solving problems and creating value that customers can't live without.

### 🎬 Chapter 1: The Phone Call That Started Everything (Day One)

Diego nervously adjusts his headset. It's his first day at GlobalTech, and the phone is ringing. On the other end: **Amazon's procurement team** asking about a massive laptop order.

**The Call:**
- Customer: Amazon Business Solutions  
- Need: 1000 premium laptops
- Timeline: 2 weeks (impossible?)
- Budget: $1.2 million
- Stakes: Potential long-term partnership worth millions

**Diego's Challenge:** Turn this inquiry into the deal that changes everything.

### 🎯 Chapter 2: Customer Master Data - Building Relationships (Week 1)

Diego learns that every customer relationship starts with perfect data:

#### **Customer Master Creation (XD01/VD01):**

```
👥 Diego's Customer Masterpiece - Amazon Profile:
┌─ General Data ────────────────────────────────────┐
│ Customer: AMAZON-US-001                           │
│ Name: Amazon Business Solutions Inc.              │
│ Search Term: AMAZON                               │
│ Industry: E-Commerce/Technology                   │
│ Language: EN (English)                            │
│ Country: US (United States)                       │
└────────────────────────────────────────────────────┘

┌─ Company Code Data (FI Integration) ──────────────┐
│ Payment Terms: Z030 (Net 30 Days)                │
│ Credit Limit: $5,000,000                         │
│ Currency: USD                                     │
│ Reconciliation Account: 140000 (AR)              │
│ Risk Category: A (Lowest Risk)                   │
└────────────────────────────────────────────────────┘

┌─ Sales Area Data (SD Specific) ───────────────────┐
│ Sales Organization: 1000                          │
│ Distribution Channel: 10 (Direct Sales)          │
│ Division: 01 (Technology Products)               │
│ Customer Group: Z1 (Enterprise)                  │
│ Price Group: 01 (Standard)                       │
│ Delivery Priority: 02 (High Priority)            │
│ Shipping Conditions: 01 (Standard Shipping)      │
│ Incoterms: FOB (Free on Board)                   │
└────────────────────────────────────────────────────┘
```

**Diego's First Lesson:** *"A customer master isn't just data - it's the foundation of every future interaction!"*

### 🛒 Chapter 3: The Sales Order Journey - Where Dreams Become Commitments (Week 2)

Diego discovers the heart of SD: the Sales Order (VA01).

#### **The Amazon Sales Order Creation:**

```
📋 Diego's Million-Dollar Sales Order:
┌─ Header Data ─────────────────────────────────────┐
│ Sales Order: 2000012345                           │
│ Order Type: OR (Standard Order)                   │
│ Sales Organization: 1000                          │
│ Distribution Channel: 10                          │
│ Division: 01                                      │
│ Sold-to Party: AMAZON-US-001                      │
│ Ship-to Party: AMAZON-WH-SEA                      │
│ Bill-to Party: AMAZON-US-001                      │
│ Payer: AMAZON-US-001                              │
│ Order Date: Today                                 │
│ Requested Delivery: +14 Days                     │
│ Purchase Order: AMZ-PO-2024-7891                 │
└────────────────────────────────────────────────────┘

┌─ Line Items ──────────────────────────────────────┐
│ Item 10:                                          │
│   Material: LAPTOP-PREMIUM-15                    │
│   Description: Premium Laptop 15"                │
│   Quantity: 1000 EA                              │
│   Unit Price: $1,200.00                          │
│   Net Value: $1,200,000                          │
│   Plant: 1000                                    │
│   Shipping Point: 1000                           │
│   Route: R001 (US West Coast)                    │
│   Delivery Date: +14 Days                        │
└────────────────────────────────────────────────────┘

Order Status: ✅ Created Successfully!
Next Step: ATP Check (Available to Promise)
```

### 💡 Chapter 4: ATP Magic - Making Promises You Can Keep (Week 2)

Diego learns the most crucial SD skill: **Available to Promise (ATP)** checking.

#### **The ATP Reality Check (CO09/MD04):**

```
🔮 Diego's ATP Analysis:
┌─ Material Availability Check ─────────────────────┐
│ Material: LAPTOP-PREMIUM-15                       │
│ Required Quantity: 1000 EA                        │
│ Required Date: +14 Days                           │
├─ Current Situation ───────────────────────────────┤
│ Stock on Hand: 250 EA                             │
│ Planned Receipts: 500 EA (+7 days)               │
│ Production Orders: 750 EA (+10 days)             │
│ Total Available: 1500 EA by Day +10              │
├─ ATP Result ──────────────────────────────────────┤
│ Confirmed Quantity: 1000 EA ✅                    │
│ Confirmed Date: Day +12 (2 days early!)          │
│ Status: PROMISE CONFIRMED 🎉                      │
└────────────────────────────────────────────────────┘

Diego's Magic Moment: "We can deliver 1000 units in 12 days!" 
Amazon's Response: "Incredible! Let's talk partnership."
```

### 🚚 Chapter 5: Delivery Excellence - From Promise to Reality (Week 3-4)

Diego masters the delivery process that turns sales orders into customer satisfaction:

#### **Delivery Document Creation (VL01N):**

```
📦 Diego's Delivery Orchestration:
┌─ Outbound Delivery ───────────────────────────────┐
│ Delivery: 8000012345                              │
│ Delivery Type: LF (Standard Delivery)             │
│ Sales Order: 2000012345                           │
│ Ship-to Party: AMAZON-WH-SEA                      │
│ Shipping Point: 1000                              │
│ Route: R001                                       │
│ Planned Goods Issue: Day +12                      │
│ Transportation: UPS Freight                       │
│ Total Weight: 2500 KG                             │
│ Total Volume: 12.5 M3                             │
└────────────────────────────────────────────────────┘

┌─ Picking Process (VL02N) ─────────────────────────┐
│ Warehouse Task: Pick 1000 LAPTOP-PREMIUM-15      │
│ Storage Location: 1001                            │
│ Batch: LOT-2024-001                              │
│ Serial Number Range: SN001001-SN001000           │
│ Picker: Team Alpha                                │
│ Status: ✅ Picking Complete                       │
│ Quality Check: ✅ Passed                          │
│ Packing: ✅ Complete                              │
└────────────────────────────────────────────────────┘

┌─ Goods Issue (VL02N + MIGO) ──────────────────────┐
│ Goods Issue Date: Day +11 (1 day early!)         │
│ Movement Type: 601 (GI for Delivery)             │
│ Stock Reduction: 1000 EA                          │
│ Cost of Goods Sold: $800,000                     │
│ Status: ✅ Shipped to Customer                     │
│ Tracking: UPS-TRK-789456123                      │
└────────────────────────────────────────────────────┘
```

### 💰 Chapter 6: Billing Brilliance - Converting Value to Revenue (Week 4)

Diego learns the final step: converting delivered value into revenue.

#### **Billing Document Creation (VF01):**

```
🧾 Diego's Billing Masterpiece:
┌─ Invoice Header ──────────────────────────────────┐
│ Billing Document: 9000012345                      │
│ Billing Type: F2 (Standard Invoice)               │
│ Billing Date: Day +12                             │
│ Payer: AMAZON-US-001                              │
│ Payment Terms: Net 30 Days                        │
│ Invoice Total: $1,200,000.00                      │
│ Tax Amount: $96,000.00 (8% Sales Tax)            │
│ Grand Total: $1,296,000.00                        │
└────────────────────────────────────────────────────┘

┌─ Revenue Recognition (Auto FI Posting) ───────────┐
│ Dr. Accounts Receivable: $1,296,000               │
│ Cr. Revenue Account: $1,200,000                   │
│ Cr. Tax Payable: $96,000                          │
│ Status: ✅ Posted to Financial Accounting          │
│ Payment Due: Day +42                              │
└────────────────────────────────────────────────────┘
```

### 🎨 Chapter 7: Pricing Mastery - The Art of Profitable Selling (Month 2)

Diego becomes a pricing virtuoso, learning condition techniques:

#### **Pricing Procedure Configuration (V/08):**

```
💰 Diego's Pricing Strategy:
┌─ Condition Types & Sequence ──────────────────────┐
│ Step│ Condition │ Description        │ Value     │
│  10 │ PR00      │ Base Price         │$1,200.00  │
│  20 │ K007      │ Volume Discount    │   -5.0%   │
│  30 │ K029      │ Partnership Bonus  │   -2.0%   │
│  40 │ MWST      │ Sales Tax          │   +8.0%   │
│  50 │ ZF01      │ Freight Cost       │  +$50.00  │
└────────────────────────────────────────────────────┘

Calculation:
Base Price (1000 × $1,200):     $1,200,000
Volume Discount (-5%):           -$60,000
Partnership Bonus (-2%):         -$22,800
Subtotal:                        $1,117,200
Sales Tax (8%):                  +$89,376
Freight:                         +$50,000
Final Price:                     $1,256,576

Amazon's Response: "This pricing is competitive! ✅"
```

### 🎯 Chapter 8: Advanced SD Features - Master Level Skills (Month 3)

Diego evolves into an SD guru, mastering complex scenarios:

#### **Credit Management (VKM1/VKM2):**

```
🛡️ Diego's Credit Risk Management:
┌─ Credit Control Area ─────────────────────────────┐
│ Customer: AMAZON-US-001                           │
│ Credit Limit: $5,000,000                         │
│ Current Exposure: $2,300,000                     │
│ Open Orders: $1,296,000                          │
│ Available Credit: $1,404,000                     │
│ Risk Category: GREEN ✅                           │
│ Payment History: 98.5% on-time                   │
│ Credit Status: APPROVED                           │
└────────────────────────────────────────────────────┘

Automatic Credit Check Results:
✅ Order released automatically
✅ Delivery released automatically  
✅ Billing released automatically
```

#### **Returns Management (VA01 with REN order type):**

```
🔄 Diego's Returns Excellence:
Scenario: Customer reports 5 defective units

┌─ Return Order (VA01) ─────────────────────────────┐
│ Return Order: 2000012346                          │
│ Order Type: REN (Standard Return)                 │
│ Reference: 2000012345 (Original Order)           │
│ Return Reason: 10 (Defective Material)           │
│ Quantity: 5 EA                                   │
│ Credit Value: $6,000.00                          │
│ Replacement: Auto-trigger new delivery           │
└────────────────────────────────────────────────────┘

Result: Customer satisfied, relationship strengthened!
```

### 🚀 Chapter 9: The Partnership Victory - Diego's Masterpiece

**Month 6 - The Follow-up Call:**

Amazon calls Diego: *"That laptop delivery was flawless. Quality perfect, timing excellent, service outstanding. We want to discuss a $50 million annual contract."*

**Diego's Response:** *"Let's build something amazing together."*

#### **The Multi-Year Contract Setup:**

```
🏆 Diego's Partnership Framework:
┌─ Contract Management (VK11/VK12) ─────────────────┐
│ Contract: 5000000001                              │
│ Customer: AMAZON-US-001                           │
│ Contract Type: WK1 (Value Contract)               │
│ Validity: 3 Years                                 │
│ Target Value: $150,000,000                        │
│ Volume Commitments: 50,000 units/year            │
│ Special Pricing: Tier-based discounts            │
│ Payment Terms: Net 45 Days                       │
│ SLA Commitments: 99.5% on-time delivery          │
└────────────────────────────────────────────────────┘

Business Impact:
💰 $150M revenue over 3 years
🤝 Strategic partnership established
📈 40% increase in annual revenue
🌟 Reference customer for other prospects
```

### 🎯 SD Process Flow Chart

```
Diego's Sales & Distribution Complete Flow:
┌─────────────────────────────────────────────────────┐
│               Customer Inquiry                      │
│              (Phone/Email/Web)                      │
└─────────────────────┬───────────────────────────────┘
                      │
        ┌─────────────▼──────────────┐
        │  Quotation (VA01 - QT)     │
        │  "Here's what we can do"   │
        └─────────────┬──────────────┘
                      │
          ┌───────────▼─────────────┐
          │  Sales Order (VA01-OR) │
          │    "Customer Says Yes" │
          └───────────┬─────────────┘
                      │
      ┌───────────────▼────────────────┐
      │     ATP Check (CO09/MD04)      │
      │  "Can we deliver as promised?" │
      └───────────────┬────────────────┘
                      │
        ┌─────────────▼──────────────┐
        │ Delivery Processing        │
        │ (VL01N→VL02N→VL02N+MIGO)   │
        └─────────────┬──────────────┘
                      │
          ┌───────────▼─────────────┐
          │   Billing (VF01)       │
          │ "Convert to Revenue"    │
          └───────────┬─────────────┘
                      │
            ┌─────────▼──────────┐
            │ Payment Processing │
            │ (F-32 Incoming)    │
            └────────────────────┘
```

### 📚 Essential SD T-Codes Reference

```
🔑 Diego's SD T-Code Mastery:
┌─ Customer Management ─────────────────────────────┐
│ XD01 - Create Customer (General)                 │
│ VD01 - Create Customer (Sales)                   │
│ XD02/VD02 - Change Customer                      │
│ XD03/VD03 - Display Customer                     │
└────────────────────────────────────────────────────┘

┌─ Sales Order Processing ──────────────────────────┐
│ VA01 - Create Sales Order                        │
│ VA02 - Change Sales Order                        │
│ VA03 - Display Sales Order                       │
│ VA05 - List Sales Orders                         │
│ VA21 - Create Quotation                          │
│ VA22 - Change Quotation                          │
│ VA23 - Display Quotation                         │
└────────────────────────────────────────────────────┘

┌─ Delivery Processing ─────────────────────────────┐
│ VL01N - Create Outbound Delivery                 │
│ VL02N - Change Outbound Delivery                 │
│ VL03N - Display Outbound Delivery                │
│ VL06O - List Outbound Deliveries                 │
│ VT01N - Create Shipment                          │
│ VT02N - Change Shipment                          │
│ VT03N - Display Shipment                         │
└────────────────────────────────────────────────────┘

┌─ Billing & Invoicing ─────────────────────────────┐
│ VF01 - Create Billing Document                   │
│ VF02 - Change Billing Document                   │
│ VF03 - Display Billing Document                  │
│ VF04 - Maintain Billing Due List                 │
│ VF11 - Cancel Billing Document                   │
└────────────────────────────────────────────────────┘

┌─ Pricing & Conditions ────────────────────────────┐
│ VK11 - Create Condition Record                   │
│ VK12 - Change Condition Record                   │
│ VK13 - Display Condition Record                  │
│ VK32 - Create Customer Material Info             │
│ V/08 - Pricing Procedure Configuration           │
└────────────────────────────────────────────────────┘

┌─ Credit Management ───────────────────────────────┐
│ VKM1 - Create Credit Master                      │
│ VKM2 - Change Credit Master                      │
│ VKM3 - Display Credit Master                     │
│ VKM4 - Credit Management Overview                │
│ FSCM - Credit Management Workbench               │
└────────────────────────────────────────────────────┘

┌─ Returns & Complaints ────────────────────────────┐
│ VA01 - Create Return Order (REN type)            │
│ VF01 - Create Credit Memo                        │
│ VA41 - Create Contract                           │
│ VA42 - Change Contract                           │
│ VA43 - Display Contract                          │
└────────────────────────────────────────────────────┘
```

### 💡 Diego's Final Wisdom: The Sales Master's Creed

*"Sales & Distribution isn't about pushing products - it's about pulling customers toward their success. Every order you process, every delivery you orchestrate, every relationship you nurture creates value that extends far beyond the transaction. SD is where customer dreams meet business reality, and both come out winning."*

---

*"Sales is not about convincing people to buy what you have. It's about discovering what they need and becoming the bridge that gets them there."*

**Created by [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali/) - SAP Storyteller & Digital Transformation Advocate**

---

## Navigation
← [Previous: Production Planning](../04-production-planning/README.md) | [Back to Main Journey](../../README.md) | [Next: Financial Accounting](../06-financial-accounting/README.md) →

## Related Stories
- 🌟 [Complete Integration Example](../01-overview/Complete-SAP-Integration-Example.md)
- 🎼 [The SAP Orchestra](../01-overview/README-Orchestra.md)