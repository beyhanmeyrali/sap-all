# 📖 Table of Contents

## From C# & Python to SAP ABAP — The Career-Switcher's Field Guide

---

### [Preface: How to Read This Book](chapters/00-preface.md)
- Who this is for (you already code — we teach the SAP delta)
- The three-pass method: analogy → "you already know this" → the ABAP way
- How to set up a free practice system
- The job-readiness mindset

---

## Part I — Orientation

### [Chapter 1: SAP & ERP, Explained for a Developer](chapters/01-sap-erp-introduction.md)
- 1.1 What an ERP actually is (the "one shared database" idea)
- 1.2 SAP ECC vs S/4HANA vs SAP BTP — the lay of the land
- 1.3 The client concept (mandant): like multi-tenant, but baked in
- 1.4 Where ABAP runs: the NetWeaver Application Server
- 1.5 What a day on an ABAP team looks like

### [Chapter 2: The SAP Modules Without the Jargon](chapters/02-sap-modules.md)
- 2.1 MM, SD, FI, CO, PP, HCM, PM, QM — one line each
- 2.2 How modules share data (the document-flow idea)
- 2.3 Which tables belong to which module (MARA, VBAK, BKPF…)
- 2.4 Why you, the developer, must speak "module"

### [Chapter 3: Types of SAP Projects (and Where You'll Fit)](chapters/03-sap-project-types.md)
- 3.1 Implementation vs Support vs Rollout vs Migration
- 3.2 On-prem vs RISE/cloud — what changes for you
- 3.3 Roles: developer, functional, basis, architect
- 3.4 The junior developer's first 90 days

---

## Part II — ABAP Foundations

### [Chapter 4: ABAP on Eclipse (ADT) — Your New Visual Studio](chapters/04-abap-on-eclipse-adt.md)
- 4.1 ADT vs SE80: the modern vs classic IDE
- 4.2 Connecting to a system, projects, favorites
- 4.3 Packages & Transports = solution/project + source control (sort of)
- 4.4 Activation, syntax check, the ABAP debugger
- 4.5 Keyboard shortcuts a Visual Studio dev will miss

### [Chapter 5: The Data Dictionary (DDIC) — SAP's Schema Layer](chapters/05-data-dictionary-ddic.md)
- 5.1 Why DDIC exists (typing the whole company's data)
- 5.2 Domains, Data Elements, Structures, Tables (vs C# types & EF)
- 5.3 Transparent tables, keys, indexes
- 5.4 Table types, search helps, foreign keys
- 5.5 SE11 walkthrough: build your first table

### [Chapter 6: ABAP Syntax Basics for C#/Python Devs](chapters/06-abap-syntax-basics.md)
- 6.1 Variables, types, `DATA`, `TYPES`
- 6.2 Control flow: IF/CASE/LOOP/WHILE/DO
- 6.3 Structures & work areas vs DTOs/records
- 6.4 Internal tables = your List<T> / DataFrame
- 6.5 Open SQL: SELECT into tables (vs LINQ/SQL)
- 6.6 Subroutines, the call stack, and message handling

### [Chapter 7: Modern ABAP Syntax (7.4+) — The Good Stuff](chapters/07-modern-abap-syntax.md)
- 7.1 Inline declarations (`DATA(...)`), `VALUE #( )`, `CORRESPONDING`
- 7.2 Table expressions, `FILTER`, `REDUCE`, `FOR` (ABAP's LINQ)
- 7.3 String templates `|...|`
- 7.4 New `SELECT` features, `@` host variables
- 7.5 Side-by-side: rewriting old ABAP the modern way

---

## Part III — Classic Output

### [Chapter 8: Reports — Classical Lists and ALV Grids](chapters/08-reports-classical-and-alv.md)
- 8.1 The report event model (INITIALIZATION, START-OF-SELECTION…)
- 8.2 Selection screens = your input form
- 8.3 Classical WRITE lists (and why you'll still see them)
- 8.4 ALV: the grid every SAP user expects (CL_SALV_TABLE)
- 8.5 Full working ALV report you can paste and run

### [Chapter 9: Module Pool (Dialog) Programming](chapters/09-module-pool-programming.md)
- 9.1 What a dialog program is (WinForms, but in SAP GUI)
- 9.2 Screens, PBO/PAI, the flow logic
- 9.3 Screen elements, table controls, GUI status
- 9.4 When to still use it vs Fiori

### [Chapter 10: Smartforms & Adobe Forms](chapters/10-smartforms-and-adobe-forms.md)
- 10.1 Why SAP prints the way it does
- 10.2 SAPscript → Smartforms → Adobe Forms timeline
- 10.3 Driver program + form, passing data
- 10.4 Generating a PDF from ABAP

---

## Part IV — OOP & Integration

### [Chapter 11: ABAP Objects — OOP vs C#/Python](chapters/11-abap-oop.md)
- 11.1 Classes, methods, attributes, visibility
- 11.2 Interfaces, inheritance, polymorphism, abstract/final
- 11.3 Static vs instance, constructors, `ME->`
- 11.4 Exceptions: class-based `cx_root` vs try/catch
- 11.5 Events, friends, and the factory pattern
- 11.6 Clean ABAP: the style guide that gets you hired

### [Chapter 12: Function Modules, RFC & BAPI](chapters/12-function-modules-rfc-bapi.md)
- 12.1 Function modules = your callable library functions
- 12.2 Import/Export/Changing/Tables parameters
- 12.3 RFC = remote procedure call (callable from .NET/Python!)
- 12.4 BAPIs: SAP's official, stable business APIs
- 12.5 Calling SAP from C# (NCo) and Python (PyRFC)

### [Chapter 13: Hands-On — Posting via BAPI_ACC_DOCUMENT_POST (SE37)](chapters/13-handson-bapi-acc-document-post.md)
- 13.1 What "posting a document" means in finance
- 13.2 Reading the BAPI interface in SE37
- 13.3 Filling header, GL, currency tables
- 13.4 BAPI_TRANSACTION_COMMIT and error handling
- 13.5 A complete, runnable example

### [Chapter 14: Enhancing Standard SAP (the Safe Way)](chapters/14-abap-enhancements.md)
- 14.1 Why you must not modify SAP code directly
- 14.2 The enhancement spectrum: User Exits → BAdIs → Enhancement Points
- 14.3 Finding the right exit/BAdI
- 14.4 Implementing a BAdI step by step
- 14.5 Clean Core: enhancements in S/4HANA

### [Chapter 15: Data Migration — BDC & LSMW](chapters/15-data-migration-lsmw-bdc.md)
- 15.1 The migration problem (load legacy data into SAP)
- 15.2 BDC: batch input & call transaction
- 15.3 Recording with SHDB
- 15.4 LSMW: the no/low-code migration workbench
- 15.5 Modern alternative: migration cockpit

---

## Part V — Modern Data & Services

### [Chapter 16: CDS Views (Core Data Services)](chapters/16-cds-views.md)
- 16.1 Why CDS exists (push logic down to HANA)
- 16.2 Your first CDS view (vs a SQL view / EF projection)
- 16.3 Associations, annotations, parameters
- 16.4 Analytical & consumption views
- 16.5 CDS as the foundation of OData & RAP

### [Chapter 17: AMDP (ABAP Managed Database Procedures)](chapters/17-amdp.md)
- 17.1 When CDS isn't enough: writing SQLScript
- 17.2 AMDP class & method anatomy
- 17.3 Calling AMDP, debugging, performance
- 17.4 AMDP vs CDS vs Open SQL: choosing the tool

### [Chapter 18: OData & REST — The Big Picture](chapters/18-odata-and-rest-intro.md)
- 18.1 REST recap (you know this) and what OData adds
- 18.2 The OData URL conventions ($filter, $expand, $top…)
- 18.3 SAP Gateway architecture
- 18.4 Two ways to build OData: SEGW (classic) vs RAP (modern)
- 18.5 Metadata document & the service catalog

### [Chapter 19: RFC Web Services — The Consumer Proxy (SPROXY)](chapters/19-consumer-proxy-sproxy.md)
- 19.1 Consuming an external SOAP/REST service from ABAP
- 19.2 SPROXY: generating a proxy from WSDL (like `Add Service Reference`)
- 19.3 Logical ports (SOAMANAGER)
- 19.4 Calling the proxy & handling faults

### [Chapter 20: ALE & IDocs — SAP's Classic Messaging](chapters/20-ale-idocs.md)
- 20.1 IDoc = a strongly-typed message/envelope
- 20.2 Basic types, segments, partner profiles
- 20.3 Inbound vs outbound flow
- 20.4 Monitoring (WE02/WE19) and reprocessing

---

## Part VI — OData Deep Dive

### [Chapter 21: Entity Types, Entity Sets, XML & JSON](chapters/21-odata-entity-types-xml-json.md)
- 21.1 Entity Type vs Entity Set (class vs collection)
- 21.2 Properties, keys, and the data model in SEGW
- 21.3 The metadata XML ($metadata) explained
- 21.4 XML vs JSON responses ($format)

### [Chapter 22: Service Methods & Import Parameters](chapters/22-odata-methods-and-parameters.md)
- 22.1 The DPC/DPC_EXT class & redefinable methods
- 22.2 Mapping URL operations to methods
- 22.3 io_tech_request_context: reading keys, filters, params
- 22.4 Returning data & status

### [Chapter 23: Reading Data — GET_ENTITY & GET_ENTITYSET](chapters/23-odata-read-crud-get-entity.md)
- 23.1 GET_ENTITYSET (the collection / list)
- 23.2 GET_ENTITY (single record by key)
- 23.3 Reading keys from the request
- 23.4 Full read-only service, end to end

### [Chapter 24: Search Strings & Query Options](chapters/24-odata-search-and-query-options.md)
- 24.1 $filter → WHERE clause mapping
- 24.2 Free-text search (search string)
- 24.3 $top, $skip, $orderby, $inlinecount (paging)
- 24.4 Implementing them in GET_ENTITYSET

### [Chapter 25: Create, Update & Delete](chapters/25-odata-create-update-delete.md)
- 25.1 CREATE_ENTITY (POST)
- 25.2 UPDATE_ENTITY (PUT/PATCH/MERGE)
- 25.3 DELETE_ENTITY (DELETE)
- 25.4 Validation, messages, and rollback

### [Chapter 26: Associations — Principal & Dependent Entity Sets](chapters/26-odata-associations.md)
- 26.1 Associations & navigation properties
- 26.2 Defining them in SEGW
- 26.3 GET_ENTITYSET for the dependent set via navigation
- 26.4 Example: Customer → Orders

### [Chapter 27: Header + Item Data in One Service](chapters/27-odata-header-and-item.md)
- 27.1 The classic header/item model (e.g., order + line items)
- 27.2 Two entity sets + association
- 27.3 Reading header with its items
- 27.4 Real example walkthrough

### [Chapter 28: Function Imports](chapters/28-odata-function-import.md)
- 28.1 When CRUD isn't enough (actions/operations)
- 28.2 Defining a function import in SEGW
- 28.3 Redefining EXECUTE_ACTION
- 28.4 Example: "approve order" function import

### [Chapter 29: CREATE_DEEP_ENTITY](chapters/29-odata-create-deep-entity.md)
- 29.1 Posting header + items in a single request
- 29.2 The deep structure & io_data_provider
- 29.3 Implementation & testing
- 29.4 Transactional save with the BAPI underneath

### [Chapter 30: GET_EXPANDED_ENTITYSET](chapters/30-odata-get-expanded-entityset.md)
- 30.1 $expand and why you override the expand method
- 30.2 Building the expanded response structure
- 30.3 Implementation & testing
- 30.4 Performance notes

### [Chapter 31: Upload & Download Files in OData](chapters/31-odata-file-upload-download.md)
- 31.1 Media entities & streams
- 31.2 GET_STREAM (download) & CREATE_STREAM (upload)
- 31.3 MIME types & content disposition
- 31.4 Example: attach and retrieve a document

---

## Part VII — Real-World Integrations

### [Chapter 32: Google Form → SAP Integration](chapters/32-google-form-integration.md)
- 32.1 The scenario (form submission lands in SAP)
- 32.2 Apps Script webhook → SAP OData/REST
- 32.3 Authentication & the inbound service
- 32.4 End-to-end build (a great portfolio piece)

### [Chapter 33: WhatsApp Integration from ABAP](chapters/33-whatsapp-integration.md)
- 33.1 The scenario (send/receive WhatsApp messages)
- 33.2 Cloud API basics & webhooks
- 33.3 Sending messages with CL_HTTP_CLIENT / RFC
- 33.4 Receiving messages into SAP

---

## Part VIII — Modern UI & RAP

### [Chapter 34: Fiori & UI5 for ABAP Developers](chapters/34-fiori.md)
- 34.1 What Fiori is (SAP's React/Angular-era UI)
- 34.2 UI5 vs the frameworks you know
- 34.3 Fiori Elements (metadata-driven UIs) vs freestyle
- 34.4 How a Fiori app talks to your OData service
- 34.5 The Launchpad & deployment

### [Chapter 35: RAP — RESTful Application Programming (The Finale)](chapters/35-rap-restful-application-programming.md)
- 35.1 What RAP is and why it replaces SEGW
- 35.2 The stack: DB table → CDS → Behavior Definition → Service
- 35.3 Managed vs unmanaged scenarios
- 35.4 Building a full RAP CRUD app step by step
- 35.5 Exposing it as OData V4 + a Fiori Elements app
- 35.6 Where to go next (your roadmap to senior)

---

## Appendices

### [Appendix A: C# / Python ↔ ABAP Cheat Sheet](chapters/appendix-a-csharp-abap-cheatsheet.md)
### [Appendix B: Glossary for Newcomers](chapters/appendix-b-glossary.md)
### [Appendix C: Transaction Code Reference](chapters/appendix-c-transaction-codes.md)
### [Appendix D: Resources, Practice Systems & Job Prep](chapters/appendix-d-resources.md)

---

*[Back to README](README.md)*
