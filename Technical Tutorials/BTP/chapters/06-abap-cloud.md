# Chapter 6: ABAP in the Cloud

> *Your Skills Transfer—With Some New Rules*

---

## 6.1 BTP ABAP Environment Overview

The **BTP ABAP Environment** (sometimes called "Steampunk") is a cloud-based ABAP runtime where you can write modern ABAP code.

**Key Point**: It's NOT a copy of your on-prem system. It's a fresh, clean ABAP environment designed for cloud-native development.

---

## 6.2 How It Differs from Classic On-Prem ABAP

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| Development tool | SE80 / Eclipse | Eclipse ADT only |
| Available objects | Everything | Allowlisted objects only |
| Database access | Open SQL + Native | Open SQL only |
| Modifications | Allowed | Not allowed |
| Transports | CTS/CTS+ | Git-based (gCTS) |
| UI development | Dynpro, ALV, WebDynpro | RAP + Fiori only |

---

## 6.3 RAP (RESTful ABAP Programming) Basics

**RAP** is the new programming model for ABAP. It focuses on:
- **CDS Views** for data modeling
- **Behavior Definitions** for business logic
- **OData services** for exposure

```abap
" CDS View
@AccessControl.authorizationCheck: #CHECK
define root view entity ZI_SalesOrder as select from zsalesorder {
  key order_id,
  customer,
  amount,
  status
}

" Behavior Definition
managed implementation in class zbp_salesorder unique;
define behavior for ZI_SalesOrder {
  create; update; delete;
}
```

---

## 6.4 When to Use BTP ABAP vs. Classic ABAP

| Scenario | Use BTP ABAP | Use Classic ABAP |
|----------|--------------|------------------|
| New extension for RISE S/4 | ✅ | ❌ |
| Greenfield cloud project | ✅ | ❌ |
| Heavy modification of existing | ❌ | ✅ (if on-prem) |
| Legacy reports/ALV | ❌ | ✅ |
| New OData services | ✅ | Possible |

---

## 6.5 Migration Path: Moving Code from On-Prem

1. **Analyze** with Custom Code Migration tools
2. **Identify** unreleased APIs (must replace)
3. **Refactor** to use released APIs
4. **Rewrite** in RAP model
5. **Test** in BTP ABAP Environment

---

*[Previous: Chapter 5](05-destinations.md) | [Next: Chapter 7](07-fiori-ui5-btp.md)*

*[Back to Table of Contents](../content.md)*
