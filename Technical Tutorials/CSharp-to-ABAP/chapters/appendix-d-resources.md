# Appendix D: Resources, Practice Systems & Job Prep

*Everything you need to go from finishing this book to passing the interview — and surviving the first month on the job.*

---

## 🛠️ Practice systems: get a real SAP to type in

You cannot learn ABAP by reading alone. You need a live system where you can write code, break things, fix them, and run transactions. Here are your realistic options.

### Option 1 — ABAP Platform Trial (Docker "Developer Edition") — recommended for classic ABAP

SAP publishes an official community edition of the ABAP application server as a Docker container. It gives you a full on-prem-style system: `SE11`, `SE37`, `SE38`, `SE80`, `SEGW`, `WE02` — the complete classic toolset — running on your own laptop.

**What you get:** A fully functional SAP system with a development client, a real ABAP stack, SAPGUI connectivity, and ADT (Eclipse) support.

**How to find it:**
- Search for **"ABAP Platform Trial SAP Developer Edition"** on the SAP Community website or GitHub.
- The official GitHub organisation is **`SAP-samples`** — look for the repository named **`abap-platform-trial-docker`** or similar.
- SAP also publishes the trial as a VirtualBox appliance — search **"SAP NetWeaver AS ABAP Developer Edition"** on the SAP Community or SAP Support Portal download area.

**Minimum system requirements:** ~16 GB RAM recommended (8 GB possible but slow), 100+ GB disk space, Docker Desktop or VirtualBox.

**Setup outline (Docker path):**
1. Install Docker Desktop.
2. Pull the SAP ABAP Platform Trial image (instructions in the GitHub repository README).
3. Start the container — first boot takes 5–15 minutes while the database starts.
4. Download SAPGUI (available free from SAP for development use — search "SAP GUI for Windows download").
5. Configure a SAPGUI connection to `localhost`, system number `00`, client `001`.
6. Log in with the default developer credentials (listed in the repo README).
7. Install ADT (Eclipse + ABAP Development Tools plugin) and add the same system as an ABAP project.
8. Done — you have a real ABAP system on your laptop.

> ⚠️ SAP updates these images occasionally and the exact image name changes. Always follow the current README in the official SAP repository rather than this outline.

---

### Option 2 — SAP BTP ABAP Environment Trial

SAP offers a **free trial of the BTP ABAP Environment** — a cloud-hosted ABAP system focused on modern development (ABAP Cloud, RAP, CDS, Tier 1 API surface only). No SAPGUI — you work entirely in ADT (Eclipse) or the ABAP Development tools in SAP Business Application Studio.

**What you get:** Modern ABAP Cloud capabilities: CDS, RAP, ABAP Unit, released APIs. You cannot use classic transactions (SE80, SEGW, SE37 for custom FMs) because those aren't part of ABAP Cloud's allowed surface area.

**How to find it:**
- Go to **trial.sap.com** and create a free BTP trial account.
- In the BTP Cockpit, navigate to **Boosters** and search for **"Prepare an Account for ABAP Trial"** — it auto-configures the environment for you.
- Connect ADT (Eclipse) to the provisioned system using the system URL and your BTP credentials.

**Best for:** Practising Part VIII content (RAP, CDS) and understanding ABAP Cloud restrictions. Pair it with the Docker Developer Edition for Parts II–VII.

---

### Option 3 — openSAP courses with system access

Several **openSAP** courses (SAP's free MOOC platform — search **"openSAP"**) include temporary ABAP system access as part of the course. Check currently running or self-paced courses on the openSAP platform. Key course titles to look for:

- **"Writing Testable Code for ABAP"** — ABAP Unit, clean code
- **"Building Apps with the ABAP RESTful Application Programming Model"** — RAP from scratch
- **"Efficient Development with ABAP"** — modern ABAP syntax
- **"Introduction to Application Development Using ABAP"** — entry-level, good for building the mental model

These courses are free; system access (where included) is temporary (typically for the duration of the course).

---

## 📚 Learning resources: where to go when you're stuck

### Official documentation

| Resource | What it covers | How to find it |
|---|---|---|
| **SAP Help Portal** | The authoritative reference for every SAP product, transaction, and API. Overwhelming at first but indispensable. | Search **"SAP Help Portal"** — it's at `help.sap.com`. |
| **ABAP Keyword Documentation** | The complete ABAP language reference — every statement, every clause, with examples. Like the C# language specification but readable. In ADT: position cursor on a keyword and press **F1**. Also browsable at the SAP Help Portal under "ABAP — Reference". | `help.sap.com` → ABAP |
| **SAP RAP documentation** | The official guide for the RESTful Application Programming Model. Covers behaviour definitions, projections, service definitions, draft handling. | Search **"SAP RAP ABAP RESTful Application Programming Model help"** |
| **ABAP Platform release notes** | What's new in each ABAP version. Essential when moving between system versions. | SAP Help Portal |

### Community and open source

| Resource | What it covers | How to find it |
|---|---|---|
| **SAP Community** | Q&A forums, blog posts, expert answers. Stack Overflow equivalent for SAP questions — ask questions, search existing threads. | Search **"SAP Community"** |
| **Clean ABAP Style Guide** | An opinionated, comprehensive guide to writing clean, modern, readable ABAP. Like the Google Style Guide but for ABAP. Published by SAP on GitHub. | Search **"Clean ABAP GitHub SAP"** — repository is `SAP/styleguides` under `clean-abap/`. |
| **abapGit** | The open-source Git client for ABAP — lets you version-control ABAP objects and share code via GitHub. Essential for portfolio projects. | Search **"abapGit"** — project is at `abapGit.org` and the GitHub repository is `abapGit/abapGit`. |
| **ABAP Open Checks (abaplint)** | A static analysis tool for ABAP code, usable in CI pipelines with abapGit. Like Roslyn analyzers for ABAP. | Search **"abaplint"** on GitHub. |
| **blogs.sap.com** | SAP's official blogging platform — thousands of practitioner-written tutorials, deep dives, and how-tos. High signal-to-noise if you filter for recent posts. | `blogs.sap.com` |

---

## 🧭 Job prep: from reader to hired

### Skills recruiters actually screen for

SAP ABAP job postings cluster around a predictable set of skills. Here is what appears most often, roughly in order of how often it's a deal-breaker:

| Skill area | What interviewers test | Where in this book |
|---|---|---|
| **ABAP syntax & OOP** | Write a class, use an internal table, implement an interface | Chapters 6, 7, 11 |
| **DDIC** | Create a table/structure, explain data elements vs. domains | Chapter 5 |
| **ALV reports** | Build a CL_SALV_TABLE report from scratch | Chapter 8 |
| **Debugging** | Use `/h`, navigate call stack, set watchpoints | Chapter 4, Appendix C |
| **BAPI / RFC** | Call a BAPI, handle return table, commit work | Chapters 12, 13 |
| **Enhancements** | Explain BAdI vs. user exit, implement a BAdI | Chapter 14 |
| **OData / SEGW** | Build a simple GET_ENTITY / GET_ENTITYSET service | Chapters 18–23 |
| **CDS Views** | Write a basic CDS view with associations and annotations | Chapter 16 |
| **RAP** | Explain behaviour definition, managed vs. unmanaged | Chapter 35 |
| **Transport management** | Explain DEV→QAS→PRD flow, what a transport request is | Appendix B, C |
| **SQL / performance** | Avoid SELECT *, use FOR ALL ENTRIES correctly, explain ST05 | Chapters 6, 7 + Appendix C |

---

### Common interview questions — and how to answer them

**"What is an internal table? How does it differ from a structure?"**
An internal table is an in-memory list of rows (like `List<T>`); a structure is the definition of a single row (like a `struct`). A work area (`ls_*`) holds one row at a time; the internal table (`lt_*`) holds many.

**"What is `sy-subrc` and why do you always check it?"**
It's the system return code — set to `0` on success, non-zero on failure — by database statements, `READ TABLE`, `CALL FUNCTION`, etc. There are no automatic exceptions for most failures; if you don't check `sy-subrc`, you silently continue on bad data.

**"Explain the difference between BAPI, RFC, and a Function Module."**
A Function Module is the basic reusable code unit. RFC-enabled means it can be called remotely (from another system or program). A BAPI is an RFC-enabled Function Module that SAP has published as an official, stable business object interface (versioned, documented, with a standard parameter convention).

**"What is a BAdI and when would you use one instead of a user exit?"**
A BAdI is the modern enhancement framework — object-oriented, supports multiple simultaneous implementations, managed via SE18/SE19. User exits are the classic approach (SMOD/CMOD) — procedure-based, only one implementation per exit. For new development always prefer BAdI; for old systems you often have to use whichever the SAP standard code calls.

**"What is a transport request and what happens if you forget to add an object to one?"**
A transport request is the unit that moves ABAP objects from DEV to QAS to PRD. If you create an object without a transport (local `$TMP` package), it exists only in DEV and can never go to production. This is a common beginner mistake that causes "it works in dev but not in prod" situations.

**"What is the difference between a STANDARD, SORTED, and HASHED internal table?"**
STANDARD = array-like, no inherent order, O(n) linear search. SORTED = auto-sorted on key fields, O(log n) binary search. HASHED = hash map on a unique key, O(1) lookup. Use HASHED when you need fast key-based reads on large tables; SORTED when you need both sorted order and fast lookup; STANDARD otherwise.

**"What does COMMIT WORK do and when must you call it?"**
It flushes buffered database changes to permanent storage. After calling a BAPI that creates/modifies data, after your own database INSERT/UPDATE/DELETE, you must call `COMMIT WORK`. Failing to do so means your data appears to insert correctly but is rolled back when the session ends.

**"How do you debug a short dump in production?"**
Go to `ST22`, find the dump by date/user/program, read the error message and the point where it occurred. You cannot run the interactive debugger in production — you analyse the dump text, add logging to the program, and potentially reproduce in DEV.

**"What is the difference between CDS and Open SQL?"**
Open SQL is the ABAP statement syntax for reading/writing data at runtime. CDS (Core Data Services) defines reusable virtual data models (views, associations, annotations) at development time that the database executes as views. Think: Open SQL is like a LINQ query in code; a CDS view is like an EF Core view that's also a data contract for OData and authorisation.

---

### Suggested 10-week study plan

Map this to the book's parts. Adjust the pace — if you're working full-time in ABAP already, compress weeks 1–3.

| Week | Focus | Book chapters | Goal |
|---|---|---|---|
| **1** | SAP world & tooling | Preface, Ch 1–3, Ch 4 | Understand what SAP is; set up ADT + Docker trial system |
| **2** | DDIC & ABAP foundations | Ch 5, Ch 6 | Create a table, write programs, understand sy-subrc |
| **3** | Modern ABAP syntax | Ch 7 | Be fluent in VALUE #(), COND, SWITCH, inline DATA(), string templates |
| **4** | Reports & ALV | Ch 8 | Build a working CL_SALV_TABLE report — put it on GitHub via abapGit |
| **5** | OOP & Function Modules | Ch 11, Ch 12, Ch 13 | Write a class hierarchy; call BAPI_ACC_DOCUMENT_POST in SE37 |
| **6** | Enhancements & migration | Ch 14, Ch 15 | Implement a BAdI; understand BDC recording |
| **7** | CDS, AMDP, IDocs | Ch 16, Ch 17, Ch 20 | Write a CDS view with an association; understand IDoc flow |
| **8** | OData basics + SEGW | Ch 18, Ch 21, Ch 22, Ch 23 | Build a GET_ENTITYSET service end-to-end; test in `/IWFND/GW_CLIENT` |
| **9** | OData deep dive | Ch 24–31 | Add filter/search, CREATE, associations, function imports |
| **10** | RAP + polish | Ch 34, Ch 35 | Build a managed RAP scenario; revisit Appendices A–C; mock interview |

> 💡 **At the end of weeks 4, 8, and 10**, commit working code to a public GitHub repository via abapGit. These three portfolio items — an ALV report, an OData service, and a RAP app — are concrete proof you can do the job. Mention them in your CV and walk through them in interviews.

---

## 📌 Quick portfolio checklist before applying

- [ ] Working ABAP OOP program demonstrating classes, inheritance, interfaces
- [ ] ALV report (CL_SALV_TABLE) reading from a custom Z-table
- [ ] At least one BAPI call with proper error handling and `COMMIT WORK`
- [ ] A BAdI implementation (even a trivial one shows you know the enhancement framework)
- [ ] An OData service with GET_ENTITY and GET_ENTITYSET implemented
- [ ] A CDS view with at least one association
- [ ] A RAP managed scenario (even if basic — it signals you understand where SAP is heading)
- [ ] All of the above in a public GitHub repo via abapGit with a clear README

---

## A closing word

You picked a language and an ecosystem that is genuinely strange to outsiders — and genuinely fascinating once it clicks. ABAP is 40 years old and it still processes a significant fraction of the world's commercial transactions every day. The systems you'll work on matter, the bugs you'll fix matter, and the skills you're building are in real demand.

The curve is steep at the start — the vocabulary is alien, the tooling is eccentric, and the database-is-king philosophy runs counter to everything you learned building stateless web services. But you're not starting from zero. You already know how to think like a developer: how to break a problem down, how to read unfamiliar code, how to debug what you can't see.

Everything else in this book is just **new spellings for things you already do.**

Go build something. Break it. Fix it. Ship it.

You've got this. 🧭

---

*[← Contents](../content.md) | [← Previous: Appendix C — Transaction Codes](appendix-c-transaction-codes.md)*
