# Chapter 5: Destinations – The Phone Book for Connections

> *The Most Important BTP Concept for ABAP Developers*

---

If you remember one thing from this entire book, make it destinations. Every Fiori app, every Joule skill, every integration you build in BTP will use destinations. They're the foundation of all connectivity in the cloud world.

---

## 5.1 What Is a Destination?

A **Destination** in BTP is a **named, centrally managed connection configuration**.

> **The Phone Book Analogy**
>
> Think of your smartphone contacts:
> - You don't type Mom's phone number every time you call
> - You just tap "Call Mom"
> - Behind the scenes: +90 212 555 1234
>
> In BTP, you reference `MY_S4_SYSTEM` instead of hardcoding URLs, usernames, and passwords everywhere.

### The Old Way vs. The New Way

**Old Way (Hard-coded in programs):**
```abap
" Classic ABAP - DON'T DO THIS
DATA: lv_url TYPE string VALUE 'https://api.acme.com:443/orders'.
DATA: lv_user TYPE string VALUE 'API_USER'.
DATA: lv_pass TYPE string VALUE 'SecretPassword123!'.

" Now you have passwords in code - nightmare!
```

**New Way (BTP Destinations):**
```javascript
// Your app just references the destination name
const destination = "ACME_ORDER_API";
// BTP handles all the messy details
```

### What's Stored in a Destination

| Property | Description | Example |
|----------|-------------|---------|
| **Name** | Unique identifier | `ACME_S4_PROD_ORDERS` |
| **Type** | Protocol type | HTTP, RFC, LDAP, MAIL |
| **URL** | Target endpoint | `https://s4.acme.com:443/sap/opu/odata/...` |
| **Proxy Type** | How to reach it | Internet, OnPremise, PrivateLink |
| **Authentication** | How to authenticate | Basic, OAuth2, Certificate |
| **Credentials** | The actual secrets | User/Password, Client ID/Secret |
| **Additional Properties** | Extra settings | `sap-client=100`, timeouts |

---

## 5.2 Why Destinations Exist (Security & Governance)

### The Problems They Solve

| Problem | Without Destinations | With Destinations |
|---------|---------------------|-------------------|
| **Password in code** | Security nightmare, audit failure | Credentials stored centrally, encrypted |
| **URL changes** | Redeploy all apps | Update one destination |
| **Password rotation** | Find and update everywhere | Change once, apps auto-use new creds |
| **Environment management** | Hard-coded dev/prod URLs | Different destinations per subaccount |
| **Audit trail** | No visibility | See who uses what, when |
| **Access control** | Anyone can hardcode | Destinations have access policies |

### The SM59 Comparison

For ABAP developers, destinations are conceptually similar to SM59 RFC destinations, but for HTTP/REST:

| SM59 (Classic ABAP) | BTP Destinations |
|---------------------|------------------|
| RFC connections | HTTP, RFC, LDAP, MAIL |
| Stored in SAP system | Stored in BTP subaccount |
| Transaction-based config | Cockpit or API config |
| Limited auth options | Modern auth (OAuth2, SAML, etc.) |

---

## 5.3 Creating Your First Destination

### Step-by-Step in BTP Cockpit

1. **Navigate to your subaccount**
   - BTP Cockpit → Your Global Account → Your Subaccount

2. **Go to Connectivity → Destinations**
   - Left menu → Connectivity → Destinations

3. **Click "New Destination"**

4. **Fill in the basic properties:**

```
Name:           ACME_S4_PROD_ORDERS
Type:           HTTP
Description:    S/4HANA Production - Sales Orders API
URL:            https://s4-prod.acme.com:443/sap/opu/odata/sap/API_SALES_ORDER_SRV
Proxy Type:     Internet
Authentication: OAuth2ClientCredentials
```

5. **Add authentication details** (for OAuth2):

```
Client ID:      your-client-id-from-s4
Client Secret:  your-client-secret
Token Service URL: https://s4-prod.acme.com/sap/bc/sec/oauth2/token
```

6. **Add additional properties** (click "New Property"):

```
sap-client:                 100
HTML5.DynamicDestination:   true
WebIDEEnabled:              true
WebIDEUsage:                odata_abap
```

7. **Save and test with "Check Connection"**

### Using the btp CLI

For automation and DevOps:

```bash
# List destinations
btp list services/destination --subaccount <subaccount-id>

# Create destination from JSON file
btp create services/destination --subaccount <subaccount-id> \
  --data @my-destination.json
```

**my-destination.json:**
```json
{
  "Name": "ACME_S4_PROD_ORDERS",
  "Type": "HTTP",
  "URL": "https://s4-prod.acme.com:443/sap/opu/odata/sap/API_SALES_ORDER_SRV",
  "ProxyType": "Internet",
  "Authentication": "OAuth2ClientCredentials",
  "tokenServiceURL": "https://s4-prod.acme.com/sap/bc/sec/oauth2/token",
  "clientId": "your-client-id",
  "clientSecret": "your-client-secret",
  "sap-client": "100"
}
```

---

## 5.4 Authentication Types Explained

This is where ABAP developers often get confused. Let's break down each type:

### 1. NoAuthentication

**Use for:** Public APIs only (rare in enterprise)

```
Authentication: NoAuthentication
```

**Example:** A public weather API, a test mock server

**Warning:** Never use for internal APIs!

---

### 2. BasicAuthentication

**Use for:** Simple APIs, legacy systems, development/testing

```
Authentication: BasicAuthentication
User:           API_USER
Password:       ********
```

**Comparison to ABAP:**
```abap
" Like classic RFC destination with user/password
" SM59 → Connection Test → Logon & Security
```

**When to use:**
- Quick setup for testing
- Legacy systems that don't support OAuth
- Internal APIs in trusted networks

**When NOT to use:**
- Production with internet-exposed APIs
- When you need user context propagation

---

### 3. OAuth2ClientCredentials

**Use for:** Modern server-to-server communication (most common!)

```
Authentication: OAuth2ClientCredentials
Client ID:      abc123-client-id
Client Secret:  ********
Token Service URL: https://target.com/oauth/token
```

**How it works:**
```
1. Your app sends Client ID + Secret to Token URL
2. Gets back an access token (valid for X minutes)
3. Uses token to call the actual API
4. Token expires → automatically gets new one
```

**This is the recommended authentication for:**
- S/4HANA Cloud APIs
- SuccessFactors APIs
- Most modern SAP services
- Joule skills calling external APIs

---

### 4. OAuth2SAMLBearerAssertion

**Use for:** SAP-to-SAP with user context propagation

```
Authentication: OAuth2SAMLBearerAssertion
Client ID:      abc123-client-id
Client Secret:  ********
Token Service URL: https://s4.acme.com/sap/bc/sec/oauth2/token
```

**The difference from ClientCredentials:**
- **ClientCredentials**: Technical user, no real user context
- **SAMLBearer**: Logged-in user's identity flows to backend

**When to use:**
- User needs specific authorizations on backend
- Audit logs should show real user
- Data access depends on user roles

**Example scenario:**
> A Fiori app shows sales orders. Using SAMLBearer, the S/4 backend returns only orders the logged-in user is authorized to see.

---

### 5. ClientCertificateAuthentication

**Use for:** High-security scenarios, mutual TLS

```
Authentication: ClientCertificateAuthentication
Key Store Location: /path/to/keystore
Key Store Password: ********
```

**When to use:**
- Banking/financial services
- Government systems
- When passwords aren't allowed
- Machine-to-machine with PKI

---

### 6. PrincipalPropagation

**Use for:** User identity forwarding through Cloud Connector to on-premise

```
Authentication: PrincipalPropagation
```

**How it works:**
```
BTP User → Cloud Connector → On-Prem SAP
           (User identity preserved)
```

**Requirements:**
- Cloud Connector configured with trust
- On-prem system trusts Cloud Connector
- User exists in both systems (or mapping configured)

---

### Authentication Quick Reference

| Type | User Context | Best For | Complexity |
|------|--------------|----------|------------|
| NoAuthentication | None | Public APIs | Trivial |
| BasicAuthentication | Technical | Legacy, Dev | Easy |
| OAuth2ClientCredentials | Technical | Server-to-server | Medium |
| OAuth2SAMLBearerAssertion | Real user | User-specific data | Medium |
| ClientCertificateAuthentication | Certificate | High security | Complex |
| PrincipalPropagation | Real user | On-prem with SSO | Complex |

---

## 5.5 On-Premise Access via Cloud Connector

When your target system is **behind a firewall** (not internet-accessible), you need Cloud Connector.

### The Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        INTERNET                               │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   ┌───────────────┐                                          │
│   │    BTP        │                                          │
│   │  Subaccount   │                                          │
│   │               │                                          │
│   │  Destination: │                                          │
│   │  ProxyType:   │                                          │
│   │  OnPremise    │                                          │
│   └───────┬───────┘                                          │
│           │                                                   │
│           │  Secure Tunnel (outbound from your network)      │
│           │  (No inbound firewall rules needed!)             │
│           ▼                                                   │
├──────────────────────────────────────────────────────────────┤
│                    YOUR CORPORATE NETWORK                     │
│                                                               │
│   ┌───────────────┐         ┌───────────────┐               │
│   │    Cloud      │         │    S/4HANA    │               │
│   │  Connector    │────────►│   On-Prem     │               │
│   │  (Installed)  │         │   10.0.0.50   │               │
│   └───────────────┘         └───────────────┘               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Setting Up the Destination for On-Premise

```
Name:           ACME_S4_ONPREM_ORDERS
Type:           HTTP
URL:            http://s4-virtual:443/sap/opu/odata/sap/API_SALES_ORDER_SRV
Proxy Type:     OnPremise     ← Key difference!
Authentication: PrincipalPropagation  (or BasicAuthentication)
Location ID:    (optional, if you have multiple Cloud Connectors)
```

**Important notes:**
- The URL uses the **virtual host** configured in Cloud Connector
- **Not** the real internal IP address
- Cloud Connector maps virtual → real host

### Virtual Host Mapping (in Cloud Connector Admin)

```
Virtual Host:     s4-virtual      (what BTP sees)
Virtual Port:     443
Internal Host:    10.0.0.50       (actual server)
Internal Port:    8000
```

### Exposed Paths (Security!)

Only expose what you need:

```
Allowed paths in Cloud Connector:
  /sap/opu/odata/sap/API_SALES_ORDER_SRV     ← Only this OData service
  /sap/bc/adt/*                               ← For ABAP Dev Tools

NOT exposed:
  /sap/bc/gui/*                               ← Keep SAP GUI out
  /                                           ← Never expose root!
```

---

## 5.6 Naming Conventions That Save Your Sanity

After working with dozens of clients, trust me: naming matters.

### The Pattern

```
{CLIENT}_{SYSTEM}_{ENVIRONMENT}_{PURPOSE}
```

### Examples

```
# Production destinations
ACME_S4_PROD_ORDERS
ACME_S4_PROD_MATERIALS
ACME_SF_PROD_EMPLOYEES
ACME_ARIBA_PROD_REQUISITIONS

# Development destinations
ACME_S4_DEV_ORDERS
ACME_S4_DEV_MATERIALS

# Test/QA destinations
ACME_S4_TEST_ORDERS

# Sandbox/mock destinations
SANDBOX_MOCK_ORDERS
```

### Why This Pattern Works

1. **Sorted together**: All ACME destinations group in cockpit
2. **Environment visible**: No accidentally calling prod from dev app
3. **Purpose clear**: Know what API this destination accesses
4. **Searchable**: Easy to find in large subaccounts

### Additional Properties Naming

For additional properties with specific environments:

```
# Good - clear purpose
HTML5.DynamicDestination: true
WebIDEEnabled: true
sap-client: 100

# Add comments via Description field
Description: "ACME Production S/4 - Sales Orders OData. Contact: john@acme.com"
```

---

## 5.7 Destinations in Fiori Apps

### How Fiori Apps Use Destinations

In your Fiori app's `manifest.json`:

```json
{
  "sap.app": {
    "dataSources": {
      "mainService": {
        "uri": "/sap/opu/odata/sap/API_SALES_ORDER_SRV/",
        "type": "OData",
        "settings": {
          "odataVersion": "2.0"
        }
      }
    }
  },
  "sap.cloud": {
    "public": true,
    "service": "basic.service"
  }
}
```

### The xs-app.json Route

The magic happens in `xs-app.json` (Application Router config):

```json
{
  "routes": [
    {
      "source": "^/sap/opu/odata/sap/(.*)$",
      "target": "/sap/opu/odata/sap/$1",
      "destination": "ACME_S4_PROD_ORDERS",
      "authenticationType": "xsuaa"
    }
  ]
}
```

**What this does:**
1. App requests `/sap/opu/odata/sap/API_SALES_ORDER_SRV/...`
2. App Router intercepts
3. Looks up `ACME_S4_PROD_ORDERS` destination
4. Forwards request with proper auth to backend
5. Returns response to app

### MTA Destination Binding

In `mta.yaml`:

```yaml
modules:
  - name: my-fiori-app
    type: approuter.nodejs
    requires:
      - name: my-destination-service
      - name: my-xsuaa

resources:
  - name: my-destination-service
    type: org.cloudfoundry.managed-service
    parameters:
      service: destination
      service-plan: lite
```

---

## 5.8 Destinations in Joule Skills

For Joule skills, destinations are **mandatory**. You cannot hardcode URLs.

### The Flow

```
Joule Agent → Skill → Action (from Action Project) → Destination → API
```

### Setting Up a Destination for Joule

Same as any destination, but ensure these properties:

```
Name:           JOULE_ACME_S4_ORDERS
Type:           HTTP
URL:            https://s4.acme.com/sap/opu/odata/sap/API_SALES_ORDER_SRV
Authentication: OAuth2ClientCredentials
Client ID:      ...
Client Secret:  ...
Token URL:      ...

# Recommended additional properties
HTML5.DynamicDestination: true
```

### Referencing in SAP Build Actions

When you create an Action Project in SAP Build:
1. Create the destination first
2. In Action Project, select "Add API"
3. Choose "Use Destination"
4. Select your destination
5. Import OpenAPI spec or define actions

The skill never sees credentials—it just references the Action Project, which references the destination.

---

## 5.9 Common Destination Patterns

### Pattern 1: Same API, Multiple Environments

```
# Three destinations, same app, different configs
ACME_S4_DEV_ORDERS   → https://s4-dev.acme.com/...
ACME_S4_TEST_ORDERS  → https://s4-test.acme.com/...
ACME_S4_PROD_ORDERS  → https://s4-prod.acme.com/...
```

Your app code stays the same. Just deploy to different subaccounts with matching destinations.

### Pattern 2: Multiple Clients (Consultant)

```
# Subaccount: CLIENT_A_PROD
CLIENTA_S4_PROD_ORDERS  → https://s4.clienta.com/...

# Subaccount: CLIENT_B_PROD
CLIENTB_S4_PROD_ORDERS  → https://s4.clientb.com/...
```

Or use a consistent name if your app is templated:

```
# In every client's subaccount, same destination name
MY_S4_ORDERS  → (client-specific URL)
```

### Pattern 3: SAP SuccessFactors Integration

```
Name:           ACME_SF_PROD_EMPLOYEES
Type:           HTTP
URL:            https://api15.sapsf.com/odata/v2
Authentication: OAuth2SAMLBearerAssertion
Client ID:      your-sf-api-key
Client Secret:  your-sf-secret
Token URL:      https://api15.sapsf.com/oauth/token
company:        yourCompanyId
```

### Pattern 4: External Non-SAP API

```
Name:           EXTERNAL_WEATHER_API
Type:           HTTP
URL:            https://api.weatherapi.com/v1
Authentication: NoAuthentication  (or API key via header)

# Add API key as additional property if needed
URL.headers.x-api-key: your-api-key-here
```

---

## 5.10 Troubleshooting Destinations

### "Check Connection" Button

Always use it! It tests:
- URL reachability
- Authentication
- SSL/TLS certificate
- Cloud Connector connectivity (for OnPremise)

### Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| Connection refused | URL wrong or system down | Verify URL, check system status |
| 401 Unauthorized | Bad credentials | Verify user/password or OAuth tokens |
| 403 Forbidden | Authenticated but no access | Check user authorizations |
| Certificate error | SSL issue | Add cert to trust store or use TrustAll (dev only) |
| Timeout | Slow target or network | Increase timeout property |
| Host not found | DNS issue or wrong virtual host | Check Cloud Connector mapping |

### Debugging OAuth2

If OAuth2 isn't working:

1. **Test token URL separately** (using Postman/Bruno):
```bash
curl -X POST https://target.com/oauth/token \
  -u "client-id:client-secret" \
  -d "grant_type=client_credentials"
```

2. **Check the response**: Should return `access_token`

3. **Common OAuth2 issues:**
   - Wrong token URL
   - Client ID/Secret mismatch
   - Missing required scope
   - Clock sync issues (token validation fails)

---

## 5.11 Destination Best Practices

### Do's

- ✅ Use descriptive names with consistent pattern
- ✅ Document purpose in Description field
- ✅ Use OAuth2 for production
- ✅ Test with "Check Connection" before use
- ✅ Set appropriate timeouts
- ✅ Use separate destinations per environment
- ✅ Review destinations during security audits

### Don'ts

- ❌ Never hardcode credentials in apps
- ❌ Don't use BasicAuth for internet-exposed prod APIs
- ❌ Don't share destinations across unrelated apps
- ❌ Don't forget to rotate credentials periodically
- ❌ Don't use TrustAll=true in production
- ❌ Don't expose unnecessary paths in Cloud Connector

---

## Key Takeaways

1. **Destinations are the foundation** — Every BTP connection uses them
2. **Like SM59 but better** — HTTP, modern auth, cloud-native
3. **Credentials stay secure** — Never in code, always in destination
4. **Environment separation** — Different destinations for dev/test/prod
5. **Naming matters** — Use `{CLIENT}_{SYSTEM}_{ENV}_{PURPOSE}`
6. **OAuth2 is preferred** — For production, always
7. **Cloud Connector** — For on-premise access with ProxyType=OnPremise

---

## What's Next?

You now understand destinations—the bridge to all your backends. Next, let's see how to write ABAP code in the cloud with the BTP ABAP Environment.

---

*[Previous: Chapter 4 – How RISE and BTP Fit Together](04-rise-and-btp.md) | [Next: Chapter 6 – ABAP in the Cloud](06-abap-cloud.md)*

*[Back to Table of Contents](../content.md)*

---

**Author:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Created with ❤️ for SAP learners worldwide*
