# Chapter 4: How RISE and BTP Fit Together

> *The Neighborhood View*

---

## 4.1 The Neighborhood View: RISE House + BTP Extension Wing

Here's the picture that makes it click:

```
┌───────────────────────────────────────────────────────────────┐
│                    THE CLOUD NEIGHBORHOOD                      │
│                                                                 │
│  ┌────────────────────────┐    ┌────────────────────────────┐  │
│  │   RISE HOUSE           │    │   BTP EXTENSION WING       │  │
│  │   (S/4HANA Cloud)      │◄──►│   (Your Custom Work)       │  │
│  │                        │    │                            │  │
│  │  • Standard ERP        │    │  • Custom Fiori Apps       │  │
│  │  • Master Data         │    │  • RAP ABAP Services       │  │
│  │  • Core Transactions   │    │  • Joule Agents/Skills     │  │
│  │  • SAP-Managed         │    │  • Integrations            │  │
│  │                        │    │  • Workflows               │  │
│  └────────────────────────┘    └────────────────────────────┘  │
│           ▲                              ▲                      │
│           │         DESTINATIONS         │                      │
│           └──────────(Bridge)────────────┘                      │
│                                                                 │
└───────────────────────────────────────────────────────────────┘
```

- **RISE** = The fully managed S/4HANA cloud house (the ERP core)
- **BTP** = The flexible extension wing/garage/workshop
- **Destinations** = The bridge connecting them

They're **designed to work together tightly**:
- A custom Fiori app in BTP pulls data from S/4 via OData
- A Joule skill reads sales orders from RISE S/4
- An integration flow in BTP connects RISE to external systems

---

## 4.2 BTP Credits in RISE Contracts

Here's something many don't realize:

> Every RISE contract includes **BTP credits** (like free fuel vouchers) for building extensions.

### What You Get

- A dedicated BTP subaccount (often called "BTP for RISE")
- Entitlements for common services (depends on contract)
- Credits to consume BTP services

### Typical Included Services

| Service | What For |
|---------|----------|
| SAP Build (Work Zone, Apps, PA) | Low-code development |
| Integration Suite | Connect systems |
| ABAP Environment | Cloud ABAP development |
| AI Core / Joule | AI capabilities |
| HANA Cloud | Additional databases |

### Watch Out

Credits aren't unlimited. High-consumption services (like AI and large HANA instances) can burn through credits fast. Monitor usage!

---

## 4.3 The New Reality for ABAP/Fiori Developers

Let's be concrete about what changes:

### Old World (On-Prem S/4)

```
┌─────────────────────────────────────┐
│         S/4HANA On-Prem             │
│                                     │
│  Your modifications live HERE:     │
│  • Z-programs in SE80              │
│  • Z-tables in SE11                │
│  • User exits and BADIs            │
│  • Fiori apps on embedded FLP      │
│  • Everything in one system         │
│                                     │
└─────────────────────────────────────┘
```

### RISE World

```
┌─────────────────────────────────────┐
│     S/4HANA Cloud (RISE)            │
│   "The Rented House"                │
│                                     │
│  ✗ Can't modify core                │
│  ✗ Can't add Z-tables               │
│  ✗ Can't use unreleased APIs        │
│  ✓ Can configure using Key User     │
│  ✓ Exposes APIs for external use    │
│                                     │
└──────────────────┬──────────────────┘
                   │
                   │ APIs
                   │ (OData, Events)
                   ▼
┌─────────────────────────────────────┐
│        BTP Subaccount               │
│   "Your Workshop Next Door"         │
│                                     │
│  ✓ Custom ABAP (RAP in ABAP Env)    │
│  ✓ Custom Fiori Apps (BAS)          │
│  ✓ Joule Agents with Skills         │
│  ✓ Integration Flows                │
│  ✓ Custom Tables (own HANA)         │
│                                     │
└─────────────────────────────────────┘
```

### What This Means Day-to-Day

| Task | Old Way | RISE + BTP Way |
|------|---------|----------------|
| Add custom field | Append structure in SE11 | Key User extensibility or BTP |
| Custom report | SE38 program | RAP in BTP ABAP Env → Fiori app |
| Custom table | SE11 in S/4 | HANA Cloud table in BTP |
| Complex logic | Function module in core | Service in BTP, call via API |
| Fiori extension | Extend in WebIDE, deploy to FLP | Extend in BAS, deploy to BTP |

---

## 4.4 Comparison Table: Classic On-Prem vs. RISE vs. BTP

| Aspect | Classic On-Prem | RISE (S/4 Cloud) | BTP (Extensions) |
|--------|-----------------|-------------------|-------------------|
| **Who manages infra?** | Your Basis team | SAP | SAP |
| **S/4HANA version** | You control | SAP delivers updates | N/A |
| **Custom ABAP** | Full freedom (risky) | Clean Core only | RAP in ABAP Env |
| **Fiori apps** | Deploy to on-prem FLP | Embedded FLP | Build/deploy in BAS |
| **Extensions location** | Inside S/4 | Outside (BTP) | This is BTP! |
| **Cost model** | Hardware + licenses | Subscription | Credits/consumption |
| **Typical for...** | Legacy SAP shops | Cloud transformation | All custom work |

---

## Practical Example: Customer Wants a Custom Dashboard

### Old Way (On-Prem)

1. Create Z-program to fetch data
2. Create Z-CDS views
3. Build Fiori app in WebIDE
4. Deploy to on-prem FLP
5. All in one system

### RISE + BTP Way

1. **In RISE S/4**: Release standard OData services (or use existing)
2. **In BTP**: Create destination pointing to S/4
3. **In BTP (BAS)**: Build Fiori app consuming the OData
4. **In BTP**: Deploy to SAP Build Work Zone
5. Users access via BTP launchpad

The dashboard **looks the same** to users, but the architecture is fundamentally different.

---

## Key Takeaways

1. **RISE is the managed ERP core** — S/4HANA with SAP handling infrastructure
2. **BTP is the extension platform** — where your custom work lives
3. **They connect via APIs** — Destinations bridge the two
4. **RISE includes BTP credits** — use them for extensions
5. **Your skills transfer** — ABAP is still ABAP, just in a new location

---

## What's Next?

You understand the big picture of RISE + BTP. Now let's get practical. In Part III, we'll dive into the core BTP concepts you'll use daily, starting with **Destinations**—the critical bridge between systems.

---

*[Previous: Chapter 3 – RISE with SAP](03-rise-with-sap.md) | [Next: Chapter 5 – Destinations](05-destinations.md)*

*[Back to Table of Contents](../content.md)*
