# Chapter 13: Cross-Customer Deployments

> *Same Solution, Different Clients*

---

## 13.1 Pattern 1: Template + Per-Client Recreation

**Most common today for consulting work.**

### The Flow

1. Build a **golden template** in internal subaccount
2. Export it (project bundle/JSON)
3. For each client:
   - Create client-specific subaccount
   - Import the template
   - Customize: destinations, grounding, prompts
   - Deploy and promote (DEV → TEST → PROD)

### Pros & Cons

✅ Full isolation  
✅ Client owns everything  
✅ Easy handover  
❌ Some rework per client

---

## 13.2 Pattern 2: Multi-Tenant SaaS Mode

**For productized solutions.**

- Deploy once in provider subaccount
- Each client subscribes (consumer subaccount)
- Shared codebase, isolated data

Needs: CAP multi-tenant or ABAP Env multi-tenancy

---

## 13.3 When to Use Which Pattern

| Scenario | Pattern |
|----------|---------|
| Consulting project | Template + Recreation |
| Selling a product | Multi-Tenant SaaS |
| POC/Demo | Template |
| Long-term partnership | Either |

---

## 13.4 Comparison Table

| Aspect | Template + Recreation | Multi-Tenant SaaS |
|--------|----------------------|-------------------|
| Effort per client | Medium | Low (after setup) |
| Initial setup | Low | High |
| Isolation | Very high | High |
| Handover ease | High | Medium |
| Reusability | High | Very high |

---

*[Previous: Chapter 12](12-multi-client-management.md) | [Next: Chapter 14](14-c-level-agents.md)*

*[Back to Table of Contents](../content.md)*
