# Kısım 8: Joule Fundamentals

> *SAP's AI Copilot Explained*

---

Welcome to Part IV. This is where things get exciting. Joule is SAP's AI assistant—the generative AI copilot that will transform how users interact with SAP systems. Let's understand it properly.

---

## 8.1 What Is Joule? (The SAP AI Copilot)

**Joule** (pronounced "jewel") is SAP's generative AI assistant that:

- Lives inside SAP applications (S/4HANA, SuccessFactors, Ariba, Concur, etc.)
- Understands SAP data, processes, and business context
- Can be extended with custom **skills** and **agents**
- Uses natural language to interact with users

> **Think of it as**: A company-specific ChatGPT that knows your SAP landscape and can actually DO things—not just answer questions.

### Where Joule Appears

```mermaid
graph TD
    subgraph "SAP Applications with Joule"
        S4[S/4HANA Cloud]
        SF[SuccessFactors]
        AR[Ariba]
        CC[Concur]
        AC[Analytics Cloud]
        BW[Build Work Zone]
    end

    subgraph "Joule UI"
        J[Joule Chat Interface]
    end

    S4 --> J
    SF --> J
    AR --> J
    CC --> J
    AC --> J
    BW --> J

    J --> |"Natural Language"| U[User]
```

### Real-World Joule Conversations

**Örnek 1: HR Manager in SuccessFactors**
```
User: "Show me employees whose probation ends this month"
Joule: "I found 12 employees with probation ending in January 2026..."
       [Displays list with names, departments, end dates]
       "Would you like me to send a reminder to their managers?"
```

**Örnek 2: Procurement in Ariba**
```
User: "What's the status of PO 4500012345?"
Joule: "Purchase Order 4500012345:
        - Vendor: ABC Electronics GmbH
        - Total: €45,000
        - Status: Partially Delivered (3 of 5 items received)
        - Expected delivery of remaining items: Feb 15, 2026"
```

**Örnek 3: Finance in S/4HANA**
```
User: "Create a sales order for customer Acme Corp, 100 units of material FG-1000"
Joule: "I'll create that sales order:
        - Customer: Acme Corp (1000234)
        - Material: FG-1000 (Finished Goods Widget)
        - Quantity: 100 EA
        - Price: €50.00 each = €5,000.00 total

        Should I proceed?"
User: "Yes"
Joule: "Sales Order 12345678 created successfully."
```

---

## 8.2 Skills vs. Agents: The Key Difference

This is the most important concept. Let's get it crystal clear.

### Skill = One Superpower

A **skill** is a single, specific capability:

```mermaid
graph LR
    U[User Request] --> S[Skill]
    S --> A[Single Action]
    A --> R[Result]

    style S fill:#4CAF50,color:white
```

**Skill Örneks:**
| Skill Name | What It Does | Input | Output |
|------------|--------------|-------|--------|
| `GetSalesOrderStatus` | Looks up order status | Order Number | Status, dates, amounts |
| `CalculateDeliveryDate` | Estimates delivery | Material, Quantity, Location | Estimated date |
| `CreatePurchaseRequisition` | Creates PR | Material, Qty, Cost Center | PR Number |
| `SendTeamsMessage` | Posts to MS Teams | Channel, Message | Confirmation |
| `LookupCustomerCredit` | Checks credit limit | Customer ID | Credit info |

### Agent = Smart Orchestrator

An **agent** decides which skills to use and chains them together:

```mermaid
graph TD
    U[User: "Handle complaint for order 12345"] --> AG[Agent]

    AG --> |Step 1| S1[GetSalesOrderStatus]
    S1 --> |"Order found, status: Delayed"| AG

    AG --> |Step 2| S2[CheckInventoryAvailability]
    S2 --> |"Item in stock"| AG

    AG --> |Step 3| S3[CreateReturnOrder]
    S3 --> |"Return RO-789 created"| AG

    AG --> |Step 4| S4[SendApologyEmail]
    S4 --> |"Email sent"| AG

    AG --> R[Response to User]

    style AG fill:#2196F3,color:white
    style S1 fill:#4CAF50,color:white
    style S2 fill:#4CAF50,color:white
    style S3 fill:#4CAF50,color:white
    style S4 fill:#4CAF50,color:white
```

**The Agent's Internal Reasoning:**
```
User wants to handle a complaint about order 12345.

Step 1: I need to understand the order situation.
        → Using skill: GetSalesOrderStatus(12345)
        → Result: Order is delayed by 2 weeks

Step 2: Can we offer immediate replacement?
        → Using skill: CheckInventoryAvailability(material: FG-1000)
        → Result: Yes, 50 units in stock

Step 3: Create the return/exchange
        → Using skill: CreateReturnOrder(originalOrder: 12345, reason: "Delay compensation")
        → Result: Return order RO-789 created

Step 4: Notify the customer
        → Using skill: SendApologyEmail(customer: Acme Corp, context: delay + replacement)
        → Result: Email sent successfully

Response: "I've handled the complaint for order 12345. Created return order RO-789
          and sent an apology email to Acme Corp with replacement shipment details."
```

### Comparison Summary

```mermaid
graph TB
    subgraph "SKILL"
        S[Single Action]
        S --> |"Örnek"| SE["GetOrderStatus(12345)"]
    end

    subgraph "AGENT"
        A[Multiple Skills + Reasoning]
        A --> |"Örnek"| AE["Handle entire complaint process"]
    end

    S --> |"Building Block"| A
```

| Aspect | Skill | Agent |
|--------|-------|-------|
| **Purpose** | Do ONE specific thing | Orchestrate MANY things |
| **Complexity** | Simple, predictable | Can reason and adapt |
| **Contains** | Single action or API call | Multiple skills + logic |
| **Örnek** | "Get order status" | "Handle customer complaint" |
| **When to build** | First (agents need skills) | After skills exist |
| **User interaction** | Usually invisible | The "face" users talk to |

---

## 8.3 How Skills "Do" Things (The Action Flow)

When Joule needs to actually DO something (not just chat), it uses this flow:

```mermaid
sequenceDiagram
    participant U as User
    participant J as Joule
    participant AG as Agent
    participant SK as Skill
    participant AP as Action Project
    participant DS as Destination Service
    participant BE as Backend API

    U->>J: "What's the status of order 12345?"
    J->>AG: Route to appropriate agent
    AG->>SK: Invoke GetOrderStatus skill
    SK->>AP: Call action "fetchOrderStatus"
    AP->>DS: Lookup destination "ACME_S4_ORDERS"
    DS-->>AP: URL + Auth Token
    AP->>BE: GET /sap/opu/odata/sap/API_SALES_ORDER_SRV/A_SalesOrder('12345')
    BE-->>AP: {SalesOrder: "12345", Status: "Delivered", ...}
    AP-->>SK: Order data
    SK-->>AG: Formatted response
    AG-->>J: Natural language response
    J-->>U: "Order 12345 was delivered on Jan 20, 2026"
```

### The Components Explained

| Component | Nedir | Where It Lives |
|-----------|------------|----------------|
| **Joule** | The AI chat interface | SAP apps, Work Zone |
| **Agent** | The "brain" that decides what to do | Joule Studio |
| **Skill** | A specific capability | Joule Studio |
| **Action Project** | API wrapper (SAP Build) | SAP Build lobby |
| **Destination** | Connection configuration | BTP subaccount |
| **Backend API** | The actual service | S/4HANA, external API |

---

## 8.4 Why Destinations Are Required for Skills

This connects back to Kısım 5. Here's why you cannot skip destinations:

### Security Requirements

```mermaid
graph TD
    subgraph "❌ NOT ALLOWED"
        SK1[Skill with hardcoded URL]
        SK1 --> |"https://s4.acme.com + password"| API1[API]
    end

    subgraph "✅ REQUIRED"
        SK2[Skill]
        SK2 --> AP[Action Project]
        AP --> |"References"| DS[Destination: ACME_S4]
        DS --> |"Secure lookup"| API2[API]
    end

    style SK1 fill:#f44336,color:white
    style SK2 fill:#4CAF50,color:white
```

### What Would Go Wrong Without Destinations

| Issue | Impact |
|-------|--------|
| **Passwords in code** | Security breach if skill is exported |
| **URL changes** | Must rebuild and redeploy skill |
| **No password rotation** | Can't change creds without touching skills |
| **No audit trail** | Can't track who calls what |
| **Dev/Prod confusion** | Risk of calling wrong environment |

### The Proper Architecture

```mermaid
graph TB
    subgraph "Joule Studio (Your Subaccount)"
        SK[Skill: GetSalesOrder]
        AG[Agent: Sales Assistant]
    end

    subgraph "SAP Build (Same Subaccount)"
        AP[Action Project: S4 Sales APIs]
    end

    subgraph "BTP Connectivity"
        DS[Destination: ACME_S4_PROD_ORDERS]
    end

    subgraph "Backend"
        S4[S/4HANA Cloud]
    end

    AG --> SK
    SK --> AP
    AP --> DS
    DS --> |"OAuth2 + Token"| S4

    style DS fill:#FF9800,color:white
```

---

## 8.5 Real-World Skill Örneks

### Örnek 1: Sales Order Status Lookup

**Scenario:** Users ask "What's the status of order X?"

**Destination:**
```
Name:        ACME_S4_PROD_SALES
Type:        HTTP
URL:         https://my300001.s4hana.ondemand.com
Proxy Type:  Internet
Auth:        OAuth2ClientCredentials
Client ID:   sb-xsuaa-acme-sales!t12345
Secret:      ********
Token URL:   https://my300001.authentication.eu10.hana.ondemand.com/oauth/token
```

**Action Project API:**
```yaml
# OpenAPI spec used in Action Project
paths:
  /sap/opu/odata/sap/API_SALES_ORDER_SRV/A_SalesOrder('{SalesOrder}'):
    get:
      operationId: getSalesOrder
      parameters:
        - name: SalesOrder
          in: path
          required: true
          schema:
            type: string
```

**Skill Configuration:**
- Name: `GetSalesOrderStatus`
- Action: `getSalesOrder` from Action Project
- Input: `SalesOrder` (string)
- Output: Order details (formatted for user)

---

### Örnek 2: Create Purchase Requisition

**Scenario:** Users say "I need 50 laptops for IT department"

**Destination:**
```
Name:        ACME_S4_PROD_PROCUREMENT
Type:        HTTP
URL:         https://my300001.s4hana.ondemand.com
Proxy Type:  Internet
Auth:        OAuth2SAMLBearerAssertion
Client ID:   sb-xsuaa-acme-proc!t12346
Secret:      ********
Token URL:   https://my300001.authentication.eu10.hana.ondemand.com/oauth/token
```

**Why SAMLBearer?** Because we need the user's identity to flow to S/4—the PR should be created by the actual requesting user, not a technical user.

**Action Project API:**
```yaml
paths:
  /sap/opu/odata/sap/API_PURCHASEREQ_PROCESS_SRV/A_PurchaseRequisitionHeader:
    post:
      operationId: createPurchaseRequisition
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PurchaseRequisition'
```

**Skill Flow:**
```mermaid
graph LR
    U["I need 50 laptops for IT"] --> SK[CreatePR Skill]
    SK --> |"Parse request"| P[Parameters]
    P --> |"Material: LAPTOP-001"| AP[Action]
    P --> |"Quantity: 50"| AP
    P --> |"Cost Center: IT-001"| AP
    AP --> S4[S/4HANA]
    S4 --> |"PR 10001234"| R[Result]
    R --> U2["Created PR 10001234 for 50 laptops"]
```

---

### Örnek 3: External Weather API for Logistics

**Scenario:** Logistics planner asks "What's the weather at our Frankfurt warehouse?"

**Destination:**
```
Name:        EXTERNAL_WEATHER_API
Type:        HTTP
URL:         https://api.openweathermap.org/data/2.5
Proxy Type:  Internet
Auth:        NoAuthentication

Additional Properties:
URL.headers.x-api-key: abc123def456
```

**Skill Configuration:**
- Name: `GetWarehouseWeather`
- Maps warehouse code → city coordinates
- Calls weather API
- Returns formatted weather info

**Why is this useful?** The agent can factor weather into delivery promises:
```
User: "Can we deliver to Munich today?"
Agent:
  1. Check inventory (Skill A)
  2. Check weather at Munich warehouse (Skill B)
  3. Weather shows heavy snow → delay likely
  4. Response: "We have stock, but heavy snow in Munich may delay delivery by 1 day"
```

---

## 8.6 The Joule Architecture Overview

```mermaid
graph TB
    subgraph "End Users"
        U1[S/4HANA User]
        U2[Work Zone User]
        U3[SuccessFactors User]
    end

    subgraph "Joule UI Layer"
        JC[Joule Chat Component]
    end

    subgraph "Joule Runtime (AI Core)"
        JR[Joule Runtime]
        LLM[LLM - GPT/Claude/Gemini]
    end

    subgraph "Joule Studio (Design Time)"
        AG[Agents]
        SK[Skills]
        GR[Grounding Documents]
    end

    subgraph "SAP Build"
        AP1[Action Project 1]
        AP2[Action Project 2]
    end

    subgraph "BTP Services"
        DS[Destination Service]
        AI[AI Core]
    end

    subgraph "Backends"
        S4[S/4HANA]
        SF[SuccessFactors]
        EXT[External APIs]
    end

    U1 --> JC
    U2 --> JC
    U3 --> JC

    JC --> JR
    JR --> LLM
    JR --> AG
    AG --> SK
    SK --> AP1
    SK --> AP2
    AG --> GR

    AP1 --> DS
    AP2 --> DS
    DS --> S4
    DS --> SF
    DS --> EXT

    JR --> AI
```

---

## 8.7 Joule Entitlements and Licensing

### What You Need

| Entitlement | Purpose | Where to Enable |
|-------------|---------|-----------------|
| **Joule** | Core Joule capability | BTP Cockpit → Entitlements |
| **AI Core** | LLM runtime | BTP Cockpit → Entitlements |
| **AI Launchpad** | Manage AI deployments | BTP Cockpit → Entitlements |
| **Joule Studio** | Build skills/agents | SAP Build lobby |

### AI Units Consumption

Joule consumes **AI Units** (credits):

```mermaid
pie title "Typical AI Unit Consumption per Request"
    "LLM Inference" : 60
    "Skill Execution" : 20
    "Grounding/RAG" : 15
    "Overhead" : 5
```

**Factors affecting consumption:**
- Complexity of request
- Number of skills chained
- Grounding document size
- LLM model used (GPT-4 > GPT-3.5)

### Enabling Joule in Your Subaccount

1. **Check Entitlements**
   - BTP Cockpit → Global Account → Entitlements
   - Search for "Joule" and "AI Core"
   - Add to your subaccount

2. **Run the Joule Booster**
   - BTP Cockpit → Boosters → Search "Joule"
   - Run "Set Up Joule"
   - This creates required service instances

3. **Access Joule Studio**
   - SAP Build lobby → Joule Studio
   - Or: BTP Cockpit → Instances → Joule Studio

---

## 8.8 Grounding: Teaching Joule Your Context

**Grounding** = giving the agent specific knowledge about your company.

### Without Grounding
```
User: "What's our return policy?"
Joule: "I don't have information about your specific return policy.
        Generally, return policies vary by company..."
```

### With Grounding
```
User: "What's our return policy?"
Joule: "According to ACME Corp's return policy document:
        - Returns accepted within 30 days
        - Original receipt required
        - Restocking fee of 15% for opened items
        - No returns on custom orders"
```

### Types of Grounding

```mermaid
graph TD
    subgraph "Grounding Sources"
        PDF[PDF Documents]
        KB[Knowledge Base Articles]
        SAP[SAP MDG Data]
        WEB[Web Pages]
    end

    subgraph "Vector Store"
        VS[Embeddings Database]
    end

    subgraph "Joule Agent"
        AG[Agent with Grounding]
    end

    PDF --> VS
    KB --> VS
    SAP --> VS
    WEB --> VS

    VS --> |"Semantic Search"| AG

    AG --> |"Contextual Response"| U[User]
```

### Setting Up Grounding

1. **Upload documents** to Joule Studio grounding section
2. **Process** (creates embeddings)
3. **Assign** to agent

---

## Temel Çıkarımlar

1. **Joule is SAP's AI copilot** — Lives in SAP apps, understands SAP context
2. **Skills are single actions** — One capability, one API call
3. **Agents orchestrate skills** — Chain multiple skills with reasoning
4. **Destinations are mandatory** — No hardcoded URLs/passwords
5. **Action Projects bridge** — SAP Build connects skills to destinations
6. **Grounding adds context** — Teach Joule your company specifics
7. **AI Units = cost** — Monitor consumption

---

## Sırada Ne Var?

Now that you understand the fundamentals, let's build your first Joule skill step by step—from destination to deployed skill.

---

*[Önceki: Kısım 7 – Fiori & UI5 in BTP](07-fiori-ui5-btp.md) | [Sonraki: Kısım 9 – Building Your First Joule Skill](09-first-joule-skill.md)*

*[İçindekilere Dön](../content.md)*

---

**Yazar:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Oluşturuldu ❤️ dünya genelindeki SAP öğrencileri için*
