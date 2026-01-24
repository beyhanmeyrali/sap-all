# Appendix A: Quick Reference Tables

> *Cheat Sheets for Everyday Use*

---

## A.1 BTP Hierarchy Cheat Sheet

| Level | What It Is | Analogy |
|-------|------------|---------|
| **Global Account** | Your contract with SAP | Lease agreement |
| **Directory** | Optional grouping | Floor in building |
| **Subaccount** | Where you work | Your apartment |
| **Environment** | Runtime (CF, Kyma, ABAP) | Type of appliance |
| **Service Instance** | Enabled service | Installed appliance |

---

## A.2 Destination Configuration Options

| Property | Values | When to Use |
|----------|--------|-------------|
| **Type** | HTTP, RFC, LDAP, MAIL | HTTP for most APIs |
| **Proxy Type** | Internet, OnPremise, PrivateLink | OnPremise needs Cloud Connector |
| **Authentication** | NoAuth, Basic, OAuth2*, ClientCert, SAML | OAuth2 for production |

### Common Additional Properties

| Property | Value | Purpose |
|----------|-------|---------|
| `sap-client` | 100, 200, etc. | SAP system client |
| `HTML5.DynamicDestination` | true | Fiori apps |
| `WebIDEEnabled` | true | BAS development |
| `TrustAll` | true | Skip cert validation (dev only!) |

---

## A.3 Joule Skill vs. Agent Comparison

| Aspect | Skill | Agent |
|--------|-------|-------|
| **Purpose** | Single action | Orchestrate multiple |
| **Complexity** | Simple | Can reason & chain |
| **Contains** | One action | Multiple skills |
| **Example** | "Get order status" | "Handle complaint" |
| **Build order** | First | After skills exist |

---

## A.4 RISE vs. Classic Comparison

| Aspect | Classic On-Prem | RISE with SAP |
|--------|-----------------|---------------|
| **Hosting** | Your data center | SAP managed (AWS/Azure/GCP) |
| **Basis work** | Your team | SAP handles |
| **Upgrades** | You control | SAP delivers |
| **Custom ABAP** | Full freedom | Clean Core enforced |
| **Extensions** | Inside S/4 | BTP (side-by-side) |
| **Cost model** | CapEx + licenses | Subscription (OpEx) |
| **BTP included** | Separate purchase | Credits included |

---

## A.5 Authentication Type Quick Reference

| Type | Use Case | Credentials Needed |
|------|----------|-------------------|
| **NoAuthentication** | Public APIs | None |
| **BasicAuthentication** | Simple/legacy APIs | User + Password |
| **OAuth2ClientCredentials** | Server-to-server | Client ID + Secret + Token URL |
| **OAuth2SAMLBearerAssertion** | SAP-to-SAP with user context | Client ID + Secret + Token URL |
| **ClientCertificateAuthentication** | High security | Certificate + Key |
| **PrincipalPropagation** | User context forwarding | Trust setup |

---

*[Back to Table of Contents](../content.md)*
