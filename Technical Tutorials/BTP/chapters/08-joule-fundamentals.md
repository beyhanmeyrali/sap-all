# Chapter 8: Joule Fundamentals

> *SAP's AI Copilot Explained*

---

## 8.1 What Is Joule? (The SAP AI Copilot)

**Joule** is SAP's generative AI assistant that:
- Lives inside SAP applications
- Understands SAP data and processes
- Can be extended with custom capabilities

Think of it as a **company-specific ChatGPT** that knows your SAP landscape.

---

## 8.2 Skills vs. Agents: The Key Difference

### Skill = One Superpower

A skill is a single, specific capability:
- "Look up sales order status"
- "Calculate delivery date"
- "Call a weather API"
- "Post to Microsoft Teams"

### Agent = Smart Boss

An agent decides which skills to use and in what order:

**Example**: "Help me handle a customer complaint"

Agent might chain:
1. Look up order (Skill A)
2. Check stock (Skill B)
3. Create return request (Skill C)
4. Send apology email (Skill D)

**You build skills first, then give them to agents.**

---

## 8.3 How Skills "Do" Things (The Action Flow)

Most useful skills need to talk to real systems:
- SAP systems (S/4HANA, SuccessFactors)
- External APIs
- Databases

### The Secure Flow

```
Joule Agent
    ↓
Skill
    ↓
Action (from Action Project)
    ↓
Destination
    ↓
Actual REST/OData Endpoint
```

No passwords in skills. Everything goes through destinations.

---

## 8.4 Why Destinations Are Required for Skills

Joule skills cannot contain:
- ❌ Raw URLs
- ❌ Credentials
- ❌ API keys directly

**Reason**: Enterprise security and governance

Destinations provide the secure, reusable way to configure:
- Where to connect
- How to authenticate
- Any proxy or tunnel needed

---

*[Previous: Chapter 7](07-fiori-ui5-btp.md) | [Next: Chapter 9](09-first-joule-skill.md)*

*[Back to Table of Contents](../content.md)*
