# Chapter 7: Fiori & UI5 in BTP

> *From WebIDE to Business Application Studio*

---

## 7.1 SAP Business Application Studio (BAS)

BAS is the cloud IDE that replaces:
- SAP WebIDE
- Eclipse for Fiori development

It runs in your browser and provides:
- Dev Spaces (pre-configured environments)
- Full UI5 tooling
- Integration with Git
- Direct deployment to BTP

---

## 7.2 Deploying Fiori Apps to BTP

### The Deployment Flow

1. **Develop** in BAS
2. **Build** with `npm run build`
3. **Deploy** to HTML5 Application Repository
4. **Expose** via SAP Build Work Zone

### MTA Deployment

```yaml
# mta.yaml
modules:
  - name: my-fiori-app
    type: html5
    path: .
    build-parameters:
      build-result: dist
```

---

## 7.3 Connecting to Backend Systems

Use **destinations** to connect your Fiori app to OData services:

```javascript
// manifest.json
"dataSources": {
  "mainService": {
    "uri": "/sap/opu/odata/sap/API_SALES_ORDER_SRV/",
    "type": "OData",
    "settings": {
      "odataVersion": "2.0"
    }
  }
}
```

The destination handles the actual URL and authentication.

---

## 7.4 Differences from On-Prem Fiori Launchpad

| On-Prem FLP | BTP (Work Zone) |
|-------------|-----------------|
| ABAP repository (/UI5/...) | HTML5 App Repository |
| SICF services | Cloud routing |
| SAP Gateway | Destination |
| Local deployment | MTA deployment |

---

*[Previous: Chapter 6](06-abap-cloud.md) | [Next: Chapter 8](08-joule-fundamentals.md)*

*[Back to Table of Contents](../content.md)*
