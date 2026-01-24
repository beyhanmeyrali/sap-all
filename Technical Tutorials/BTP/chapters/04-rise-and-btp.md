# Chapter 4: How RISE and BTP Fit Together

> *The Neighborhood View*

---

## 4.1 The Neighborhood View: RISE House + BTP Extension Wing

Here's the picture that makes it click:

```mermaid
graph LR
    subgraph "THE CLOUD NEIGHBORHOOD"
        subgraph "RISE HOUSE - S/4HANA Cloud"
            S4_STD[Standard ERP]
            S4_MD[Master Data]
            S4_TX[Core Transactions]
            S4_MGD[SAP-Managed]
        end

        subgraph "BTP EXTENSION WING"
            BTP_FIORI[Custom Fiori Apps]
            BTP_RAP[RAP ABAP Services]
            BTP_JOULE[Joule Agents/Skills]
            BTP_INT[Integrations]
            BTP_WF[Workflows]
        end
    end

    S4_STD --> |"OData APIs"| BTP_FIORI
    S4_MD --> |"Destinations"| BTP_RAP
    S4_TX --> |"Events"| BTP_JOULE

    style S4_STD fill:#4CAF50,color:white
    style BTP_FIORI fill:#2196F3,color:white
    style BTP_JOULE fill:#FF9800,color:white
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

```mermaid
flowchart TD
    CONTRACT[RISE Contract] --> |"Includes"| BTP_CREDITS[BTP Credits Pool]

    BTP_CREDITS --> |"Use for"| USE1[Custom Apps]
    BTP_CREDITS --> |"Use for"| USE2[Integration Suite]
    BTP_CREDITS --> |"Use for"| USE3[AI Core / Joule]
    BTP_CREDITS --> |"Use for"| USE4[ABAP Environment]
    BTP_CREDITS --> |"Use for"| USE5[HANA Cloud]

    USE1 --> MONITOR[Monitor Usage!<br/>Credits are not unlimited]

    style BTP_CREDITS fill:#4CAF50,color:white
    style MONITOR fill:#f44336,color:white
```

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

```mermaid
graph TD
    subgraph "S/4HANA On-Prem"
        SE80[SE80: Z-programs]
        SE11[SE11: Z-tables]
        EXITS[User Exits & BADIs]
        FLP[Embedded Fiori Launchpad]

        SE80 --> SAME[Everything in One System]
        SE11 --> SAME
        EXITS --> SAME
        FLP --> SAME
    end

    style SAME fill:#FF9800,color:white
```

### RISE World

```mermaid
graph TD
    subgraph "S/4HANA Cloud - RISE"
        S4[Clean Core]
        S4 --> |"❌"| NO1[Can't modify core]
        S4 --> |"❌"| NO2[Can't add Z-tables]
        S4 --> |"❌"| NO3[Can't use unreleased APIs]
        S4 --> |"✓"| YES1[Configure using Key User]
        S4 --> |"✓"| YES2[Exposes APIs for external use]
    end

    S4 --> |"APIs<br/>OData, Events"| BTP

    subgraph "BTP Subaccount"
        BTP[Your Workshop]
        BTP --> |"✓"| CAN1[Custom ABAP via RAP]
        BTP --> |"✓"| CAN2[Custom Fiori Apps]
        BTP --> |"✓"| CAN3[Joule Agents & Skills]
        BTP --> |"✓"| CAN4[Integration Flows]
        BTP --> |"✓"| CAN5[Custom Tables in HANA]
    end

    style S4 fill:#f44336,color:white
    style BTP fill:#4CAF50,color:white
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

```mermaid
graph TD
    subgraph "Decision Matrix"
        Q1{Where does it live?}

        Q1 --> |"Standard ERP Logic"| RISE[RISE S/4HANA]
        Q1 --> |"Custom Extensions"| BTP[BTP]
        Q1 --> |"Master Data"| RISE
        Q1 --> |"AI/Joule Agents"| BTP
        Q1 --> |"Integrations"| BTP
        Q1 --> |"Custom Reports"| BTP
    end

    style RISE fill:#4CAF50,color:white
    style BTP fill:#2196F3,color:white
```

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

## 4.5 Practical Example: Customer Wants a Custom Dashboard

### Old Way (On-Prem)

```mermaid
sequenceDiagram
    participant DEV as Developer
    participant SE80 as SE80
    participant SE11 as SE11
    participant WEBIDE as WebIDE
    participant FLP as On-Prem FLP

    DEV->>SE80: Create Z-program for data
    DEV->>SE11: Create Z-CDS views
    DEV->>WEBIDE: Build Fiori app
    DEV->>FLP: Deploy to on-prem FLP
    Note over DEV,FLP: All in one system
```

### RISE + BTP Way

```mermaid
sequenceDiagram
    participant S4 as RISE S/4HANA
    participant BTP as BTP Subaccount
    participant BAS as Business App Studio
    participant WZ as SAP Build Work Zone

    S4->>S4: Enable standard OData services
    BTP->>BTP: Create destination to S/4
    BAS->>BAS: Build Fiori app consuming OData
    BAS->>BTP: Deploy to BTP
    BTP->>WZ: Configure in Work Zone
    Note over S4,WZ: S/4 core stays clean!
```

The dashboard **looks the same** to users, but the architecture is fundamentally different.

---

## 4.6 The Complete Architecture Picture

```mermaid
graph TD
    subgraph "End Users"
        USER[Business Users]
    end

    subgraph "Access Layer"
        WZ[SAP Build Work Zone]
        JOULE[Joule Copilot]
    end

    subgraph "BTP Extension Layer"
        FIORI[Custom Fiori Apps]
        RAP[RAP Services]
        AGENTS[Joule Agents]
        CPI[Integration Suite]
    end

    subgraph "Connection Layer"
        DEST[Destinations]
        CC[Cloud Connector]
    end

    subgraph "RISE S/4HANA Cloud"
        S4[Clean Core S/4HANA]
        API[Released OData APIs]
    end

    subgraph "On-Premise Systems"
        ECC[Legacy ECC]
        THIRD[Third-Party Systems]
    end

    USER --> WZ
    USER --> JOULE
    WZ --> FIORI
    JOULE --> AGENTS
    FIORI --> RAP
    AGENTS --> RAP
    RAP --> DEST
    DEST --> API
    API --> S4
    CPI --> CC
    CC --> ECC
    CC --> THIRD

    style S4 fill:#4CAF50,color:white
    style BTP fill:#2196F3,color:white
    style USER fill:#FF9800,color:white
```

---

## 4.7 Real-World URL Examples

When working with RISE + BTP, you'll encounter these URL patterns:

| System | URL Pattern | Example |
|--------|-------------|---------|
| **BTP Cockpit** | `cockpit.btp.cloud.sap` | `https://cockpit.btp.cloud.sap/cockpit/?idp=...` |
| **S/4HANA Cloud** | `my{number}.s4hana.ondemand.com` | `https://my300001.s4hana.ondemand.com` |
| **Business App Studio** | `{region}.applicationstudio.cloud.sap` | `https://eu10.applicationstudio.cloud.sap` |
| **Joule Studio** | `joule-studio-{region}.cfapps.{region}.hana.ondemand.com` | `https://joule-studio-eu10.cfapps.eu10.hana.ondemand.com` |
| **OData Service** | `{s4_url}/sap/opu/odata/sap/{service}` | `https://my300001.s4hana.ondemand.com/sap/opu/odata/sap/API_SALES_ORDER_SRV` |

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

---

**Author:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Created with ❤️ for SAP learners worldwide*
