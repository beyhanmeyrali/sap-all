# Chapter 17: Clean Core Rules That Bite Old Habits

> *The Mindset Shift Every ABAP Developer Must Make*

---

## 17.1 What Can't You Do Anymore?

In a Clean Core / RISE environment:

❌ **No modifications to standard code**
- No SMOD/CMOD implementations
- No enhancement points in SAP code

❌ **No custom objects in S/4 core**
- No Z-tables in DDIC
- No Z-programs in SE38

❌ **No unreleased APIs**
- Can't call internal function modules
- Must use released APIs only

❌ **No direct database access**
- No native SQL
- Must go through released interfaces

---

## 17.2 Extensions vs. Modifications

### Old Way: Modifications
```abap
" Inside standard program
ENHANCEMENT my_enhancement
  " Change standard behavior
  " Risk: Breaks on upgrade
ENDENHANCEMENT.
```

### New Way: Extensions
```
┌─────────────────────┐      ┌─────────────────────┐
│  S/4HANA (Clean)    │      │  BTP Extension      │
│                     │      │                     │
│  Standard behavior  │ API  │  Custom behavior    │
│  Released APIs      │─────►│  RAP services       │
│                     │      │  Side-by-side       │
└─────────────────────┘      └─────────────────────┘
```

---

## 17.3 Adapting Your Development Mindset

| Old Habit | New Approach |
|-----------|--------------|
| Add Z-field to standard table | Key User extensibility or custom CDS |
| Create Z-report | Create RAP service + Fiori app |
| Modify user exit | Use BAdI (released) or side-by-side |
| Direct table access | OData service or released CDS |
| Copy & modify std program | Build new in BTP |

### The Key Question

Before coding, ask: *"Can I do this without touching the S/4 core?"*

If yes → Do it in BTP  
If no → Find a released alternative or push back on requirement

---

## Benefits of Clean Core

- ✅ Faster upgrades (no conflicts)
- ✅ Easier support (standard behavior)
- ✅ Cloud-ready (can move to RISE)
- ✅ Future-proof (SAP's direction)

---

*[Previous: Chapter 16](16-cloud-connector.md) | [Next: Appendix A](appendix-a-reference-tables.md)*

*[Back to Table of Contents](../content.md)*
