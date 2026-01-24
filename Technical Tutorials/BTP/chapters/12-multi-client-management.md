# Chapter 12: Managing Multiple Clients on BTP

> *The Consultant's Survival Guide*

---

If you're a consultant, partner, or work in a shared services team, you'll manage BTP for multiple clients. This chapter shows you how to do it without losing your sanity.

---

## 12.1 The Classic Trap: One Subaccount for Everything

### What People Do Wrong

```mermaid
graph TD
    subgraph "❌ Anti-Pattern: Everything in One"
        GA[Global Account]
        GA --> SA[Single Subaccount]
        SA --> A1[Client A's Joule Agent]
        SA --> A2[Client B's Destinations]
        SA --> A3[Client C's Test App]
        SA --> A4[Your Playground]
        SA --> A5[POC from 2023]
        SA --> A6[Who knows what this is?]
    end

    style SA fill:#f44336,color:white
```

### The Problems This Causes

| Problem | Impact |
|---------|--------|
| **Entitlement chaos** | Can't tell who's using what quota |
| **Security nightmare** | Client A can see Client B's destinations |
| **Naming collisions** | `S4_ORDERS` — whose S4? |
| **Billing impossible** | Can't charge back costs per client |
| **Handover problems** | "Here's access to everything" |
| **Cleanup paralysis** | Afraid to delete anything |

---

## 12.2 Best Practice: One Subaccount Per Client

### The Recommended Structure

```mermaid
graph TD
    subgraph "Global Account: Partner Account"
        DIR1[Directory: EMEA Clients]
        DIR2[Directory: APAC Clients]
        DIR3[Directory: Internal]

        subgraph "EMEA"
            SA1[ACME_PROD]
            SA2[ACME_DEV]
            SA3[GLOBEX_PROD]
            SA4[GLOBEX_DEV]
        end

        subgraph "APAC"
            SA5[TOSHIBA_PROD]
            SA6[TOSHIBA_DEV]
        end

        subgraph "Internal"
            SA7[TEMPLATES]
            SA8[DEMOS]
            SA9[TRAINING]
        end
    end

    DIR1 --> EMEA
    DIR2 --> APAC
    DIR3 --> Internal

    style SA1 fill:#4CAF50,color:white
    style SA3 fill:#4CAF50,color:white
    style SA5 fill:#4CAF50,color:white
```

### Benefits

| Benefit | Description |
|---------|-------------|
| **Isolation** | Clients can't see each other's data |
| **Clear billing** | Usage tracked per subaccount |
| **Easy handover** | Transfer entire subaccount to client |
| **Clean naming** | No prefixes needed within subaccount |
| **Simpler security** | Assign roles per subaccount |

---

## 12.3 Naming Conventions That Scale

### Subaccount Naming

**Pattern:** `{REGION}_{CLIENT}_{ENVIRONMENT}`

```
Examples:
TR_ACME_PROD       → Turkey, Acme Corp, Production
TR_ACME_DEV        → Turkey, Acme Corp, Development
EU_GLOBEX_PROD     → Europe, Globex Inc, Production
US_INITECH_POC     → USA, Initech, Proof of Concept
```

### Destination Naming

Within client subaccount, simpler names work:

```mermaid
graph TD
    subgraph "Subaccount: TR_ACME_PROD"
        D1[S4_SALES]
        D2[S4_MATERIALS]
        D3[SF_EMPLOYEES]
        D4[ARIBA_PROCUREMENT]
    end

    subgraph "Subaccount: TR_GLOBEX_PROD"
        D5[S4_SALES]
        D6[S4_MATERIALS]
        D7[CRM_CUSTOMERS]
    end
```

**Why this works:**
- Within ACME's subaccount, `S4_SALES` is unambiguous
- No collision with GLOBEX's `S4_SALES` (different subaccount)

### For Shared Subaccounts (if you must)

If you have a shared subaccount, use full prefixes:

```
ACME_S4_PROD_SALES
GLOBEX_S4_PROD_SALES
INITECH_S4_DEV_ORDERS
```

---

## 12.4 Security & Access Management

### Identity Provider Setup

```mermaid
graph TD
    subgraph "Options"
        O1[SAP IAS<br/>Your managed IdP]
        O2[Client's IdP<br/>Azure AD, Okta, etc.]
        O3[Mixed<br/>Different per client]
    end

    subgraph "Recommendation per Client Type"
        R1[Consulting Project → SAP IAS]
        R2[Managed Service → Client's IdP]
        R3[POC/Demo → SAP IAS]
    end
```

### Role Assignments

**Your Team (Consultants):**
```yaml
Global Account Level:
  - Global Account Administrator (you)
  - Directory Administrator (team leads)

Subaccount Level (per client):
  - Subaccount Administrator
  - Cloud Foundry Space Developer
  - Destination Administrator
```

**Client Team:**
```yaml
Subaccount Level (their subaccount only):
  - Subaccount Viewer (read-only) or
  - Subaccount Administrator (full control)
  - Role specific to their apps
```

### Access Pattern for Client Handover

```mermaid
sequenceDiagram
    participant C as Consultant
    participant SA as Client Subaccount
    participant CL as Client Admin

    C->>SA: Create and configure subaccount
    Note over C,SA: Full admin during project

    C->>CL: Add client as Subaccount Admin
    CL->>SA: Client gains access

    Note over CL,SA: Transition period: Both have access

    CL->>C: Remove consultant access
    Note over CL,SA: Client owns subaccount
```

---

## 12.5 Entitlement Distribution

### Visualizing Entitlements

```mermaid
pie title "AI Units Distribution"
    "ACME_PROD" : 40
    "ACME_DEV" : 10
    "GLOBEX_PROD" : 30
    "GLOBEX_DEV" : 10
    "Internal" : 10
```

### Managing Quotas

In BTP Cockpit:

1. **Global Account → Entitlements**
2. **Entity Assignments** (formerly "Manage Quota")
3. Distribute to subaccounts:

```yaml
AI Core (AI Units):
  Total: 1000 units/month
  Distribution:
    - ACME_PROD: 400
    - ACME_DEV: 100
    - GLOBEX_PROD: 300
    - GLOBEX_DEV: 100
    - Internal: 100

Cloud Foundry Runtime:
  Total: 16 GB
  Distribution:
    - ACME_PROD: 4 GB
    - ACME_DEV: 2 GB
    - GLOBEX_PROD: 4 GB
    - GLOBEX_DEV: 2 GB
    - Internal: 4 GB
```

---

## 12.6 Cost Allocation and Chargeback

### Tracking Costs Per Client

```mermaid
graph TD
    subgraph "BTP Cockpit"
        UA[Usage Analytics]
    end

    subgraph "Per Subaccount Metrics"
        M1[AI Units consumed]
        M2[CF Runtime hours]
        M3[HANA storage]
        M4[API calls]
    end

    UA --> M1
    UA --> M2
    UA --> M3
    UA --> M4

    subgraph "Client Invoice"
        I[Monthly cost report per client]
    end

    M1 --> I
    M2 --> I
    M3 --> I
    M4 --> I
```

### Export and Report

1. **BTP Cockpit → Usage Analytics**
2. **Select subaccount** (or directory)
3. **Export to CSV**
4. **Build monthly reports per client**

### Sample Cost Report Structure

```markdown
# Monthly BTP Cost Report - ACME Corp

Period: January 2026

| Service | Usage | Unit Cost | Total |
|---------|-------|-----------|-------|
| AI Core | 350 AI Units | €0.10 | €35.00 |
| CF Runtime | 720 GB-hours | €0.05 | €36.00 |
| HANA Cloud | 50 GB | €2.00 | €100.00 |
| Destination Service | 10,000 calls | €0.001 | €10.00 |
| **Total** | | | **€181.00** |
```

---

## 12.7 Template-Based Setup

### The Golden Template Approach

```mermaid
flowchart TD
    subgraph "Template Subaccount"
        T[TEMPLATES]
        T --> TA[Standard Agents]
        T --> TD[Destination Templates]
        T --> TF[Fiori App Templates]
    end

    subgraph "New Client Setup"
        C[Create Client Subaccount]
        C --> |"Copy from"| T
        C --> CUST[Customize for Client]
        CUST --> DEP[Deploy]
    end
```

### Standard Setup Checklist

When onboarding a new client:

```yaml
Client Onboarding Checklist:
  Subaccount:
    - [ ] Create subaccount with naming convention
    - [ ] Enable Cloud Foundry
    - [ ] Assign entitlements

  Connectivity:
    - [ ] Create destination to S/4HANA
    - [ ] Create destination to SuccessFactors (if needed)
    - [ ] Set up Cloud Connector (if on-prem)

  Identity:
    - [ ] Configure trust with IdP
    - [ ] Create initial users
    - [ ] Set up role collections

  Services:
    - [ ] Deploy standard Fiori apps
    - [ ] Set up Joule agents from template
    - [ ] Configure Work Zone

  Documentation:
    - [ ] Document subaccount details
    - [ ] Document credentials (secure storage)
    - [ ] Create handover package
```

---

## 12.8 Multi-Client Joule Agents

### Option 1: Separate Agents Per Client

```mermaid
graph TD
    subgraph "ACME Subaccount"
        A1[Customer Service Agent<br/>for ACME]
    end

    subgraph "GLOBEX Subaccount"
        A2[Customer Service Agent<br/>for GLOBEX]
    end
```

**Pros:** Full isolation, client-specific customization
**Cons:** More maintenance, changes replicated manually

### Option 2: Shared Agent Template

Build once, deploy to each:

```mermaid
flowchart TD
    subgraph "Template"
        T[Generic Customer Service Agent]
    end

    subgraph "ACME"
        A1[Deployed Agent]
        A1G[ACME Grounding Docs]
        A1D[ACME Destinations]
    end

    subgraph "GLOBEX"
        A2[Deployed Agent]
        A2G[GLOBEX Grounding Docs]
        A2D[GLOBEX Destinations]
    end

    T --> |"Deploy"| A1
    T --> |"Deploy"| A2
    A1G --> A1
    A1D --> A1
    A2G --> A2
    A2D --> A2
```

---

## 12.9 Disaster Recovery Considerations

### Per-Client Backup Strategy

```yaml
Backup Frequency:
  Destinations: Weekly export
  Agents: After each change
  Fiori Apps: Git repository
  Configurations: Document + export

Storage:
  Location: Secure company storage
  Format: JSON exports + documentation
  Retention: 90 days minimum
```

### Quick Recovery Plan

```mermaid
flowchart TD
    DR[Disaster Detected] --> ASSESS{Assess Scope}
    ASSESS --> |"Single destination"| FIX1[Recreate from backup]
    ASSESS --> |"Entire subaccount"| FIX2[Create new + restore all]
    ASSESS --> |"Global account"| FIX3[SAP Support + full restore]

    FIX1 --> TEST[Test functionality]
    FIX2 --> TEST
    FIX3 --> TEST

    TEST --> DONE[Back to normal]
```

---

## Key Takeaways

1. **One subaccount per client** — Essential for isolation
2. **Naming conventions** — `{REGION}_{CLIENT}_{ENV}`
3. **Clear entitlement distribution** — Track quotas per client
4. **Proper access control** — Consultants vs. client roles
5. **Template approach** — Standardize and replicate
6. **Cost tracking** — Enable chargeback

---

## What's Next?

Now let's look at deploying the same solution across multiple clients—template-based vs. multi-tenant approaches.

---

*[Previous: Chapter 11 – Agent Lifecycle & Deployment](11-agent-lifecycle.md) | [Next: Chapter 13 – Cross-Customer Deployments](13-cross-customer-deployments.md)*

*[Back to Table of Contents](../content.md)*

---

**Author:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Created with ❤️ for SAP learners worldwide*
