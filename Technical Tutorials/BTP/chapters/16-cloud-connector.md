# Chapter 16: Cloud Connector for On-Premise Access

> *The Secure Tunnel to Your Data Center*

---

## 16.1 What Is Cloud Connector?

A lightweight agent installed on-premise that creates a secure tunnel between your internal network and BTP.

```
BTP Cloud ◄──── Secure Tunnel ────► Cloud Connector ◄──► On-Prem Systems
```

It's outbound-only (initiated from your side), so minimal firewall changes needed.

---

## 16.2 Setting Up for Legacy System Access

### Installation
1. Download from SAP Support Portal
2. Install on a Windows/Linux server in your DMZ
3. Configure connection to your BTP subaccount

### Virtual Host Mapping
```
Virtual Host (what BTP sees):    s4-virtual:443
Internal Host (actual):          10.0.0.50:8000
```

BTP destinations use the virtual host name.

### Path Exposure
Only expose paths you need:
```
/sap/opu/odata/sap/*     → OData services
/sap/bc/adt/*            → ABAP tools
```

---

## 16.3 Common Scenarios

| Scenario | Setup |
|----------|-------|
| Joule skill calling on-prem S/4 | Destination with ProxyType=OnPremise |
| Fiori app consuming on-prem OData | Cloud Connector + destination |
| BTP ABAP calling legacy RFC | RFC destination via CC |

---

*[Previous: Chapter 15](15-integration-suite.md) | [Next: Chapter 17](17-clean-core.md)*

*[Back to Table of Contents](../content.md)*
