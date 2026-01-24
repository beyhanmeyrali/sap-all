# Chapter 11: Agent Lifecycle & Deployment

> *From Dev to Prod—The Right Way*

---

## 11.1 Dev → Test → Prod: The Standard Lifecycle

Typical setup for one client:

```
DEV subaccount → Build & test (sandbox destinations)
TEST subaccount → Integration test (real-ish data)
PROD subaccount → Live (production destinations)
```

Each has its own:
- Destinations
- Deployed agents/skills
- User assignments

---

## 11.2 Transferring Projects Across Tenants

**Key Truth**: Joule agents are NOT transported like classic ABAP.

Instead: **Export / Transfer + Redeploy**

1. Build in DEV Joule Studio
2. Export/Transfer project
3. Import in PROD Joule Studio
4. Reconfigure destinations
5. Deploy

---

## 11.3 The SAP Build Promote Feature

Some SAP Build capabilities have a **Promote** button:
- Works similar to workflow promotion
- Check if your subaccounts have formations set up
- Not always available for all agent types yet

---

## 11.4 Manual + Git Version Control

The consultant favorite:
1. Export skills/agents as JSON bundles
2. Store in Git (with destination placeholders)
3. Import in target subaccount
4. Adjust configurations
5. Deploy

Benefits: Version history, rollback, code review.

---

## 11.5 Environment-Specific Configuration

What changes per environment:

| Item | DEV | PROD |
|------|-----|------|
| Destination | `*_DEV_*` | `*_PROD_*` |
| AI Model | Cheaper/smaller | Production model |
| Grounding docs | Test docs | Real docs |
| Users | Developers | End users |

---

*[Previous: Chapter 10](10-building-agents.md) | [Next: Chapter 12](12-multi-client-management.md)*

*[Back to Table of Contents](../content.md)*
