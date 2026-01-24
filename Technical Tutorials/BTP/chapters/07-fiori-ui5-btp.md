# Chapter 7: Fiori & UI5 in BTP

> *From WebIDE to Business Application Studio*

---

For Fiori developers, the cloud transition means new tools, new deployment targets, and new architectures. This chapter covers everything you need to build and deploy Fiori apps in BTP.

---

## 7.1 SAP Business Application Studio (BAS)

**SAP Business Application Studio (BAS)** is the cloud IDE that replaces:
- SAP Web IDE (deprecated)
- Eclipse for Fiori development
- Local VS Code setups (though you can still use those)

### What BAS Provides

| Feature | Description |
|---------|-------------|
| **Browser-based** | No local installation required |
| **Dev Spaces** | Pre-configured environments for different project types |
| **Full UI5 tooling** | Yeoman generators, Fiori tools, UI5 CLI |
| **Git integration** | Built-in version control |
| **Direct BTP deployment** | Deploy without leaving the IDE |
| **Extensions** | VS Code compatible extensions |

### Dev Space Types

When you create a Dev Space, choose the right type:

| Dev Space Type | Use For |
|---------------|---------|
| **SAP Fiori** | Fiori Elements, freestyle SAPUI5, extensions |
| **Full Stack Cloud Application** | CAP projects, Node.js, Java |
| **SAP HANA Native** | Native HANA development |
| **Basic** | General development |

### Setting Up BAS

1. **Access BAS**
   - BTP Cockpit → Subaccount → Service Marketplace
   - Find "SAP Business Application Studio"
   - Subscribe (usually included in BTP entitlements)
   - Click "Go to Application"

2. **Create a Dev Space**
   - Click "Create Dev Space"
   - Name: `fiori-development`
   - Type: "SAP Fiori"
   - Add extensions: "Fiori Tools", "CDS Tools", "Java Tools" (optional)
   - Start the Dev Space

3. **Wait for startup** (can take a few minutes the first time)

### BAS vs. WebIDE Comparison

| Aspect | Web IDE | BAS |
|--------|---------|-----|
| **Architecture** | Monolithic | Container-based Dev Spaces |
| **Performance** | Often slow | Faster, isolated |
| **Extensions** | Limited | VS Code compatible |
| **Future** | Deprecated | SAP's strategic direction |
| **Git** | Built-in | Full Git CLI + UI |

---

## 7.2 Creating a Fiori App from Scratch

### The Fiori Elements Path (Recommended)

Fiori Elements generates UI automatically from OData annotations—minimal JavaScript needed.

**Step 1: Open BAS Terminal**
```bash
# Navigate to projects folder
cd projects

# Create new Fiori app using generator
yo
```

**Step 2: Use the Fiori Application Generator**

Or use the graphical wizard:
1. File → New Project from Template
2. Choose "SAP Fiori Application"
3. Select template:
   - **List Report Page** — For table-based apps
   - **Worklist** — For task-based apps
   - **Object Page** — For detail views
   - **Overview Page** — For dashboards
   - **Form Entry Object Page** — For data entry

**Step 3: Configure Data Source**

```
Data source: Connect to an SAP System
System: ACME_S4_PROD (your destination)
Service: /sap/opu/odata/sap/API_SALES_ORDER_SRV
```

**Step 4: Configure Entity Selection**

```
Main entity: SalesOrder
Navigation entity: SalesOrderItem
Automatically add table columns: Yes
```

**Step 5: Project Attributes**

```
Module name: salesorder-display
Application title: Sales Order Display
Namespace: com.acme.salesorder
Add deployment configuration: Yes
Add FLP configuration: Yes
```

### Project Structure After Generation

```
salesorder-display/
├── webapp/
│   ├── Component.js              # App component
│   ├── manifest.json             # App descriptor (most important!)
│   ├── index.html                # Standalone entry point
│   ├── i18n/
│   │   └── i18n.properties       # Translations
│   ├── localService/             # Mock data for testing
│   │   ├── mockserver.js
│   │   └── metadata.xml
│   ├── test/                     # Test files
│   └── view/                     # Views (if freestyle)
├── package.json                  # NPM dependencies
├── ui5.yaml                      # UI5 tooling config
├── ui5-local.yaml                # Local config
├── xs-app.json                   # App Router config
└── mta.yaml                      # Multi-Target Application descriptor
```

---

## 7.3 The manifest.json Deep Dive

The `manifest.json` is the heart of every Fiori app. Let's understand its key sections:

### Data Source Configuration

```json
{
  "sap.app": {
    "id": "com.acme.salesorder",
    "type": "application",
    "applicationVersion": {
      "version": "1.0.0"
    },
    "title": "Sales Order Display",
    "dataSources": {
      "mainService": {
        "uri": "/sap/opu/odata/sap/API_SALES_ORDER_SRV/",
        "type": "OData",
        "settings": {
          "odataVersion": "2.0",
          "annotations": ["annotation0"],
          "localUri": "localService/metadata.xml"
        }
      },
      "annotation0": {
        "uri": "annotations/annotation0.xml",
        "type": "ODataAnnotation"
      }
    }
  }
}
```

### Model Configuration

```json
{
  "sap.ui5": {
    "models": {
      "": {
        "dataSource": "mainService",
        "preload": true,
        "settings": {
          "synchronizationMode": "None",
          "operationMode": "Server",
          "autoExpandSelect": true,
          "earlyRequests": true
        }
      },
      "i18n": {
        "type": "sap.ui.model.resource.ResourceModel",
        "settings": {
          "bundleName": "com.acme.salesorder.i18n.i18n"
        }
      }
    }
  }
}
```

### Routing Configuration

```json
{
  "sap.ui5": {
    "routing": {
      "routes": [
        {
          "pattern": ":?query:",
          "name": "SalesOrderList",
          "target": "SalesOrderList"
        },
        {
          "pattern": "SalesOrder({key}):?query:",
          "name": "SalesOrderObjectPage",
          "target": "SalesOrderObjectPage"
        }
      ],
      "targets": {
        "SalesOrderList": {
          "type": "Component",
          "id": "SalesOrderList",
          "name": "sap.fe.templates.ListReport",
          "options": {
            "settings": {
              "entitySet": "SalesOrder"
            }
          }
        },
        "SalesOrderObjectPage": {
          "type": "Component",
          "id": "SalesOrderObjectPage",
          "name": "sap.fe.templates.ObjectPage",
          "options": {
            "settings": {
              "entitySet": "SalesOrder"
            }
          }
        }
      }
    }
  }
}
```

### Cloud Configuration

```json
{
  "sap.cloud": {
    "public": true,
    "service": "basic.service"
  }
}
```

---

## 7.4 Connecting to Backend Systems

### Using Destinations in Fiori Apps

**The xs-app.json file routes requests through destinations:**

```json
{
  "welcomeFile": "/index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^/sap/opu/odata/sap/(.*)$",
      "target": "/sap/opu/odata/sap/$1",
      "destination": "ACME_S4_PROD",
      "authenticationType": "xsuaa",
      "csrfProtection": true
    },
    {
      "source": "^(.*)$",
      "target": "$1",
      "service": "html5-apps-repo-rt",
      "authenticationType": "xsuaa"
    }
  ]
}
```

### How Request Flow Works

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Browser                             │
│                                                                   │
│  Fiori App requests: /sap/opu/odata/sap/API_SALES_ORDER_SRV    │
└─────────────────────────────┬─────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Application Router                          │
│                                                                   │
│  1. Receives request                                             │
│  2. Matches route in xs-app.json                                 │
│  3. Looks up destination: ACME_S4_PROD                          │
│  4. Adds authentication headers                                  │
│  5. Forwards to backend                                          │
└─────────────────────────────┬─────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Destination Service                           │
│                                                                   │
│  ACME_S4_PROD:                                                   │
│    URL: https://s4.acme.com:443                                  │
│    Auth: OAuth2ClientCredentials                                 │
└─────────────────────────────┬─────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       S/4HANA Backend                            │
│                                                                   │
│  /sap/opu/odata/sap/API_SALES_ORDER_SRV                         │
│  Returns OData response                                          │
└─────────────────────────────────────────────────────────────────┘
```

### Multiple Destinations in One App

For apps consuming multiple services:

```json
{
  "routes": [
    {
      "source": "^/s4/(.*)$",
      "target": "/$1",
      "destination": "ACME_S4_PROD",
      "authenticationType": "xsuaa"
    },
    {
      "source": "^/sf/(.*)$",
      "target": "/$1",
      "destination": "ACME_SF_PROD",
      "authenticationType": "xsuaa"
    }
  ]
}
```

---

## 7.5 Deploying Fiori Apps to BTP

### Understanding the MTA Deployment

**MTA = Multi-Target Application**

An MTA bundles multiple modules (UI app, app router, services) into one deployable unit.

### The mta.yaml File

```yaml
_schema-version: "3.2"
ID: com.acme.salesorder
version: 1.0.0

parameters:
  enable-parallel-deployments: true

build-parameters:
  before-all:
    - builder: custom
      commands:
        - npm ci
        - npm run build

modules:
  # The Fiori App
  - name: salesorder-display
    type: html5
    path: .
    build-parameters:
      build-result: dist
      builder: custom
      commands:
        - npm run build:cf
      supported-platforms: []

  # The App Router (handles authentication and routing)
  - name: salesorder-approuter
    type: approuter.nodejs
    path: approuter
    parameters:
      disk-quota: 256M
      memory: 256M
    requires:
      - name: salesorder-destination
      - name: salesorder-uaa
      - name: salesorder-html5-repo-runtime

  # Deployer module (deploys HTML5 content)
  - name: salesorder-deployer
    type: com.sap.application.content
    path: .
    requires:
      - name: salesorder-html5-repo-host
        parameters:
          content-target: true
    build-parameters:
      build-result: resources
      requires:
        - name: salesorder-display
          artifacts:
            - salesorder-display.zip
          target-path: resources/

resources:
  # HTML5 Application Repository
  - name: salesorder-html5-repo-host
    type: org.cloudfoundry.managed-service
    parameters:
      service: html5-apps-repo
      service-plan: app-host

  - name: salesorder-html5-repo-runtime
    type: org.cloudfoundry.managed-service
    parameters:
      service: html5-apps-repo
      service-plan: app-runtime

  # Destination Service
  - name: salesorder-destination
    type: org.cloudfoundry.managed-service
    parameters:
      service: destination
      service-plan: lite

  # XSUAA (Authentication)
  - name: salesorder-uaa
    type: org.cloudfoundry.managed-service
    parameters:
      service: xsuaa
      service-plan: application
      path: xs-security.json
```

### The xs-security.json File

Defines OAuth2 scopes and roles:

```json
{
  "xsappname": "salesorder-display",
  "tenant-mode": "dedicated",
  "scopes": [
    {
      "name": "$XSAPPNAME.Display",
      "description": "Display Sales Orders"
    },
    {
      "name": "$XSAPPNAME.Edit",
      "description": "Edit Sales Orders"
    }
  ],
  "role-templates": [
    {
      "name": "Viewer",
      "description": "View Sales Orders",
      "scope-references": ["$XSAPPNAME.Display"]
    },
    {
      "name": "Editor",
      "description": "Edit Sales Orders",
      "scope-references": ["$XSAPPNAME.Display", "$XSAPPNAME.Edit"]
    }
  ]
}
```

### Deployment Steps

**Step 1: Build the MTA**

```bash
# In BAS terminal
mbt build

# Creates mta_archives/com.acme.salesorder_1.0.0.mtar
```

**Step 2: Deploy to Cloud Foundry**

```bash
# Login to CF
cf login -a https://api.cf.eu10.hana.ondemand.com

# Deploy the MTA
cf deploy mta_archives/com.acme.salesorder_1.0.0.mtar
```

**Step 3: Verify Deployment**

```bash
# Check apps
cf apps

# Check services
cf services

# Get app URL
cf app salesorder-approuter
```

---

## 7.6 SAP Build Work Zone (The New Launchpad)

Instead of on-prem Fiori Launchpad, BTP uses **SAP Build Work Zone** (formerly SAP Launchpad Service).

### Standard vs. Advanced Edition

| Feature | Standard | Advanced |
|---------|----------|----------|
| **Fiori Launchpad** | ✅ Yes | ✅ Yes |
| **Central Entry Point** | ✅ Yes | ✅ Yes |
| **Workpages** | ❌ No | ✅ Yes |
| **Workspaces** | ❌ No | ✅ Yes |
| **Content Federation** | Limited | Full |
| **Cost** | Lower | Higher |

### Adding Your App to Work Zone

1. **Create Site in Work Zone**
   - BTP Cockpit → Instances → Work Zone
   - Create Site → Name: "ACME Portal"

2. **Register Your App**
   - Content Manager → Content Explorer
   - Find your app (auto-discovered from HTML5 repo)
   - Add to "My Content"

3. **Create Role and Assign**
   - Create Role: "Sales Team"
   - Add app to role
   - Assign role to users/groups

4. **Add to Group and Catalog**
   - Create Catalog: "Sales Applications"
   - Create Group: "Sales Orders"
   - Add app to both

5. **Access the Launchpad**
   - Site Directory → Your Site → Go to Site
   - App appears as tile

### FLP Configuration in manifest.json

```json
{
  "sap.app": {
    "crossNavigation": {
      "inbounds": {
        "salesorder-display": {
          "semanticObject": "SalesOrder",
          "action": "display",
          "title": "Display Sales Orders",
          "signature": {
            "parameters": {},
            "additionalParameters": "allowed"
          }
        }
      }
    }
  }
}
```

---

## 7.7 Local Development and Testing

### Running Locally with UI5 Tooling

**Start local server:**
```bash
# Simple start
npm start

# Or with specific config
ui5 serve --open index.html
```

**With mock data:**
```bash
npm run start-local
```

### Using Cds-plugin-ui5 for CAP Projects

If your backend is CAP:

```bash
# Start CAP + UI5 together
cds watch
```

### Testing with Different Environments

**ui5-local.yaml for local development:**
```yaml
specVersion: "2.5"
metadata:
  name: salesorder-display
type: application
framework:
  name: SAPUI5
  version: "1.120.0"
  libraries:
    - name: sap.m
    - name: sap.ui.core
    - name: sap.fe.templates
server:
  customMiddleware:
    - name: fiori-tools-proxy
      afterMiddleware: compression
      configuration:
        backend:
          - path: /sap
            url: http://localhost:8080  # Local mock or CAP
```

### OPA5 Testing

Fiori Elements comes with test templates:

```javascript
// test/integration/opaTests.qunit.js
sap.ui.define([
  "sap/ui/test/Opa5",
  "sap/ui/test/opaQunit",
  "./pages/SalesOrderList"
], function(Opa5, opaQunit, SalesOrderList) {
  "use strict";

  Opa5.extendConfig({
    viewNamespace: "com.acme.salesorder.view"
  });

  QUnit.module("Sales Order List");

  opaQunit("Should see the list with sales orders", function(Given, When, Then) {
    Given.iStartMyApp();
    Then.onTheSalesOrderList.iSeeTheList();
    Then.iTeardownMyApp();
  });
});
```

---

## 7.8 Extending Standard Fiori Apps

### The Extension Project Approach

For extending standard SAP Fiori apps:

1. **Find the original app** in SAP Fiori Apps Reference Library
2. **Create extension project** in BAS
3. **Use SAPUI5 flexibility** to modify

### Creating an Extension Project

```bash
# Using the generator
yo @sap/fiori:extend
```

Or via BAS:
1. File → New Project from Template
2. Choose "Adaptation Project"
3. Select base application
4. Add your modifications

### Types of Extensions

| Extension Type | What It Does |
|----------------|--------------|
| **Controller Extension** | Add custom logic |
| **View Extension** | Add UI elements |
| **Fragment** | Insert new sections |
| **i18n Override** | Change texts |
| **Annotation Modification** | Change UI annotations |

### Example: Adding a Custom Button

**manifest.json extension:**
```json
{
  "sap.ui5": {
    "extends": {
      "extensions": {
        "sap.ui.controllerExtensions": {
          "sap.fe.templates.ObjectPage.ObjectPageController": {
            "controllerName": "com.acme.extension.ObjectPageExt"
          }
        }
      }
    }
  }
}
```

**ObjectPageExt.controller.js:**
```javascript
sap.ui.define([
  "sap/ui/core/mvc/ControllerExtension"
], function(ControllerExtension) {
  "use strict";

  return ControllerExtension.extend("com.acme.extension.ObjectPageExt", {
    onInit: function() {
      // Extension initialization
    },

    onCustomAction: function() {
      sap.m.MessageToast.show("Custom action triggered!");
    }
  });
});
```

---

## 7.9 Differences from On-Prem Fiori Development

### Summary Comparison

| Aspect | On-Prem Fiori | BTP Fiori |
|--------|---------------|-----------|
| **IDE** | Web IDE / Eclipse | BAS |
| **Repository** | ABAP (/UI5/...) | HTML5 App Repository |
| **Gateway** | SAP Gateway (SICF) | Destination + App Router |
| **Launchpad** | Fiori Launchpad (FLP) | SAP Build Work Zone |
| **Deployment** | Transport / UI5 Deploy | MTA Deploy (cf deploy) |
| **Authentication** | SAP Logon | XSUAA / IAS |
| **Transport** | CTS | Git / CI/CD |

### Mental Model Shift

**On-Prem thinking:**
> "Deploy to the ABAP repository, register in SICF, configure in FLP"

**BTP thinking:**
> "Build MTA, deploy to Cloud Foundry, app auto-registers in Work Zone"

---

## 7.10 Best Practices for BTP Fiori Development

### 1. Use Fiori Elements When Possible

- 80% of apps can be Fiori Elements
- Automatically responsive
- Built-in accessibility
- Consistent UX

### 2. Leverage Annotations

OData annotations drive the UI:

```xml
<!-- annotations.xml -->
<Annotations Target="API_SALES_ORDER_SRV.SalesOrder">
  <Annotation Term="UI.LineItem">
    <Collection>
      <Record Type="UI.DataField">
        <PropertyValue Property="Value" Path="SalesOrder"/>
        <PropertyValue Property="Label" String="Sales Order"/>
      </Record>
      <Record Type="UI.DataField">
        <PropertyValue Property="Value" Path="SoldToParty"/>
        <PropertyValue Property="Label" String="Customer"/>
      </Record>
    </Collection>
  </Annotation>
</Annotations>
```

### 3. Design Mobile-First

BTP apps are often accessed on mobile. Test on different screen sizes.

### 4. Use Proper Error Handling

```javascript
// Handle OData errors
this.getModel().read("/SalesOrderSet", {
  success: function(oData) {
    // Handle success
  },
  error: function(oError) {
    MessageBox.error("Failed to load data: " + oError.message);
  }
});
```

### 5. Implement Proper Security

- Define scopes in xs-security.json
- Check authorizations in app
- Use role-based access

---

## Key Takeaways

1. **BAS is the future** — WebIDE is deprecated, learn BAS now
2. **Fiori Elements is preferred** — Auto-generated UI from annotations
3. **MTA deployment** — Multi-Target Applications bundle everything
4. **Work Zone replaces FLP** — Cloud-native launchpad
5. **Destinations are critical** — Secure backend connections
6. **xs-app.json routes requests** — Through App Router to destinations
7. **Extensions still work** — SAPUI5 flexibility for standard apps

---

## What's Next?

You now understand Fiori development in BTP. Next, let's dive into the exciting world of AI—Joule fundamentals and how to build intelligent skills and agents.

---

*[Previous: Chapter 6 – ABAP in the Cloud](06-abap-cloud.md) | [Next: Chapter 8 – Joule Fundamentals](08-joule-fundamentals.md)*

*[Back to Table of Contents](../content.md)*

---

**Author:** [Beyhan Meyrali](https://www.linkedin.com/in/beyhanmeyrali) — SAP Storyteller & Digital Transformation Advocate

*Created with ❤️ for SAP learners worldwide*
