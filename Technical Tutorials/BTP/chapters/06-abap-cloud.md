# Chapter 6: ABAP in the Cloud – The ABAP Environment

> *Your Skills Transfer—With Some New Rules*

---

This chapter is for every ABAP developer wondering: "Can I still write ABAP in the cloud?" The answer is yes—but with important differences. Let's explore the BTP ABAP Environment and what it means for your career.

---

## 6.1 BTP ABAP Environment Overview

The **BTP ABAP Environment** (sometimes called "Steampunk" in SAP internal lingo) is a cloud-based ABAP runtime where you can write modern ABAP code.

### What It Is

- A **fully managed ABAP system** running in the cloud
- Designed for **cloud-native development**
- Uses the **ABAP RESTful Application Programming Model (RAP)**
- Exposes services via **OData** for Fiori apps
- Runs on **SAP HANA Cloud** database

### What It Is NOT

- ❌ **NOT** a copy of your on-prem system
- ❌ **NOT** a place to run SE38 reports with ALV
- ❌ **NOT** compatible with all classic ABAP code
- ❌ **NOT** a migration target for all Z-code

> **Think of it as**: A fresh, modern ABAP environment built for cloud apps—not a dumping ground for legacy code.

### When to Use It

| Use Case | BTP ABAP Environment |
|----------|---------------------|
| New extension for RISE S/4 | ✅ Perfect |
| Greenfield cloud project | ✅ Ideal |
| Custom OData services | ✅ Yes |
| Side-by-side extensions | ✅ Designed for this |
| Running legacy reports | ❌ Not suitable |
| Heavy modifications | ❌ Use on-prem |

---

## 6.2 How It Differs from Classic On-Prem ABAP

This is the critical section. Let's compare side by side:

### Development Environment

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| **IDE** | SE80 (SAP GUI) or Eclipse | Eclipse ADT **only** |
| **Access** | SAP GUI transactions | Eclipse + ABAP Cloud Project |
| **Code editor** | SE38/SE24/SE80 | Eclipse ADT |
| **Debugging** | Transaction ABAP Debugger | Eclipse ADT Debugger |

**Key point**: No SAP GUI access. Everything through Eclipse ADT.

### Available Objects and APIs

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| **Available objects** | Everything | **Allowlisted objects only** |
| **Function modules** | All | Only released ones |
| **BAPIs** | All | Only released ones |
| **Classes** | All | Only released ones |
| **Database tables** | Direct access | Via CDS views |

### The "Released APIs" Concept

In BTP ABAP, you can only use **released APIs**—objects that SAP guarantees will remain stable:

```abap
" This works - CL_SALV_TABLE is released
DATA(lo_alv) = cl_salv_table=>factory( ... ).  " ❌ WAIT - ALV not available in cloud!

" Actually, in BTP ABAP you'd do this instead:
" Build a RAP service and consume via Fiori Elements
```

**How to check if an API is released:**
1. In Eclipse ADT, open the class/function module
2. Look for the "Release Contract" property
3. Or use the **ABAP Test Cockpit** (ATC) to check compatibility

### Database Access

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| **SQL** | Open SQL + Native SQL | Open SQL **only** |
| **Direct table access** | SELECT from any table | Only via CDS views |
| **ADBC** | Available | Limited |
| **Native hints** | Allowed | Not allowed |

```abap
" Classic ABAP - works on-prem
SELECT * FROM mara INTO TABLE @DATA(lt_materials).

" BTP ABAP - use CDS views instead
SELECT * FROM I_Product INTO TABLE @DATA(lt_products).
```

### Modifications and Extensions

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| **Modifications** | Allowed (risky) | **Not allowed** |
| **User exits** | Available | Not available |
| **BADIs** | All | Only released ones |
| **Enhancement points** | Available | Not available |

### Transport System

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| **Transport system** | CTS/CTS+ | **gCTS** (Git-based) |
| **Transport requests** | SE09/SE10 | Git commits |
| **Import/Export** | STMS | Git push/pull |
| **Version control** | Limited | Full Git history |

### UI Development

| Aspect | Classic On-Prem | BTP ABAP Environment |
|--------|-----------------|----------------------|
| **Dynpro** | Available | ❌ Not available |
| **Selection screens** | Available | ❌ Not available |
| **ALV** | Available | ❌ Not available |
| **WebDynpro ABAP** | Available | ❌ Not available |
| **UI** | **RAP + Fiori Elements** | ✅ Primary method |

---

## 6.3 RAP (RESTful ABAP Programming) Basics

**RAP** is the new programming model for ABAP in the cloud. If you learn one thing from this chapter, learn RAP.

### The RAP Stack

```
┌─────────────────────────────────────┐
│          Fiori Elements UI          │  ← Auto-generated UI
├─────────────────────────────────────┤
│           OData Service             │  ← Exposed automatically
├─────────────────────────────────────┤
│        Service Definition           │  ← What to expose
├─────────────────────────────────────┤
│       Behavior Definition           │  ← Business logic
├─────────────────────────────────────┤
│      Behavior Implementation        │  ← ABAP code
├─────────────────────────────────────┤
│    CDS View Entity (Interface)      │  ← Data model
├─────────────────────────────────────┤
│    CDS View Entity (Consumption)    │  ← UI annotations
├─────────────────────────────────────┤
│         Database Tables             │  ← Persistence
└─────────────────────────────────────┘
```

### Step 1: Database Table

```abap
@EndUserText.label : 'Sales Order'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define table zsalesorder {
  key client         : abap.clnt not null;
  key order_id       : abap.char(10) not null;
      customer       : abap.char(10);
      order_date     : abap.dats;
      total_amount   : abap.dec(15,2);
      currency       : abap.cuky;
      status         : abap.char(1);
      created_by     : abap.uname;
      created_at     : timestampl;
      last_changed_by: abap.uname;
      last_changed_at: timestampl;
}
```

### Step 2: CDS View Entity (Interface View)

```sql
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Sales Order Interface'
define root view entity ZI_SalesOrder
  as select from zsalesorder
{
  key order_id       as OrderId,
      customer       as Customer,
      order_date     as OrderDate,
      total_amount   as TotalAmount,
      currency       as Currency,
      status         as Status,

      @Semantics.user.createdBy: true
      created_by     as CreatedBy,
      @Semantics.systemDateTime.createdAt: true
      created_at     as CreatedAt,
      @Semantics.user.lastChangedBy: true
      last_changed_by as LastChangedBy,
      @Semantics.systemDateTime.lastChangedAt: true
      last_changed_at as LastChangedAt
}
```

### Step 3: CDS View Entity (Consumption View with UI Annotations)

```sql
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Sales Order'
@Metadata.allowExtensions: true

@UI: {
  headerInfo: {
    typeName: 'Sales Order',
    typeNamePlural: 'Sales Orders',
    title: { type: #STANDARD, value: 'OrderId' }
  }
}

define root view entity ZC_SalesOrder
  provider contract transactional_query
  as projection on ZI_SalesOrder
{
      @UI.facet: [{ id: 'General',
                    purpose: #STANDARD,
                    type: #IDENTIFICATION_REFERENCE,
                    label: 'General Information',
                    position: 10 }]

      @UI: { lineItem: [{ position: 10 }],
             identification: [{ position: 10 }] }
  key OrderId,

      @UI: { lineItem: [{ position: 20 }],
             identification: [{ position: 20 }] }
      Customer,

      @UI: { lineItem: [{ position: 30 }],
             identification: [{ position: 30 }] }
      OrderDate,

      @UI: { lineItem: [{ position: 40 }],
             identification: [{ position: 40 }] }
      @Semantics.amount.currencyCode: 'Currency'
      TotalAmount,

      Currency,

      @UI: { lineItem: [{ position: 50, criticality: 'StatusCriticality' }],
             identification: [{ position: 50 }] }
      Status,

      /* Administrative fields */
      CreatedBy,
      CreatedAt,
      LastChangedBy,
      LastChangedAt
}
```

### Step 4: Behavior Definition

```abap
managed implementation in class zbp_i_salesorder unique;
strict ( 2 );

define behavior for ZI_SalesOrder alias SalesOrder
persistent table zsalesorder
lock master
authorization master ( instance )
etag master LastChangedAt
{
  // Standard operations
  create;
  update;
  delete;

  // Field controls
  field ( readonly ) OrderId, CreatedBy, CreatedAt, LastChangedBy, LastChangedAt;
  field ( mandatory ) Customer;

  // Validations
  validation validateCustomer on save { create; update; field Customer; }
  validation validateAmount on save { create; update; field TotalAmount; }

  // Determinations
  determination setInitialStatus on modify { create; }

  // Actions
  action ( features : instance ) confirmOrder result [1] $self;
  action ( features : instance ) cancelOrder result [1] $self;

  // Mappings
  mapping for zsalesorder
  {
    OrderId = order_id;
    Customer = customer;
    OrderDate = order_date;
    TotalAmount = total_amount;
    Currency = currency;
    Status = status;
    CreatedBy = created_by;
    CreatedAt = created_at;
    LastChangedBy = last_changed_by;
    LastChangedAt = last_changed_at;
  }
}
```

### Step 5: Behavior Implementation

```abap
CLASS zbp_i_salesorder DEFINITION PUBLIC ABSTRACT FINAL
  FOR BEHAVIOR OF zi_salesorder.
ENDCLASS.

CLASS zbp_i_salesorder IMPLEMENTATION.
ENDCLASS.

" Local handler class
CLASS lhc_salesorder DEFINITION INHERITING FROM cl_abap_behavior_handler.
  PRIVATE SECTION.
    METHODS:
      get_instance_authorizations FOR INSTANCE AUTHORIZATION
        IMPORTING keys REQUEST requested_authorizations
        FOR salesorder RESULT result,

      validateCustomer FOR VALIDATE ON SAVE
        IMPORTING keys FOR salesorder~validateCustomer,

      validateAmount FOR VALIDATE ON SAVE
        IMPORTING keys FOR salesorder~validateAmount,

      setInitialStatus FOR DETERMINE ON MODIFY
        IMPORTING keys FOR salesorder~setInitialStatus,

      confirmOrder FOR MODIFY
        IMPORTING keys FOR ACTION salesorder~confirmOrder RESULT result,

      cancelOrder FOR MODIFY
        IMPORTING keys FOR ACTION salesorder~cancelOrder RESULT result.
ENDCLASS.

CLASS lhc_salesorder IMPLEMENTATION.
  METHOD get_instance_authorizations.
    " Authorization logic here
  ENDMETHOD.

  METHOD validateCustomer.
    " Read the data
    READ ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        FIELDS ( Customer )
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_orders).

    LOOP AT lt_orders INTO DATA(ls_order).
      IF ls_order-Customer IS INITIAL.
        APPEND VALUE #(
          %tky = ls_order-%tky
        ) TO failed-salesorder.

        APPEND VALUE #(
          %tky = ls_order-%tky
          %msg = new_message_with_text(
            severity = if_abap_behv_message=>severity-error
            text = 'Customer is required' )
          %element-Customer = if_abap_behv=>mk-on
        ) TO reported-salesorder.
      ENDIF.
    ENDLOOP.
  ENDMETHOD.

  METHOD validateAmount.
    READ ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        FIELDS ( TotalAmount )
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_orders).

    LOOP AT lt_orders INTO DATA(ls_order).
      IF ls_order-TotalAmount < 0.
        APPEND VALUE #( %tky = ls_order-%tky ) TO failed-salesorder.
        APPEND VALUE #(
          %tky = ls_order-%tky
          %msg = new_message_with_text(
            severity = if_abap_behv_message=>severity-error
            text = 'Amount cannot be negative' )
          %element-TotalAmount = if_abap_behv=>mk-on
        ) TO reported-salesorder.
      ENDIF.
    ENDLOOP.
  ENDMETHOD.

  METHOD setInitialStatus.
    READ ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        FIELDS ( Status )
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_orders).

    MODIFY ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        UPDATE FIELDS ( Status )
        WITH VALUE #( FOR order IN lt_orders
          ( %tky = order-%tky
            Status = 'N' ) )  " N = New
      REPORTED DATA(lt_reported).
  ENDMETHOD.

  METHOD confirmOrder.
    MODIFY ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        UPDATE FIELDS ( Status )
        WITH VALUE #( FOR key IN keys
          ( %tky = key-%tky
            Status = 'C' ) )  " C = Confirmed
      FAILED failed
      REPORTED reported.

    READ ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        ALL FIELDS
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_orders).

    result = VALUE #( FOR order IN lt_orders
      ( %tky = order-%tky
        %param = order ) ).
  ENDMETHOD.

  METHOD cancelOrder.
    MODIFY ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        UPDATE FIELDS ( Status )
        WITH VALUE #( FOR key IN keys
          ( %tky = key-%tky
            Status = 'X' ) )  " X = Cancelled
      FAILED failed
      REPORTED reported.

    READ ENTITIES OF zi_salesorder IN LOCAL MODE
      ENTITY salesorder
        ALL FIELDS
        WITH CORRESPONDING #( keys )
      RESULT DATA(lt_orders).

    result = VALUE #( FOR order IN lt_orders
      ( %tky = order-%tky
        %param = order ) ).
  ENDMETHOD.
ENDCLASS.
```

### Step 6: Service Definition

```sql
@EndUserText.label: 'Sales Order Service Definition'
define service ZUI_SalesOrder {
  expose ZC_SalesOrder as SalesOrder;
}
```

### Step 7: Service Binding

In Eclipse ADT:
1. Right-click Service Definition → New Service Binding
2. Choose binding type: OData V4 - UI
3. Activate and publish

Now you have a complete RAP application with:
- ✅ Database table
- ✅ Data model (CDS views)
- ✅ Business logic (validations, determinations, actions)
- ✅ OData service
- ✅ Ready for Fiori Elements UI

---

## 6.4 When to Use BTP ABAP vs. Classic ABAP

This decision matrix helps you choose:

### Use BTP ABAP Environment When:

| Scenario | Why BTP ABAP |
|----------|--------------|
| **New extension for RISE S/4** | Clean Core requires side-by-side |
| **Greenfield cloud project** | Modern, cloud-native from start |
| **Building OData services** | RAP is designed for this |
| **Need Git version control** | gCTS provides full Git integration |
| **Multi-tenant SaaS app** | Built-in multitenancy support |
| **Joule skill backend** | Perfect for AI-powered extensions |

### Keep Classic On-Prem ABAP When:

| Scenario | Why On-Prem |
|----------|-------------|
| **Running existing reports** | ALV, selection screens not in cloud |
| **Heavy batch processing** | Better performance on-prem |
| **Accessing unreleased APIs** | Only released APIs in cloud |
| **Dynpro-based applications** | Dynpro not available |
| **Extensive modifications** | Modifications not allowed in cloud |
| **Legacy integration (RFC heavy)** | RFC less common in cloud |

### The Hybrid Approach

Many projects use both:

```
┌────────────────────────────────────────────────────────────┐
│                     User Interface                          │
│                    (Fiori Elements)                         │
└─────────────────────────┬──────────────────────────────────┘
                          │ OData
┌─────────────────────────┼──────────────────────────────────┐
│                    BTP ABAP Environment                     │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │ RAP Extension   │  │ Custom Logic    │                  │
│  │ New Features    │  │ Orchestration   │                  │
│  └────────┬────────┘  └────────┬────────┘                  │
│           │                    │                            │
└───────────┼────────────────────┼────────────────────────────┘
            │ OData/RFC          │
┌───────────┼────────────────────┼────────────────────────────┐
│           ▼                    ▼                            │
│  ┌─────────────────────────────────────────────┐           │
│  │              S/4HANA (RISE)                  │           │
│  │  - Standard APIs                             │           │
│  │  - Master Data                               │           │
│  │  - Core Transactions                         │           │
│  └─────────────────────────────────────────────┘           │
│                  On-Prem or RISE                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 6.5 Migration Path: Moving Code from On-Prem

If you have existing ABAP code and want to move it to BTP ABAP Environment:

### Step 1: Analyze Your Code

Use the **Custom Code Migration** tools:

1. **ATC (ABAP Test Cockpit)** with cloud checks
   - Run ATC with "ABAP Cloud" check variant
   - Identifies incompatible APIs

2. **Custom Code Migration App** in Fiori
   - Visual analysis of your Z-code
   - Shows compatibility percentage
   - Suggests alternatives

### Step 2: Identify What Must Change

| Issue | Solution |
|-------|----------|
| Using unreleased function module | Find released alternative or wrap in RFC |
| Direct table access | Create CDS view |
| ALV/selection screen | Rewrite as RAP + Fiori |
| Dynpro | Complete UI redesign |
| BADI (unreleased) | Use released extension points |
| Native SQL | Convert to Open SQL |

### Step 3: Common API Replacements

| Classic API | Cloud Alternative |
|-------------|-------------------|
| `BAPI_SALESORDER_CREATEFROMDAT2` | Released API: `I_SalesOrderTP` |
| `CONVERSION_EXIT_*` | CDS conversion functions |
| `READ TABLE ... BINARY SEARCH` | `READ TABLE ... WITH KEY` (optimizer handles it) |
| `SUBMIT ... AND RETURN` | Background jobs via APIs |
| `CALL TRANSACTION` | APIs, not transactions |

### Step 4: Refactor to RAP

Instead of porting 1:1, redesign using RAP:

**Old approach (Classic):**
```abap
" Classic report
REPORT z_sales_report.
SELECT-OPTIONS: s_kunnr FOR kna1-kunnr.
PARAMETERS: p_date TYPE sy-datum.

START-OF-SELECTION.
  SELECT * FROM vbak WHERE kunnr IN s_kunnr
                       AND erdat = p_date
    INTO TABLE @DATA(lt_orders).

  cl_salv_table=>factory( IMPORTING r_salv_table = DATA(lo_alv)
                          CHANGING  t_table = lt_orders ).
  lo_alv->display( ).
```

**New approach (RAP):**
```sql
" CDS View with parameters
@EndUserText.label: 'Sales Orders Report'
define view entity ZC_SalesOrderReport
  with parameters p_date : abap.dats
  as select from I_SalesOrder
{
  key SalesOrder,
      SoldToParty,
      SalesOrderDate,
      TotalNetAmount
}
where SalesOrderDate = $parameters.p_date
```

Then consume via Fiori Elements List Report—no ABAP code for UI needed!

### Step 5: Test Thoroughly

1. **Unit tests** with ABAP Unit
2. **Integration tests** with RAP test doubles
3. **End-to-end tests** with Fiori app

---

## 6.6 Setting Up Your BTP ABAP Development Environment

### Prerequisites

1. **Eclipse IDE** (latest version)
2. **ABAP Development Tools (ADT)** plugin
3. **BTP subaccount** with ABAP Environment entitlement
4. **Service key** for the ABAP system

### Step-by-Step Setup

1. **Create ABAP Environment Instance**
   - BTP Cockpit → Subaccount → Service Marketplace
   - Find "ABAP Environment" → Create Instance
   - Choose plan (e.g., `free` for trial, `standard` for prod)

2. **Create Service Key**
   - Instance → Create Service Key
   - Download the JSON (contains URLs and credentials)

3. **Connect Eclipse to ABAP Environment**
   - Eclipse → ABAP Perspective
   - File → New → ABAP Cloud Project
   - Choose "Service Key"
   - Paste service key JSON
   - Authenticate via browser (OAuth)

4. **Create Your First Package**
   - Right-click on system → New → ABAP Package
   - Package name: `ZTEST_MYPACKAGE`
   - Add to transport (creates Git commit)

### Project Structure Best Practices

```
ZTEST_MYPROJECT
├── src
│   ├── zi_salesorder.ddls.asddls        " Interface view
│   ├── zc_salesorder.ddls.asddls        " Consumption view
│   ├── zi_salesorder.bdef.asbdef        " Behavior definition
│   ├── zbp_i_salesorder.clas.abap       " Behavior implementation
│   ├── zui_salesorder.srvd.srvdsrv      " Service definition
│   └── zui_salesorder_o4.srvb.srvb      " Service binding
└── test
    └── ztc_salesorder.clas.abap          " Test class
```

---

## 6.7 gCTS: Git-based Change and Transport System

One of the biggest improvements in BTP ABAP is **gCTS**—Git-based transport.

### How It Works

```
┌────────────────────────────────────────────────────────────┐
│                     Developer Workflow                      │
│                                                             │
│  1. Make changes in Eclipse ADT                            │
│  2. Release transport request                               │
│  3. gCTS commits to Git repository                         │
│  4. Pull request / code review (optional)                  │
│  5. Import to target system via Git                        │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

### Benefits Over Classic CTS

| Classic CTS | gCTS |
|-------------|------|
| Binary transport files | Human-readable Git commits |
| Limited history | Full Git history |
| No branching | Full branching support |
| Manual conflict resolution | Git merge tools |
| No code review built-in | Pull request workflows |

### Typical Git Workflow

```bash
# Your ABAP changes become Git commits
main
├── feature/new-order-validation    (developer branch)
├── feature/customer-lookup         (another feature)
└── release/v1.0                    (release branch)
```

---

## Key Takeaways

1. **BTP ABAP is modern ABAP** — Cloud-native, RAP-based, Git-powered
2. **Not everything works** — Only released APIs, no Dynpro, no ALV
3. **RAP is the future** — Learn CDS views + behavior definitions
4. **Eclipse ADT required** — No SAP GUI access
5. **gCTS for transport** — Git-based, enables modern DevOps
6. **Side-by-side model** — BTP ABAP extends RISE S/4, doesn't replace it
7. **Analyze before migrating** — Use Custom Code Migration tools

---

## What's Next?

You now understand ABAP in the cloud. Next, let's look at the other side of development—Fiori and UI5 in BTP—and how to deploy those apps.

---

*[Previous: Chapter 5 – Destinations](05-destinations.md) | [Next: Chapter 7 – Fiori & UI5 in BTP](07-fiori-ui5-btp.md)*

*[Back to Table of Contents](../content.md)*

---

**Author:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Created with ❤️ for SAP learners worldwide*
