# Chapter 15: SAP Integration Suite Basics

> *Connecting Everything to Everything*

---

## 15.1 When to Use Integration Suite

Use Integration Suite when you need:
- Complex data transformations
- Message routing
- Protocol conversion (SOAP ↔ REST)
- Event-driven integration
- B2B/EDI scenarios

For simple REST calls, destinations may be enough.

---

## 15.2 APIs, iFlows, and Event Mesh

### API Management
- Publish APIs
- Rate limiting
- Analytics

### Integration Flows (iFlows)
- Visual message flow design
- Transformations, mappings
- Error handling

### Event Mesh
- Publish/subscribe patterns
- Asynchronous integration
- Event-driven architecture

---

## 15.3 Connecting BTP to External Systems

Common patterns:
1. **S/4 → BTP → External**: Use iFlow for transformation
2. **External → BTP → S/4**: Event Mesh for async
3. **Multiple sources → Aggregation**: Composite API

---

*[Previous: Chapter 14](14-c-level-agents.md) | [Next: Chapter 16](16-cloud-connector.md)*

*[Back to Table of Contents](../content.md)*
