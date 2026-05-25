# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A **content repository** of SAP learning material written in Markdown — not a software project. There is no build system, package manager, test suite, or lint config. Work here is writing, editing, translating, and organizing prose, tables, and Mermaid diagrams. "Building" means rendering Markdown; validation means reading the files and checking internal links.

The author/voice is Beyhan Meyrali. All content is teaching material aimed at SAP developers and consultants.

## Four bodies of content

1. **`Mini Tutorials - Introduction to ERP World/`** — Story-driven introductions to SAP ERP modules (MM, PP, SD, FI, CO, HCM, QM, PM, etc.), one numbered folder per topic (`01-overview` … `13-reporting`, plus `logistics-execution`). Each folder holds a `README.md`. Narrative-heavy: recurring personas (Maria, James, Marcus, Emma's lemonade stand) teach concepts through scenarios rather than reference-style enumeration.

2. **`Technical Tutorials/BTP/`** — "SAP BTP Mini Book for Old ABAPers", a structured book. `README.md` is the front matter, `content.md` is the table of contents, `links.md` collects source links, and `chapters/NN-name.md` holds each chapter (`00-preface` … `17-clean-core`, plus `appendix-a`…`appendix-d`). Uses the Feynman technique: plain language, analogies (apartment buildings, phone books), Mermaid diagrams. Note: `README.md` references `todo.md` and `agent-resources.md` that are not present.

3. **`Technical Tutorials/CSharp-to-ABAP/`** — "From C# & Python to SAP ABAP — The Career-Switcher's Field Guide", a structured book using the same layout as the BTP book (`README.md`, `content.md` TOC, `chapters/NN-name.md` for `00-preface` … `35-rap-restful-application-programming`, plus `appendix-a`…`appendix-d`). Teaches SAP ABAP to an experienced C#/.NET + Python developer aiming to become job-ready. Signature convention: the **three-pass method** per concept — (1) analogy, (2) "You already know this" with C#/Python code, (3) "The ABAP way" with real ABAP — plus `> ⚠️ C#/Python gotcha` and `> 🧭 On the job` callouts, Mermaid diagrams, and a `## 🧠 Recap` + prev/next nav footer on every chapter. Bilingual via a parallel `TR/` subtree (mirrors `chapters/` plus its own `README.md`/`content.md`), same convention as the BTP book — keep the two in sync.

4. **`Technical Tutorials/sap_development_handbook.md`** — A single large (~2400-line) reference doc covering ABAP / RAP / CAP / BTP with an anchor-linked internal TOC. Edit in place; keep the TOC anchors in sync with section headings.

`My Sap Blog Posts/` contains published-article PDFs (read-only source material, not edited here).

## Bilingual (English + Turkish) — two different conventions

Content is maintained in both English and Turkish, but the two areas use **different patterns** — match the one already in use where you are editing:

- **Mini Tutorials**: Turkish lives in a sibling file with a `_tr.md` suffix (e.g. `README.md` → `README_tr.md`) in the same folder.
- **BTP book**: Turkish lives in a parallel `TR/` subtree that mirrors the English structure (`BTP/chapters/01-...md` → `BTP/TR/chapters/01-...md`), including its own `README.md` and `content.md`.

When you add or substantially change English content, update the corresponding Turkish file (or note it as pending). Turkish keeps SAP technical terms in English (e.g. "subaccount", "destination", "Clean Core") rather than translating them.

## Conventions when editing

- **Mermaid diagrams** are used throughout BTP chapters (```` ```mermaid ````, mostly `graph LR`/`graph TD` with colored node styles). Preserve and extend this style rather than replacing diagrams with prose.
- **Relative Markdown links** connect chapters/TOCs. When renaming or moving a file, update the links in `README.md`, `content.md`, and any chapter that points to it. Spaces in folder names are URL-encoded (`%20`) in links.
- Keep the established teaching voice (analogies, named personas, emoji section markers) consistent with the surrounding file — don't flatten narrative tutorials into dry reference text or vice versa.

## Git

Commit messages in history are descriptive content summaries (e.g. "Complete Turkish translation for SAP BTP Mini Book"). Default branch is `main`.
