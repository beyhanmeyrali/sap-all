# Appendix C: Transaction Code Reference

*The codes you'll type every single day — grouped, described, and demystified.*

---

## 📌 What is a transaction code?

A transaction code (T-code) is the fastest way to navigate SAP. Type it into the command field in the top-left of SAPGUI and press Enter — you go straight there. No menus, no navigation tree.

> 💡 **Navigation tips before you start:**
> - Type `/n` before a code to close the current screen and open the new one: `/nSE11`
> - Type `/o` before a code to open it in a **new SAPGUI window** (new session): `/oSE37`
> - Type `/nEX` to log off. Type `/i` to delete the current session.
> - You can also just enter the code directly if you're already on the SAP Easy Access menu — no slash needed.
> - In ADT (Eclipse) you access most functionality through the repository browser, but you'll still type T-codes into a SAPGUI window for things ADT doesn't cover.

---

## 🛠️ Development

| T-Code | What it's for |
|---|---|
| `SE11` | **Data Dictionary** — create/edit tables, structures, data elements, domains, type groups, search helps. Your first stop for anything data-model related. |
| `SE16N` | **Table data browser (new)** — view the actual rows in any table. Like running SELECT * in SSMS. Use during development to verify your data writes. |
| `SE16` | **Table data browser (classic)** — older version of SE16N; still works, still used. |
| `SE24` | **Class Builder** — create and maintain ABAP classes and interfaces in the old workbench. You'll use ADT for this in new projects, but SE24 is still used for viewing/modifying legacy classes. |
| `SE37` | **Function Builder** — create, test, and maintain Function Modules. Also where you test BAPIs interactively before writing calling code. |
| `SE38` | **ABAP Editor** — the classic program (report) editor. Like Notepad++ for ABAP programs. ADT is better for new development; SE38 is still used for quick checks. |
| `SE80` | **Object Navigator / ABAP Workbench** — the old all-in-one IDE. Browse packages, programs, classes, function groups, everything. You'll use ADT more often, but SE80 is useful for navigating unfamiliar codebases. |
| `SE91` | **Message Maintenance** — create and edit message classes (the numbered error/warning/success messages used in ABAP programs). |
| `SE93` | **Transaction Maintenance** — create new transaction codes that call your programs or screens. |
| `ADT` | Not a T-code — **ABAP Development Tools** is the Eclipse plugin. Open Eclipse, connect to your system, and develop there. Supports syntax highlighting, code completion, refactoring, unit tests, and the ABAP debugger. The recommended IDE for all modern ABAP development. |

---

## 🔍 Data Browsing

| T-Code | What it's for |
|---|---|
| `SE16N` | Browse any table with filters, column selection, and export. The developer's best friend for checking data quickly. |
| `SE16` | Classic table browser — same purpose as SE16N, slightly less featured. Still faster than writing a SELECT for a quick check. |
| `SE17` | General Table Display — an older read-only viewer. Less used today but still available. |

> 💡 In SE16N, press **F8** (Execute) or the clock icon to run the query. Use the **Settings** menu to increase the row limit if your query returns "max rows reached."

---

## 🐛 Debugging & Runtime Analysis

| T-Code | What it's for |
|---|---|
| `/h` | **Activate the ABAP Debugger** — type `/h` in the command field and press Enter before triggering the action you want to debug. SAP will stop at the next ABAP statement. This is the `F9 / Attach debugger` equivalent. Works in SAPGUI; ADT has its own integrated debugger. |
| `ST22` | **ABAP Runtime Error dump log** — shows short dumps (uncaught exceptions that crashed a program). Like the Windows Event Log / Application Insights for unhandled exceptions. First place to check when "the program died with a short dump." |
| `SM50` | **Work Process Overview** — shows currently running work processes. Like Task Manager for SAP server processes. Use it to see if a long-running program is stuck or if a dialog process is consuming all CPU. |
| `SM37` | **Background Job Monitor** — view the status, logs, and spool output of background jobs. Like checking a Windows Task Scheduler job's last run result. |
| `SAT` | **ABAP Runtime Analysis (Trace)** — profiles performance: which statements take the longest, how many DB calls were made, etc. Like a lightweight profiler / SQL Server Execution Plan viewer. |
| `ST05` | **SQL Trace / Performance Trace** — records every database statement executed during a transaction, with execution times. Invaluable for finding missing indices, full table scans, and N+1 SELECT problems. Like SQL Server Profiler or EF Core's query logging. |

---

## 🌐 OData / Gateway

| T-Code | What it's for |
|---|---|
| `SEGW` | **SAP Gateway Service Builder** — the classic tool for building OData services. Design entity types, generate DPC/MPC classes, implement CRUD logic. The go-to for OData development in systems before RAP took over. See Chapters 21–31. |
| `/IWFND/MAINT_SERVICE` | **Maintain OData Services** — register and activate an OData service so it's reachable via the gateway. Like deploying an endpoint. If your service exists in SEGW but can't be called, it probably hasn't been activated here. |
| `/IWFND/GW_CLIENT` | **Gateway Client** — a built-in HTTP client for testing OData requests directly in SAP. Like Postman, but inside the system. Essential for testing your OData services without leaving SAP. |
| `/IWFND/ERROR_LOG` | **Gateway Error Log** — shows errors thrown by OData service calls. First place to look when your Fiori app or external client says "500 Internal Server Error." |

---

## 📨 IDoc / ALE

| T-Code | What it's for |
|---|---|
| `WE02` | **IDoc List** — browse IDocs by type, status, date, and partner. Your main view for checking whether an IDoc was created and processed correctly. |
| `WE05` | **IDoc Lists (extended)** — an alternative IDoc browser with more filter options. |
| `WE19` | **Test Tool for IDoc Processing** — reprocess or manually trigger an IDoc. Like re-running a failed message from a message queue. |
| `WE20` | **Partner Profile Maintenance** — configure which IDocs flow to/from which business partners, and how (port, message type, etc.). Like configuring a message broker topic subscription. |
| `BD87` | **Status Monitor for ALE Messages** — find and reprocess failed inbound IDocs in bulk. Useful after fixing the root cause of a batch of failed messages. |
| `WE60` | **IDoc Type Documentation** — view the structure and field documentation for an IDoc type. Like reading a Swagger schema for an IDoc. |

---

## 🔌 Web Services (SOAP)

| T-Code | What it's for |
|---|---|
| `SOAMANAGER` | **SOA Manager** — manage SOAP web service configurations, logical ports, and bindings. Use this to configure a consumed or published SOAP service endpoint URL and authentication. |
| `SPROXY` | **Enterprise Services Browser / Proxy Generation** — generate ABAP proxy classes from a WSDL so you can call an external SOAP service from ABAP. Like running `dotnet-svcutil` or `Add Service Reference` in Visual Studio. See Chapter 19. |

---

## 🧩 Enhancements

| T-Code | What it's for |
|---|---|
| `SMOD` | **Enhancement Display** — view the available SAP user exits (the older enhancement framework). Useful for finding where SAP provides a hook point in standard code. |
| `CMOD` | **Enhancement Management** — create and manage enhancement *projects* that bundle and activate user exits. The activation point for classic enhancements. |
| `SE18` | **BAdI Builder** — view BAdI definitions: where SAP placed a hook, what the interface looks like. Like reading an interface definition before implementing it. |
| `SE19` | **BAdI Implementation** — create your implementation of a BAdI. Like adding a class that implements an interface. See Chapter 14. |

---

## 📄 Forms

| T-Code | What it's for |
|---|---|
| `SMARTFORMS` | **Form Builder for Smartforms** — design and maintain Smartforms print forms (the predecessor to Adobe Forms). Still common in many live systems. See Chapter 10. |
| `SFP` | **Adobe Forms / Form Builder** — create and maintain Adobe LiveCycle-based print and interactive forms. The current standard for new form development. See Chapter 10. |
| `SE71` | **SAPscript Form Painter** — the oldest SAP form technology (pre-Smartforms). You'll encounter it in very old systems. Mostly legacy-only. |

---

## 📦 Data Migration

| T-Code | What it's for |
|---|---|
| `SHDB` | **BDC Recording** — record a SAPGUI screen session for later playback as a BDC batch input. Like Selenium record-and-replay, but for SAP screens. See Chapter 15. |
| `LSMW` | **Legacy System Migration Workbench** — a structured tool for migrating data via BDC recording, BAPI calls, or IDocs. Point-and-click data load framework. See Chapter 15. |
| `LTMC` | **SAP S/4HANA Migration Cockpit** — the modern S/4HANA data migration tool. Uses migration templates (Excel/XML) and staging tables instead of BDC. Replaces LSMW in S/4HANA greenfield projects. |

---

## 🚂 Transport Management

| T-Code | What it's for |
|---|---|
| `SE01` | **Transport Organiser (Extended)** — the full view of all transport requests in the system. Create, display, and manage transports. |
| `SE09` | **Workbench Transport Organiser** — view and manage transports for ABAP development objects (programs, classes, tables). Your main transport management screen as a developer. |
| `SE10` | **Customising Transport Organiser** — view and manage transports for configuration / customising changes (not code). |
| `STMS` | **Transport Management System** — the system-to-system transport control centre. Basis uses this to import transports from DEV to QA to PRD. As a developer you'll use it to check import queues and statuses. |

---

## 🧭 Other useful T-codes worth memorising early

| T-Code | What it's for |
|---|---|
| `SU01` | User maintenance — create/modify user accounts (mainly Basis, but useful to know). |
| `SU53` | Show last authorisation check failure — tells you exactly which authorisation object caused an "unauthorised" error. Share this screenshot with your Basis team when reporting auth issues. |
| `SPRO` | SAP Reference IMG — the customising (configuration) entry point. Most ABAP developers visit it rarely, but every functional consultant lives here. |
| `SM30` / `SM31` | Table View Maintenance — configure customising entries in `V_`-prefix views. Like editing a reference-data table via a generated CRUD screen. |
| `SLG1` | Application Log Viewer — read logs written with the `BAL_*` function modules. |
| `SICK` | Installation Check — confirms the system is healthy; rarely needed but good to know. |

---

*[← Contents](../content.md) | [← Previous: Appendix B — Glossary](appendix-b-glossary.md) | [Next: Appendix D — Resources & Job Prep →](appendix-d-resources.md)*
