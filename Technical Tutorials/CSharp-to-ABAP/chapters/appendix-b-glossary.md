# Appendix B: Glossary for Newcomers

*Every SAP term that stopped you mid-sentence — defined in plain English, with a C#/Python parallel where one exists.*

---

## 📌 How to use this glossary

Alphabetical order. If you hit an unfamiliar term in a standup or ticket, look it up here first. Each entry tells you what the thing *is*, what you'd call it in C#/Python, and — where useful — the transaction code that opens it.

---

## A

**ABAP**
The programming language that runs inside SAP systems. Stands for *Advanced Business Application Programming*. Created by SAP in the 1980s, it lives entirely inside the SAP runtime (no standalone compiler). Think of it as "the only language spoken by the SAP kernel" — like writing .NET code that can only run on the .NET CLR, except the CLR is also your database server, web server, and job scheduler combined. Current release (ABAP 7.5+) supports full OOP, lambdas, template strings, and more.

**ABAP Dictionary** — see *DDIC*.

**ABAP Unit**
The built-in unit testing framework. Like `xUnit` in C# or `pytest` in Python, but baked into the ABAP Workbench and ADT. Test classes are tagged `FOR TESTING`. Run from ADT with Ctrl+Shift+F10.

**ADT (ABAP Development Tools)**
The Eclipse-based IDE for ABAP. This is your Visual Studio / VS Code equivalent in the SAP world. You write and debug ABAP here instead of in SAPGUI's SE80. ADT connects to a live SAP system over HTTP — there is no local compilation. See Chapter 4.

**ALE (Application Link Enabling)**
SAP's framework for distributing data between SAP systems using IDocs. Think of it as an internal pub/sub or ESB, but older and XML-free (IDocs use their own flat-file format). Configuration lives in transaction `BD64`.

**ALV (ABAP List Viewer)**
The standard grid/report display component in SAP. Like a DataGrid in WinForms or a `<DataGrid>` in WPF — except it's the *only* grid you use in classic ABAP reports, and it comes with built-in sort, filter, export, and layout-save features for free. Main class: `CL_SALV_TABLE` (OOP ALV) or function module `REUSE_ALV_GRID_DISPLAY` (classic). See Chapter 8.

**AMDP (ABAP Managed Database Procedures)**
A way to write SAP HANA stored procedures using ABAP class methods. Like writing a SQL Server stored procedure but declaring it inside a C# class with a special attribute. Useful for pushing complex calculations down to the database layer. See Chapter 17.

---

## B

**BAdI (Business Add-In)**
A SAP enhancement framework that lets you add custom logic to standard SAP code without modifying it — like implementing an interface that SAP calls at defined hook points. The closest C# analogy is a plugin pattern: SAP defines the interface, you provide the implementation, the framework calls it. Managed in `SE18` (definition) and `SE19` (implementation). See Chapter 14.

**BAPI (Business Application Programming Interface)**
A stable, documented RFC-enabled function module that represents a SAP business object method. Think of it as a published REST endpoint, but for function calls: `BAPI_CUSTOMER_CREATEFROMDATA1`, `BAPI_ACC_DOCUMENT_POST`, etc. BAPIs are the safe, supported way to post data into SAP from outside. Always call `BAPI_TRANSACTION_COMMIT` after a successful BAPI call, or your data won't be saved. See Chapter 12.

**BDC (Batch Data Communication)**
A technique for automating SAP GUI screen interactions in bulk — like a recorded macro or Selenium script that drives SAPGUI screens to enter data. Used for data migration when no BAPI exists. See Chapter 15.

**BTP (SAP Business Technology Platform)**
SAP's cloud platform. Like Azure or AWS but SAP-flavoured: hosts ABAP environments, integration middleware (Integration Suite), low-code tools, and more. The *SAP BTP ABAP Environment* is a cloud-based ABAP system where modern ABAP (RAP, CDS, Tier 1 APIs) runs.

---

## C

**CDS (Core Data Services)**
A layer for defining data models and views using an SQL-like annotation-driven syntax, which gets "pushed down" to the database. Think of it as a very powerful SQL view with built-in OData, authorisation, and UI annotation support. In C# terms: imagine if your LINQ expression compiled to a database view that also auto-generated a REST endpoint. The modern backbone of S/4HANA development. See Chapter 16.

**Client / Mandant**
A logical tenant inside a single SAP system. One physical SAP installation can host multiple clients (e.g., client 100 = Development, 200 = QA, 300 = Production). Every client-dependent table has a `MANDT` key field. ABAP's Open SQL automatically filters by the current client — you don't write `WHERE mandt = sy-mandt`, it's injected for you. This surprises every newcomer who looks at the database and wonders why `MANDT` is everywhere.

**CMOD**
Transaction for activating and managing enhancement projects (collections of SAP user exits / BAdiS). Usually paired with `SMOD`. See *Enhancements*.

**Commit Work**
`COMMIT WORK` — the explicit database commit statement in ABAP. Unlike EF Core's `SaveChanges()` or SQLAlchemy's `session.commit()`, nothing goes to the database permanently until you call this. Missing it is one of the most common bugs in ABAP data-migration code. Also: `ROLLBACK WORK` to undo.

---

## D

**Data Element**
A named, reusable ABAP type that carries semantic meaning: a field label, search help, documentation, and a domain. Like a named type alias in C# (`typedef`) plus metadata. When you declare a DDIC table column as `zfield TYPE ze_customer_id`, you're referencing a Data Element. Maintained in `SE11`.

**DDIC (Data Dictionary)**
SAP's centralised schema layer — the place where all table definitions, structure types, data elements, domains, and type groups live. Like a combination of Entity Framework's model definitions and SQL Server's information schema, but stored inside the SAP system itself. Transaction: `SE11`. See Chapter 5.

**DPC / MPC (Data Provider Class / Model Provider Class)**
The two generated ABAP classes that back a classic OData service built with SEGW. MPC defines the entity model; DPC implements the CRUD logic. You extend `*_EXT` variants to write your actual code. See Chapters 21–31.

**Domain**
A DDIC object that defines the value range and data type for a Data Element. Think of it as an enum definition or a constrained scalar type: `ZDOMAIN_STATUS` might allow only `'A'`, `'I'`, `'D'`. Maintained in `SE11`.

---

## E

**Enhancement**
A way to extend standard SAP code without modifying the original source (so SAP Support Packages don't overwrite your changes). Types: User Exits (old), BAdiS (modern), Enhancement Spots, Implicit/Explicit Enhancements. See Chapter 14.

---

## F

**Fiori**
SAP's modern UI design system and app framework, built on top of UI5. Like the SAP equivalent of Material Design + a component library. Fiori apps are web applications (HTML5/JavaScript) that consume OData services. See Chapter 34.

**Function Module**
A reusable, callable unit of ABAP code with a formal interface (Import, Export, Changing, Tables, Exceptions parameters). Like a static method in a utility class, but with a formal contract enforced by the system. Some Function Modules are RFC-enabled and can be called from outside SAP. Maintained in `SE37`. See Chapter 12.

---

## G

**Gateway (SAP Gateway)**
The SAP component that hosts and exposes OData services. Think of it as the API gateway / IIS hosting layer for your OData endpoints. Transaction `/IWFND/MAINT_SERVICE` registers services; `/IWFND/GW_CLIENT` lets you test them like Postman. See Chapter 18.

---

## H

**HANA (SAP HANA)**
SAP's in-memory relational database — the database that modern SAP systems run on. Columnar storage, in-memory processing, extremely fast for aggregations. You write **Open SQL** against it (not native HANA SQL) unless you're writing an AMDP. The fact that it's in-memory is mostly invisible to you as an ABAP developer.

---

## I

**IDoc (Intermediate Document)**
SAP's proprietary message format for system-to-system data exchange. An IDoc is a flat, structured document with a control record, data records, and status records — like an EDI message or a strongly-typed XML document but in SAP's own format. Used heavily in ALE integration and with external EDI partners. See Chapter 20. Transactions: `WE02`, `WE05`, `WE19`.

**Internal Table**
An in-memory collection of rows that all share the same structure (type). The most important data structure in ABAP. Like `List<T>` in C# or a `DataFrame` in pandas, but with three flavours: `STANDARD` (array-like), `SORTED` (auto-sorted binary search), `HASHED` (hash map, O(1) key lookup). See Appendix A and Chapter 6.

---

## L

**LSMW (Legacy System Migration Workbench)**
A point-and-click data migration tool that can record BDC sessions, call BAPIs, or use IDocs to load data from flat files. Like a built-in ETL tool for SAP. Transaction: `LSMW`. See Chapter 15.

**LUW (Logical Unit of Work)**
The boundary within which database changes are either all committed or all rolled back — equivalent to a database transaction. In ABAP dialog programs, the LUW is controlled with `COMMIT WORK` / `ROLLBACK WORK`. Important: RFC calls to remote systems each have their own LUW, which is why BAPI commits need special handling.

---

## M

**Module Pool**
A type of ABAP program that implements screen-based dialog (form) applications — like a WinForms or WPF app, but running inside SAPGUI. Built from screens designed in Screen Painter, connected by flow logic (`PROCESS BEFORE OUTPUT` / `PROCESS AFTER INPUT`). See Chapter 9.

**MANDT** — see *Client / Mandant*.

---

## N

**NetWeaver (SAP NetWeaver)**
The application server platform that runs ABAP (and Java) programs. The runtime layer underneath everything: it manages work processes, memory, database connections, and the ICM (Internet Communication Manager) for HTTP. Think of it as IIS + CLR + SQL Server connection pool, all in one product. Modern S/4HANA still runs on NetWeaver.

---

## O

**OData**
An open protocol (built on HTTP + REST + Atom/JSON) for building data APIs, heavily used in SAP. Like REST with a schema: entity types, entity sets, navigation properties, and standard query options (`$filter`, `$expand`, `$top`, etc.) are part of the protocol. SAP Gateway serves OData. See Chapters 18–31.

**Open SQL**
The ABAP dialect of SQL — a portable SQL subset that the ABAP runtime translates to the native database dialect (HANA SQL, Oracle, etc.). You always write Open SQL in ABAP reports and classes; HANA-specific SQL only appears in AMDPs. Key difference from C# LINQ: Open SQL is a string-based DSL embedded in the ABAP source, not a type-safe query builder.

---

## P

**Package**
An organisational container in ABAP — like a C# namespace or a Java package, but also a transport unit. Every ABAP object (class, table, program) belongs to a package. Packages control which objects travel together on a Transport Request. Object: `DEVC`. Transaction: `SE80`.

---

## R

**RAP (RESTful Application Programming)**
SAP's modern application development model, replacing SEGW-based OData in new development. Built on CDS views + behaviour definitions + service definitions. Strongly typed, annotation-driven, and designed for ABAP Cloud. Like moving from ASP.NET Web Forms to ASP.NET Core Minimal API. See Chapter 35.

**RFC (Remote Function Call)**
A protocol for calling Function Modules across system boundaries — between SAP systems, or from .NET / Java / Python to SAP via libraries like `pyrfc` or the SAP JCo connector. Like gRPC or a named pipe, but SAP-specific. See Chapter 12.

**RICEF**
An acronym used in SAP project scoping: **R**eports, **I**nterfaces, **C**onversions, **E**nhancements, **F**orms. These are the six main categories of ABAP development work. If someone asks "how many RICEFs are in scope?" they want a count of these object types. Knowing the acronym makes you sound experienced in day one.

---

## S

**S/4HANA**
The current generation of SAP's flagship ERP, running exclusively on SAP HANA. The successor to SAP ECC. Important for developers: S/4HANA enforces stricter coding rules (no direct reads of certain tables, no SELECT *, deprecation of classic APIs), pushing toward CDS + RAP + ABAP Cloud.

**SAPGUI**
The thick-client desktop application for interacting with SAP systems — menus, transaction codes, screens. Like a terminal emulator, but with forms. Developers still use it daily for many tasks even when writing code in ADT (Eclipse). On newer systems you can use SAP Business Client or Fiori Launchpad instead.

**SEGW (SAP Gateway Service Builder)**
The classic transaction for building OData services by designing entity types and wiring them to ABAP code. Like a code-generation wizard that produces DPC/MPC classes. Being superseded by RAP in new development, but still extremely common in existing systems. See Chapter 18.

**Selection Screen**
A parameter-input screen that SAP generates automatically from `SELECT-OPTIONS` and `PARAMETERS` statements in a report. Like a Windows dialog box that collects filter criteria before a report runs. No HTML, no controllers — SAP renders it from declarations.

**Smartforms / Adobe Forms**
SAP's printable form technologies. Smartforms is the older SAPScript successor (transaction `SMARTFORMS`); Adobe Forms (transaction `SFP`) is the modern standard, using Adobe LiveCycle Designer. Like RDLC / Crystal Reports / SSRS, but deeply integrated with SAP print and archiving. See Chapter 10.

**SMOD / CMOD**
`SMOD` = view available SAP user exits (the older enhancement approach). `CMOD` = create/manage enhancement projects that activate those exits. Used for classic (pre-BAdI) enhancements. See Chapter 14.

**SOAMANAGER**
Transaction for managing SOAP/web service configurations and bindings in SAP. Used when consuming or publishing traditional SOAP services.

**SPROXY (Service Proxy)**
Transaction for generating ABAP proxy classes from a WSDL, so you can call an external SOAP web service from ABAP. Like using `dotnet-svcutil` to generate a WCF client proxy. See Chapter 19.

**Structure**
A DDIC type (or locally declared type) that groups fields — like a C# `struct` or a Python `dataclass`. Structures have **value semantics**: assigning one to another copies all fields. A structure used as one row of an internal table is called a **work area**.

---

## T

**Transaction Code (T-Code)**
A short code you type into the SAP command field to jump directly to a function — like a keyboard shortcut or a URL path for SAPGUI screens. `SE11` opens the DDIC, `SE38` opens the ABAP editor, `SE37` opens Function Module maintenance, etc. See Appendix C for a full reference.

**Transport Request**
A package of ABAP objects (classes, tables, programs, customising entries) that moves together through the system landscape (DEV → QAS → PRD). Like a pull request + deployment bundle in one: you develop in DEV, release the transport, and the Basis team imports it to production. Everything you create in ABAP is assigned to a transport. Transactions: `SE01`, `SE09`, `SE10`, `STMS`.

---

## U

**UI5 (SAPUI5 / OpenUI5)**
SAP's JavaScript UI framework — like React or Angular but with hundreds of enterprise-grade components and deep SAP system integration. Fiori apps are built on UI5. OpenUI5 is the open-source version. You can write UI5 apps without touching ABAP if you just consume OData services.

---

## W

**Work Area**
A single-row variable that holds one row's worth of data — used to read from or write to an internal table. Like a `Customer customer;` variable in C# that you populate from a list. In ABAP: `DATA ls_customer TYPE zcustomer.` where `ls_` is the conventional prefix for a "local structure" / work area.

**Work Process**
An OS-level process inside the SAP application server that handles one request at a time. SAP has a fixed pool of work processes (Dialog, Background, Update, Enqueue, Spool). Like a thread-pool worker, but heavier — you can see them in `SM50`. Understanding them matters when debugging performance and deadlocks.

---

## Additional terms worth knowing

**Activation** — Before an ABAP object is usable, it must be *activated* (like compiling). In ADT: Ctrl+F3. If you can't find your new table or class, you probably forgot to activate it.

**BASIS** — The SAP system administration team. They manage transports, user authorisations, system copies, kernel upgrades. Build a good relationship with Basis; they control what gets to production.

**Client-Independent vs Client-Dependent** — Some DDIC objects (like table *definitions*) are client-independent (exist in all clients). Data rows in most business tables are client-dependent (each client has its own rows). Cross-client tables (e.g., `T001` company codes) look the same from any client.

**Enhancement Spot** — A defined hook point in SAP standard code where you can inject custom logic via a BAdI or Implicit Enhancement. Like a middleware pipeline insertion point.

**ICF (Internet Communication Framework)** — SAP's HTTP server framework that handles inbound web requests. Used for OData, REST, SOAP, and custom HTTP handlers inside SAP.

**Message Class** — A container for numbered error/warning/info/success messages (`SE91`). Like resource files for user-facing messages. Referenced as `MESSAGE E001(zmessage_class)` in ABAP.

**Namespace** — SAP objects owned by customers must be in the customer namespace (`Z*` or `Y*` prefix) or a registered namespace (`/COMPANY/`). You'll always start object names with `Z` unless your company has a registered namespace.

**Screen Painter** — The visual screen designer for Module Pool programs inside SAPGUI / SE80. Like the WinForms designer, but older and quirkier.

**SE16N / SE16** — Table data browser transactions. SE16N is the newer version. Like running `SELECT * FROM table` in SQL Server Management Studio — fast way to check what's in a table during development and debugging.

**SY (System fields)** — The `SY` structure (e.g., `sy-subrc`, `sy-datum`, `sy-uname`, `sy-tabix`) is ABAP's equivalent of environment variables: filled automatically by the runtime. Always check `sy-subrc` after statements that can fail.

---

*[← Contents](../content.md) | [← Previous: Appendix A — Cheat Sheet](appendix-a-csharp-abap-cheatsheet.md) | [Next: Appendix C — Transaction Codes →](appendix-c-transaction-codes.md)*
