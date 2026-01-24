# Ek A: Hızlı Referans Tables

> *Cheat Sheets for Everyday Use*

---

## A.1 BTP Hierarchy Cheat Sheet

```mermaid
graph TD
    GA[Global Account] --> DIR1[Directory 1]
    GA --> DIR2[Directory 2]
    DIR1 --> SA1[Subaccount DEV]
    DIR1 --> SA2[Subaccount TEST]
    DIR1 --> SA3[Subaccount PROD]
    DIR2 --> SA4[Subaccount HR]

    SA1 --> ENV1[Cloud Foundry]
    SA1 --> ENV2[ABAP Environment]

    ENV1 --> SVC1[Service Instance]
    ENV1 --> SVC2[Destination]

    style GA fill:#9C27B0,color:white
    style SA3 fill:#4CAF50,color:white
    style SA4 fill:#4CAF50,color:white
```

| Level | Nedir | Analogy | Örnek |
|-------|------------|---------|---------|
| **Global Account** | Your contract with SAP | Lease agreement | `ACME_Corp_GA` |
| **Directory** | Optional grouping | Floor in building | `Production`, `Non-Prod` |
| **Subaccount** | Where you work | Your apartment | `ACME_PROD_EU10` |
| **Environment** | Runtime (CF, Kyma, ABAP) | Type of appliance | `Cloud Foundry` |
| **Service Instance** | Enabled service | Installed appliance | `destination-service` |

---

## A.2 Destination Configuration Options

```mermaid
flowchart TD
    D[Create Destination] --> TYPE{Type?}

    TYPE --> HTTP[HTTP]
    TYPE --> RFC[RFC]
    TYPE --> LDAP[LDAP]
    TYPE --> MAIL[MAIL]

    HTTP --> PROXY{Proxy Type?}
    PROXY --> INTERNET[Internet]
    PROXY --> ONPREM[OnPremise]
    PROXY --> PVTLINK[PrivateLink]

    ONPREM --> CC[Requires Cloud Connector]

    INTERNET --> AUTH{Authentication?}
    AUTH --> NOAUTH[NoAuth]
    AUTH --> BASIC[Basic]
    AUTH --> OAUTH[OAuth2*]
    AUTH --> CERT[ClientCert]

    style HTTP fill:#4CAF50,color:white
    style OAUTH fill:#2196F3,color:white
    style CC fill:#FF9800,color:white
```

| Property | Values | Ne Zaman Kullanılır |
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
| `WebIDEUsage` | odata_abap, odata_gen | Service type |
| `TrustAll` | true | Skip cert validation (dev only!) |
| `URL.headers.x-api-key` | your-api-key | API key header |

### Complete Destination Örneks

**S/4HANA Cloud (OAuth2):**
```yaml
Name: ACME_S4_PROD
Type: HTTP
URL: https://my300001.s4hana.ondemand.com
Proxy Type: Internet
Authentication: OAuth2ClientCredentials
Client ID: sb-xsuaa-acme!t12345
Client Secret: ********
Token Service URL: https://my300001.authentication.eu10.hana.ondemand.com/oauth/token
Additional Properties:
  sap-client: 100
```

**On-Premise via Cloud Connector:**
```yaml
Name: ACME_ECC_ONPREM
Type: HTTP
URL: http://ecc-virtual:443
Proxy Type: OnPremise
Authentication: BasicAuthentication
User: BTPUSER
Password: ********
Additional Properties:
  sap-client: 800
```

---

## A.3 Joule Skill vs. Agent Comparison

```mermaid
graph LR
    subgraph "Skills = Tools"
        S1[Skill: Get Order]
        S2[Skill: Check Stock]
        S3[Skill: Create Return]
    end

    subgraph "Agent = Brain"
        A[Agent] --> |"Decides"| S1
        A --> |"Chains"| S2
        A --> |"Orchestrates"| S3
    end

    USER[User Request] --> A

    style A fill:#2196F3,color:white
    style USER fill:#FF9800,color:white
```

| Aspect | Skill | Agent |
|--------|-------|-------|
| **Purpose** | Single action | Orchestrate multiple |
| **Complexity** | Simple | Can reason & chain |
| **Contains** | One action | Multiple skills |
| **Örnek** | "Get order status" | "Handle complaint" |
| **Build order** | First | After skills exist |
| **Instructions** | What this skill does | How to reason |
| **Input** | Specific parameters | Natural language |
| **Output** | Data | Formatted response |

---

## A.4 RISE vs. Classic Comparison

```mermaid
graph TD
    subgraph "Classic On-Prem"
        C1[Your Hardware]
        C2[Your Basis Team]
        C3[You Control Everything]
        C4[Modify SAP Code]
    end

    subgraph "RISE with SAP"
        R1[SAP Managed Cloud]
        R2[SAP Handles Ops]
        R3[SAP Controls Updates]
        R4[Clean Core Only]
    end

    C1 --> |"Transform to"| R1
    C2 --> |"Shifts to"| R2

    style C4 fill:#f44336,color:white
    style R4 fill:#4CAF50,color:white
```

| Aspect | Classic On-Prem | RISE with SAP |
|--------|-----------------|---------------|
| **Hosting** | Your data center | SAP managed (AWS/Azure/GCP) |
| **Basis work** | Your team | SAP handles |
| **Upgrades** | You control | SAP delivers |
| **Custom ABAP** | Full freedom | Clean Core enforced |
| **Extensions** | Inside S/4 | BTP (side-by-side) |
| **Cost model** | CapEx + licenses | Subscription (OpEx) |
| **BTP included** | Separate purchase | Credits included |
| **Disaster Recovery** | Your responsibility | SAP managed |
| **Security Patches** | You apply | SAP applies |

---

## A.5 Authentication Type Hızlı Referans

```mermaid
flowchart TD
    Q1{Need user context?}
    Q1 --> |"No"| Q2{Server-to-server?}
    Q1 --> |"Yes"| Q3{SAP-to-SAP?}

    Q2 --> |"Yes"| OAUTH[OAuth2ClientCredentials]
    Q2 --> |"No, public API"| NOAUTH[NoAuthentication]

    Q3 --> |"Yes"| SAML[OAuth2SAMLBearerAssertion]
    Q3 --> |"No, on-prem"| PP[PrincipalPropagation]

    style OAUTH fill:#4CAF50,color:white
    style SAML fill:#2196F3,color:white
    style PP fill:#FF9800,color:white
```

| Type | Use Case | Credentials Needed | User Context |
|------|----------|-------------------|--------------|
| **NoAuthentication** | Public APIs | None | No |
| **BasicAuthentication** | Simple/legacy APIs | User + Password | No (technical) |
| **OAuth2ClientCredentials** | Server-to-server | Client ID + Secret + Token URL | No |
| **OAuth2SAMLBearerAssertion** | SAP-to-SAP with user | Client ID + Secret + Token URL | Yes |
| **ClientCertificateAuthentication** | High security | Certificate + Key | No |
| **PrincipalPropagation** | On-prem with user | Trust setup | Yes |

---

## A.6 Region Hızlı Referans

| Region Code | Location | Hyperscaler | Common Use |
|-------------|----------|-------------|------------|
| `eu10` | Frankfurt, Germany | AWS | EU (GDPR), main region |
| `eu20` | Netherlands | AWS | EU backup |
| `eu11` | Frankfurt | Azure | EU Azure customers |
| `us10` | US East (Virginia) | AWS | US customers |
| `us20` | US West (Washington) | AWS | US West |
| `ap10` | Sydney, Australia | AWS | APAC customers |
| `ap11` | Singapore | AWS | Southeast Asia |
| `jp10` | Tokyo, Japan | AWS | Japan |

---

## A.7 URL Pattern Hızlı Referans

| System | URL Pattern | Örnek |
|--------|-------------|---------|
| **BTP Cockpit** | `cockpit.btp.cloud.sap` | `https://cockpit.btp.cloud.sap` |
| **BTP Trial** | `cockpit.hanatrial.ondemand.com` | `https://cockpit.hanatrial.ondemand.com` |
| **S/4HANA Cloud** | `my{number}.s4hana.ondemand.com` | `https://my300001.s4hana.ondemand.com` |
| **BAS** | `{region}.applicationstudio.cloud.sap` | `https://eu10.applicationstudio.cloud.sap` |
| **Joule Studio** | `joule-studio-{region}.cfapps.{region}.hana.ondemand.com` | `https://joule-studio-eu10.cfapps.eu10.hana.ondemand.com` |
| **SAP API Hub** | `api.sap.com` | `https://api.sap.com` |
| **Cloud Connector Admin** | `localhost:8443` | `https://localhost:8443` |

---

## A.8 Common OData Service Paths

| Service | Path |
|---------|------|
| **Sales Order** | `/sap/opu/odata/sap/API_SALES_ORDER_SRV` |
| **Business Partner** | `/sap/opu/odata/sap/API_BUSINESS_PARTNER` |
| **Material** | `/sap/opu/odata/sap/API_PRODUCT_SRV` |
| **Purchase Order** | `/sap/opu/odata/sap/API_PURCHASEORDER_PROCESS_SRV` |
| **Purchase Requisition** | `/sap/opu/odata/sap/API_PURCHASEREQ_PROCESS_SRV` |
| **GL Account** | `/sap/opu/odata/sap/API_GLACCOUNTINCHARTOFACCOUNTS_SRV` |
| **Cost Center** | `/sap/opu/odata/sap/API_COSTCENTER_SRV` |

---

## A.9 Clean Core Hızlı Referans

| Allowed in Clean Core | Not Allowed |
|-----------------------|-------------|
| Using released APIs | Modifying SAP code |
| Key User extensibility | Z-tables in S/4 core |
| Side-by-side extensions (BTP) | User exits/enhancements |
| Custom CDS views (released) | Unreleased function modules |
| Released BADIs | Direct table modifications |
| Extension fields via API | SMOD/CMOD |

---

*[İçindekilere Dön](../content.md)*

---

**Yazar:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Oluşturuldu ❤️ dünya genelindeki SAP öğrencileri için*
