# Chapter 9: Building Your First Joule Skill

> *Step-by-Step Walkthrough*

---

## 9.1 The End-to-End Flow

Here's the complete path to create a working Joule skill:

### Step 1: Create a Destination

In BTP Cockpit → Connectivity → Destinations:
```
Name:           MY_ORDER_API
Type:           HTTP
URL:            https://api.example.com/orders
Proxy Type:     Internet
Authentication: OAuth2ClientCredentials
```

### Step 2: Create an Action Project in SAP Build

1. Go to SAP Build lobby
2. Create new **Action Project**
3. Import OpenAPI spec or define actions manually
4. Select the destination you created
5. This creates a safe, governed "API wrapper"

### Step 3: Create a Skill in Joule Studio

1. Go to Joule Studio
2. Create new **Skill**
3. Add an **Action** to the skill
4. Point it to your Action Project
5. Define parameters and responses

### Step 4: Test and Deploy

1. Test the skill with sample inputs
2. Deploy to make it available
3. Assign to an Agent if needed

---

## 9.2 Adding REST Endpoints as Skills

For external REST APIs:
1. Get the OpenAPI/Swagger spec
2. Create destination pointing to the API
3. Import spec in Action Project
4. Create skill wrapping the action

---

## 9.3 OpenAPI Specs vs. Manual Action Definitions

| Method | When to Use |
|--------|-------------|
| OpenAPI Import | API has good documentation |
| Manual Definition | Simple API, no spec available |

Manual actions require defining:
- Endpoint path
- HTTP method
- Request/response schema
- Parameters

---

*[Previous: Chapter 8](08-joule-fundamentals.md) | [Next: Chapter 10](10-building-agents.md)*

*[Back to Table of Contents](../content.md)*
