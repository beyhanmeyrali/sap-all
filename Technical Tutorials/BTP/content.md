# 📖 Table of Contents

## SAP BTP Mini Book for Old ABAPers

---

### [Preface](chapters/00-preface.md)
- Who This Book Is For (Classic ABAP & Fiori Developers)
- The Feynman Philosophy: No Fluff, Just Coffee & Clarity
- How to Use This Book

---

## Part I: The Big Picture – Understanding SAP BTP

### [Chapter 1: What Even Is SAP BTP?](chapters/01-what-is-sap-btp.md)
- 1.1 The Cloud Toolbox Concept
- 1.2 What's Inside: Databases, AI, Workflows, and Integrations
- 1.3 Where Joule Fits In – SAP's AI Assistant
- 1.4 The Old House vs. The New Neighborhood (On-Prem vs. Cloud Mindset)

### [Chapter 2: The BTP Architecture – Your New Apartment Building](chapters/02-btp-architecture.md)
- 2.1 Global Account – Your Lease Contract with SAP
- 2.2 Directories – Organizing Floors (Optional)
- 2.3 Subaccounts – Your Actual Apartments
- 2.4 Why Isolation Matters: Dev vs. Test vs. Prod
- 2.5 Regions, Compliance, and Data Residency
- 2.6 Entitlements & Quotas – How Much You Can Use

---

## Part II: RISE with SAP – The Managed Cloud Move

### [Chapter 3: RISE with SAP Demystified](chapters/03-rise-with-sap.md)
- 3.1 What RISE Actually Is (It's Not a Product!)
- 3.2 S/4HANA Cloud Private Edition vs. Public Edition
- 3.3 The RISE Bundle: What's Included
- 3.4 Infrastructure & Operations – Say Goodbye to Basis Worries
- 3.5 The Clean Core Promise

### [Chapter 4: How RISE and BTP Fit Together](chapters/04-rise-and-btp.md)
- 4.1 The Neighborhood View: RISE House + BTP Extension Wing
- 4.2 BTP Credits in RISE Contracts
- 4.3 The New Reality for ABAP/Fiori Developers
- 4.4 Comparison Table: Classic On-Prem vs. RISE vs. BTP

---

## Part III: Core BTP Concepts for ABAP Developers

### [Chapter 5: Destinations – The Phone Book for Connections](chapters/05-destinations.md)
- 5.1 What Is a Destination?
- 5.2 Why Destinations Exist (Security & Governance)
- 5.3 Creating Your First Destination
- 5.4 Authentication Types: Basic, OAuth2, Certificates, API Keys
- 5.5 On-Premise Access via Cloud Connector
- 5.6 Naming Conventions That Save Your Sanity

### [Chapter 6: ABAP in the Cloud – The ABAP Environment](chapters/06-abap-cloud.md)
- 6.1 BTP ABAP Environment Overview
- 6.2 How It Differs from Classic On-Prem ABAP
- 6.3 RAP (RESTful ABAP Programming) Basics
- 6.4 When to Use BTP ABAP vs. Classic ABAP
- 6.5 Migration Path: Moving Code from On-Prem

### [Chapter 7: Fiori & UI5 in BTP](chapters/07-fiori-ui5-btp.md)
- 7.1 SAP Business Application Studio (BAS)
- 7.2 Deploying Fiori Apps to BTP
- 7.3 Connecting to Backend Systems
- 7.4 Differences from On-Prem Fiori Launchpad

---

## Part IV: Joule – AI Agents & Skills

### [Chapter 8: Joule Fundamentals](chapters/08-joule-fundamentals.md)
- 8.1 What Is Joule? (The SAP AI Copilot)
- 8.2 Skills vs. Agents: The Key Difference
- 8.3 How Skills "Do" Things (The Action Flow)
- 8.4 Why Destinations Are Required for Skills

### [Chapter 9: Building Your First Joule Skill](chapters/09-first-joule-skill.md)
- 9.1 The End-to-End Flow
  - Step 1: Create a Destination
  - Step 2: Create an Action Project in SAP Build
  - Step 3: Create a Skill in Joule Studio
  - Step 4: Test and Deploy
- 9.2 Adding REST Endpoints as Skills
- 9.3 OpenAPI Specs vs. Manual Action Definitions

### [Chapter 10: Building Joule Agents](chapters/10-building-agents.md)
- 10.1 What Makes an Agent "Smart"
- 10.2 Assigning Skills to Agents
- 10.3 Multi-Step Reasoning and Chaining
- 10.4 Grounding Agents on Client Documents

### [Chapter 11: Agent Lifecycle & Deployment](chapters/11-agent-lifecycle.md)
- 11.1 Dev → Test → Prod: The Standard Lifecycle
- 11.2 Transferring Projects Across Tenants
- 11.3 The SAP Build Promote Feature
- 11.4 Manual + Git Version Control (The Consultant Favorite)
- 11.5 Environment-Specific Configuration (Destinations, Models)

---

## Part V: Multi-Client Reality – The Consultant's Guide

### [Chapter 12: Managing Multiple Clients on BTP](chapters/12-multi-client-management.md)
- 12.1 The Classic Trap: One Subaccount for Everything
- 12.2 Best Practice: One Subaccount Per Client/Project
- 12.3 Naming Conventions for Multi-Client Environments
- 12.4 Security & Access Management
- 12.5 Cost Allocation and Chargeback

### [Chapter 13: Cross-Customer Deployments](chapters/13-cross-customer-deployments.md)
- 13.1 Pattern 1: Template + Per-Client Recreation
- 13.2 Pattern 2: Multi-Tenant SaaS Mode
- 13.3 When to Use Which Pattern
- 13.4 Comparison Table: Effort, Isolation, Handover

### [Chapter 14: Building C-Level Agents for Enterprise Clients](chapters/14-c-level-agents.md)
- 14.1 What Makes a C-Level Agent
- 14.2 Cross-System Orchestration
- 14.3 Managing AI Unit Consumption
- 14.4 Practical Tips for Executive Dashboards

---

## Part VI: Integration & Advanced Topics

### [Chapter 15: SAP Integration Suite Basics](chapters/15-integration-suite.md)
- 15.1 When to Use Integration Suite
- 15.2 APIs, iFlows, and Event Mesh
- 15.3 Connecting BTP to External Systems

### [Chapter 16: Cloud Connector for On-Premise Access](chapters/16-cloud-connector.md)
- 16.1 What Is Cloud Connector?
- 16.2 Setting Up for Legacy System Access
- 16.3 Common Scenarios: S/4 On-Prem + Joule in Cloud

### [Chapter 17: Clean Core Rules That Bite Old Habits](chapters/17-clean-core.md)
- 17.1 What Can't You Do Anymore?
- 17.2 Extensions vs. Modifications
- 17.3 Adapting Your Development Mindset

---

## Appendices

### [Appendix A: Quick Reference Tables](chapters/appendix-a-reference-tables.md)
- A.1 BTP Hierarchy Cheat Sheet
- A.2 Destination Configuration Options
- A.3 Joule Skill vs. Agent Comparison
- A.4 RISE vs. Classic Comparison

### [Appendix B: Glossary for Old ABAPers](chapters/appendix-b-glossary.md)
- Common BTP Terms Translated

### [Appendix C: Useful Links & Resources](chapters/appendix-c-resources.md)
- SAP Help Documentation
- SAP Community
- GitHub Samples

### [Appendix D: Troubleshooting Common Issues](chapters/appendix-d-troubleshooting.md)
- Destination Errors
- Joule Deployment Issues
- Authentication Problems

---

*[Back to README](README.md)*
