# Chapter 10: Building Joule Agents

> *The Smart Boss That Chains Skills*

---

## 10.1 What Makes an Agent "Smart"

An agent can:
- **Reason** about which skill to use
- **Chain** multiple skills in sequence
- **Handle** complex multi-step requests
- **Ground** responses in documents

Unlike a simple skill (one action), an agent orchestrates.

---

## 10.2 Assigning Skills to Agents

In Joule Studio:
1. Create or edit an **Agent**
2. Go to **Skills** tab
3. Add the skills you've built
4. Define when each skill applies

The agent now has a "toolkit" of capabilities.

---

## 10.3 Multi-Step Reasoning and Chaining

Example: "Handle customer complaint for order 12345"

Agent reasoning:
1. User wants complaint handled
2. Need order details → Use **Order Lookup Skill**
3. Check if return possible → Use **Return Policy Skill**
4. Create return → Use **Create Return Skill**
5. Notify customer → Use **Email Skill**

This happens automatically based on the agent's instructions.

---

## 10.4 Grounding Agents on Client Documents

**Grounding** = Teaching the agent to use specific knowledge.

You can ground on:
- Company policies (PDFs)
- Product catalogs
- SAP MDG data
- Custom knowledge bases

This makes the agent context-aware for your organization.

---

*[Previous: Chapter 9](09-first-joule-skill.md) | [Next: Chapter 11](11-agent-lifecycle.md)*

*[Back to Table of Contents](../content.md)*
