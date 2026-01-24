# Chapter 5: Destinations – The Phone Book for Connections

> *The Most Important BTP Concept for ABAP Developers*

---

## 5.1 What Is a Destination?

A **Destination** in BTP is a **named, centrally managed connection configuration**.

> **Analogy**: It's like a **contact card in your phone**.
> - You don't type Mom's number every time
> - You just say "Call Mom"
> - Behind the scenes: +90 212 555 1234

In BTP: reference `MY_S4_SYSTEM` instead of hardcoding URLs and passwords.

---

## 5.2 Why Destinations Exist (Security & Governance)

- ❌ Passwords in code = security nightmare
- ❌ URLs in code = can't change without redeploying
- ✅ Central credential management
- ✅ Rotate passwords without touching apps
- ✅ Audit trail of who uses what

---

## 5.3 Creating Your First Destination

In BTP Cockpit → Connectivity → Destinations:

```
Name:           MY_S4_ORDERS
Type:           HTTP
URL:            https://s4.acme.com:443/sap/opu/...
Proxy Type:     Internet
Authentication: BasicAuthentication
User:           API_USER
Password:       ********
```

---

## 5.4 Authentication Types

| Type | When to Use |
|------|-------------|
| **BasicAuthentication** | Simple, not for production |
| **OAuth2ClientCredentials** | Modern, server-to-server |
| **OAuth2SAMLBearerAssertion** | SAP-to-SAP with user context |
| **ClientCertificateAuthentication** | High security |
| **NoAuthentication** | Public APIs only |

---

## 5.5 On-Premise Access via Cloud Connector

For systems behind your firewall:

```
BTP Cloud ◄──── Secure Tunnel ────► Cloud Connector ◄──► On-Prem S/4
```

Use `ProxyType: OnPremise` in destination configuration.

---

## 5.6 Naming Conventions That Save Your Sanity

```
{CLIENT}_{SYSTEM}_{ENV}_{PURPOSE}

Examples:
ACME_S4_PROD_ORDERS
BIMSA_SF_DEV_EMPLOYEES
```

---

*[Previous: Chapter 4](04-rise-and-btp.md) | [Next: Chapter 6](06-abap-cloud.md)*

*[Back to Table of Contents](../content.md)*
