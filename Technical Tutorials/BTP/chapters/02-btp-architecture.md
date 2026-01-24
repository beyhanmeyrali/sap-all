# Chapter 2: The BTP Architecture

> *Your New Apartment Building*

---

Imagine SAP BTP is a **massive apartment building** owned by SAP. You (or your company) rent space in it. Let's explore each level.

---

## 2.1 Global Account – Your Lease Contract with SAP

When you sign up for SAP BTP (trial or paid), SAP gives you one **Global Account**.

```mermaid
graph TD
    subgraph "SAP BTP"
        SAP[SAP as Landlord]
        SAP --> GA[Your Global Account]
        GA --> |"Contains"| ENT[Entitlements & Quotas]
        GA --> |"Contains"| BILL[Billing Info]
        GA --> |"Contains"| REGIONS[Available Regions]
    end

    style SAP fill:#FF9800,color:white
    style GA fill:#2196F3,color:white
```

This is basically **your name on the building's lease agreement**. It knows:

- ✅ How much you're allowed to use (quotas, entitlements)
- ✅ How you're billed
- ✅ Which regions (data centers) you can use

You normally have **only one global account per company** (sometimes more for very large orgs or separate contracts).

> **Think**: *"This is Acme Corp's official contract folder with SAP."*

### In the BTP Cockpit

When you log in, you see your Global Account at the top level. Everything else is nested inside it.

**URL Pattern:** `https://cockpit.btp.cloud.sap/cockpit/?idp=...`

---

## 2.2 Directories – Organizing Floors (Optional)

You can create **Directories** inside your global account to organize things logically.

```mermaid
graph TD
    GA[Global Account: Acme Corp]

    GA --> DIR1[Directory: Production]
    GA --> DIR2[Directory: Development & Test]
    GA --> DIR3[Directory: HR Projects]
    GA --> DIR4[Directory: EMEA Region]

    style GA fill:#2196F3,color:white
    style DIR1 fill:#4CAF50,color:white
    style DIR2 fill:#FF9800,color:white
    style DIR3 fill:#9C27B0,color:white
    style DIR4 fill:#00BCD4,color:white
```

Examples:
- One directory called **"Production"**
- One called **"Development & Test"**
- One called **"HR Projects"**
- One called **"EMEA Region"**

Directories are **just folders** to group subaccounts. They're optional—you can skip them and put everything directly under the global account.

> **Analogy**: Floors in the building labeled by purpose or team.

---

## 2.3 Subaccounts – Your Actual Apartments

Here's where the real work happens!

```mermaid
graph TD
    GA[Global Account]
    GA --> DIR[Directory: Finance]

    DIR --> SUB1[Subaccount: FIN_DEV]
    DIR --> SUB2[Subaccount: FIN_TEST]
    DIR --> SUB3[Subaccount: FIN_PROD]

    subgraph "What Lives in a Subaccount"
        DEST[Destinations]
        SERV[Service Instances]
        APPS[Deployed Apps]
        USERS[Users & Roles]
        AGENT[Joule Agents]
    end

    SUB3 --> DEST
    SUB3 --> SERV
    SUB3 --> APPS
    SUB3 --> USERS
    SUB3 --> AGENT

    style GA fill:#2196F3,color:white
    style SUB3 fill:#4CAF50,color:white
```

A **Subaccount** is the place where you actually do things:

- ✅ Create apps
- ✅ Enable services (Joule, destinations, databases, AI Core, etc.)
- ✅ Deploy skills, agents, destinations
- ✅ Manage users & roles
- ✅ Choose a region (e.g., Europe Frankfurt, US East)
- ✅ Get separate quotas

### Key Point: Isolation

Each subaccount is **almost completely isolated** from the others—even if they're under the same global account.

This is powerful for:

| Use Case | Why Isolation Helps |
|----------|---------------------|
| **Dev vs Test vs Prod** | A broken test app can't crash production |
| **Different teams/projects** | HR team won't accidentally mess with Finance |
| **Different regions** | EU data stays in EU (GDPR compliance) |
| **Cost control** | See exactly what each project consumes |
| **Security** | Revoke access to one subaccount without affecting others |

---

## 2.4 Why Isolation Matters: Dev vs. Test vs. Prod

In classic SAP, you had separate systems (development, quality, production) with transport paths between them.

In BTP, **subaccounts serve a similar purpose**:

```mermaid
graph TD
    GA[Global Account: Acme Corp]

    subgraph "HR Projects"
        HR_DEV[HR_DEV<br/>Low quotas, sandbox]
        HR_TEST[HR_TEST<br/>Integration testing]
        HR_PROD[HR_PROD<br/>Live users]
    end

    subgraph "Finance Projects"
        FIN_DEV[FIN_DEV]
        FIN_PROD[FIN_PROD]
    end

    GA --> HR_DEV
    GA --> HR_TEST
    GA --> HR_PROD
    GA --> FIN_DEV
    GA --> FIN_PROD

    HR_DEV --> |"Deploy"| HR_TEST
    HR_TEST --> |"Deploy"| HR_PROD

    style HR_PROD fill:#4CAF50,color:white
    style FIN_PROD fill:#4CAF50,color:white
    style HR_DEV fill:#FF9800,color:white
    style HR_TEST fill:#2196F3,color:white
```

Each subaccount has its own:
- Destinations (pointing to different backends)
- User assignments
- Service instances
- Deployed applications

---

## 2.5 Regions, Compliance, and Data Residency

When you create a subaccount, you **choose a region** (data center location):

```mermaid
graph LR
    subgraph "Europe"
        EU10[eu10 - Frankfurt]
        EU20[eu20 - Netherlands]
    end

    subgraph "Americas"
        US10[us10 - US East]
        US20[us20 - US West]
    end

    subgraph "Asia Pacific"
        AP10[ap10 - Sydney]
        JP10[jp10 - Tokyo]
    end

    style EU10 fill:#2196F3,color:white
    style EU20 fill:#2196F3,color:white
    style US10 fill:#4CAF50,color:white
    style US20 fill:#4CAF50,color:white
    style AP10 fill:#FF9800,color:white
    style JP10 fill:#FF9800,color:white
```

| Region | Location | Common Use |
|--------|----------|------------|
| `eu10` | Frankfurt, Germany | EU customers, GDPR |
| `eu20` | Netherlands | EU backup |
| `us10` | US East (Virginia) | US customers |
| `ap10` | Sydney | APAC customers |
| `jp10` | Tokyo | Japan customers |

**Why this matters**:
- GDPR requires EU citizen data to stay in EU
- Some industries have data residency requirements
- Network latency considerations

---

## 2.6 Entitlements & Quotas – How Much You Can Use

**Entitlements** = What services you're *allowed* to use
**Quotas** = How *much* of each service you can use

```mermaid
flowchart TD
    CONTRACT[SAP Contract] --> GA[Global Account<br/>Pool of Entitlements]

    GA --> |"Assign"| SUB1[DEV Subaccount<br/>20 HANA units]
    GA --> |"Assign"| SUB2[TEST Subaccount<br/>30 HANA units]
    GA --> |"Assign"| SUB3[PROD Subaccount<br/>50 HANA units]

    style CONTRACT fill:#9C27B0,color:white
    style GA fill:#2196F3,color:white
    style SUB3 fill:#4CAF50,color:white
```

Think of it like a buffet:
- **Entitlements**: Which dishes you're allowed to take (included in your plan)
- **Quotas**: How many plates of each dish (limits)

### How Entitlements Flow

```
SAP Contract → Global Account (pool of entitlements)
                    ↓
              Assign to Subaccounts
                    ↓
              Each subaccount gets a portion
```

For example:
- Global Account has 100 HANA units
- DEV subaccount gets 20 units
- TEST subaccount gets 30 units
- PROD subaccount gets 50 units

### Viewing Entitlements

In BTP Cockpit:
1. Go to your Global Account
2. Click **Entitlements** → **Entity Assignments**
3. See what's assigned where

---

## 2.7 The Complete Picture

```mermaid
graph TD
    subgraph "SAP BTP Architecture"
        GA[Global Account<br/>ACME_CORP]

        subgraph "Directory: Production"
            PROD_S4[Subaccount: S4_PROD<br/>Region: eu10]
            PROD_BTP[Subaccount: BTP_PROD<br/>Region: eu10]
        end

        subgraph "Directory: Non-Production"
            DEV[Subaccount: DEV<br/>Region: eu10]
            TEST[Subaccount: TEST<br/>Region: eu10]
            SANDBOX[Subaccount: SANDBOX<br/>Region: us10]
        end

        GA --> PROD_S4
        GA --> PROD_BTP
        GA --> DEV
        GA --> TEST
        GA --> SANDBOX
    end

    subgraph "Inside PROD_BTP Subaccount"
        DEST[Destinations]
        JOULE[Joule Agents]
        FIORI[Fiori Apps]
        HANA[HANA Cloud]
        INT[Integration Suite]
    end

    PROD_BTP --> DEST
    PROD_BTP --> JOULE
    PROD_BTP --> FIORI
    PROD_BTP --> HANA
    PROD_BTP --> INT

    style GA fill:#9C27B0,color:white
    style PROD_BTP fill:#4CAF50,color:white
```

---

## Quick Building Analogy Summary

| BTP Concept | Building Analogy |
|-------------|------------------|
| **Global Account** | The lease / ownership papers |
| **Directory** | A labeled floor (optional organizer) |
| **Subaccount** | One actual apartment |
| **Region** | Which building location (city) |
| **Entitlements** | What appliances you're allowed to have |
| **Quotas** | How many of each appliance |

---

## The BTP Cockpit View

```mermaid
graph TD
    TOP[Global Account] --> |"Click to enter"| INSIDE

    subgraph INSIDE[Inside Global Account]
        DIR1[Directory 1]
        DIR2[Directory 2]

        DIR1 --> SUB_A[Subaccount A]
        DIR1 --> SUB_B[Subaccount B]
        DIR2 --> SUB_C[Subaccount C]
    end

    style TOP fill:#2196F3,color:white
    style SUB_A fill:#4CAF50,color:white
    style SUB_B fill:#4CAF50,color:white
    style SUB_C fill:#4CAF50,color:white
```

**Most of your time** (95%+) is spent inside a subaccount—that's where the action happens:
- Creating destinations
- Building skills
- Deploying agents
- Managing users

---

## Real-World Example: Accessing BTP Cockpit

```
1. Open browser
2. Navigate to: https://cockpit.btp.cloud.sap
3. Log in with your SAP Universal ID
4. Select your Global Account
5. Navigate to your Subaccount
6. You're in!
```

---

## Key Takeaways

1. **Global Account** = Your contract with SAP (one per company usually)
2. **Directories** = Optional folders to organize (by team, region, purpose)
3. **Subaccounts** = Where you actually work (isolated environments)
4. **Regions** = Data center location (compliance + performance)
5. **Entitlements/Quotas** = What you can use and how much

---

## What's Next?

Now you understand the BTP structure. But how does this relate to **RISE with SAP**? That's what's confusing many old-school SAP folks. Let's demystify it.

---

*[Previous: Chapter 1 – What Is SAP BTP?](01-what-is-sap-btp.md) | [Next: Chapter 3 – RISE with SAP Demystified](03-rise-with-sap.md)*

*[Back to Table of Contents](../content.md)*

---

**Author:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Created with ❤️ for SAP learners worldwide*
