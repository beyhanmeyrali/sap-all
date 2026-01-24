# Chapter 12: Managing Multiple Clients on BTP

> *The Consultant's Survival Guide*

---

## 12.1 The Classic Trap: One Subaccount for Everything

Many start with one big subaccount and throw everything in:
- Client A's Joule agent
- Client B's destination
- Client C's test extension
- Your playground stuff

**After 2-3 clients**:
- Entitlements get messy
- Security nightmare
- Naming chaos
- Cost allocation impossible
- Handover problems

---

## 12.2 Best Practice: One Subaccount Per Client/Project

```
Global Account: Partner Global Account
├── Directory: "EMEA Clients"
│   ├── Subaccount: CLIENT_A_PROD
│   ├── Subaccount: CLIENT_A_DEV
│   └── Subaccount: CLIENT_B_PROD
├── Directory: "Internal Tools"
│   └── Subaccount: TEMPLATES
```

**Benefits**:
- Strong isolation
- Clear billing
- Easy handover
- Security boundaries

---

## 12.3 Naming Conventions for Multi-Client Environments

### Subaccounts
```
{REGION}_{CLIENT}_{ENV}
TR_ACME_PROD
TR_ACME_DEV
IST_BIMSA_POC
```

### Destinations
```
{CLIENT}_{SYSTEM}_{ENV}_{PURPOSE}
ACME_S4_PROD_ORDERS
BIMSA_SF_DEV_EMPLOYEES
```

---

## 12.4 Security & Access Management

- Use SAP IAS or client's Identity Provider
- Assign yourself Global Account Administrator
- Give client roles only in their subaccount
- Use technical users for integrations

---

## 12.5 Cost Allocation and Chargeback

Each subaccount shows its own usage:
- BTP Cockpit → Usage Analytics
- Track per-client consumption
- Enable internal billing

---

*[Previous: Chapter 11](11-agent-lifecycle.md) | [Next: Chapter 13](13-cross-customer-deployments.md)*

*[Back to Table of Contents](../content.md)*
