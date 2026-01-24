# Appendix D: Troubleshooting Common Issues

> *When Things Don't Work*

---

## Destination Errors

### "Connection refused" or "Host not found"

**Cause**: URL is wrong or system is down

**Fix**:
1. Verify URL is correct
2. Check if target system is running
3. For OnPremise: verify Cloud Connector is running

### "401 Unauthorized"

**Cause**: Authentication failed

**Fix**:
1. Check username/password
2. For OAuth: verify client ID/secret
3. Check token URL is correct
4. Verify user has required permissions

### "403 Forbidden"

**Cause**: Authenticated but not authorized

**Fix**:
1. Check user roles on target system
2. Verify path is exposed (Cloud Connector)
3. Check if IP is whitelisted

### "Certificate error"

**Cause**: SSL/TLS issue

**Fix**:
1. Check if certificate is valid
2. For dev: add `TrustAll=true` (not for prod!)
3. Import certificate to trust store

---

## Joule Deployment Issues

### Skill not appearing after deploy

**Possible causes**:
- Deployment still in progress
- Skill not enabled for Joule
- Wrong subaccount

**Fix**:
1. Wait a few minutes
2. Check deployment status in Joule Studio
3. Verify subaccount is correct

### "Action not found" error

**Cause**: Action Project not linked properly

**Fix**:
1. Verify Action Project is deployed
2. Check destination is configured
3. Re-link action in skill

### Agent doesn't use the skill

**Cause**: Skill not assigned or instructions unclear

**Fix**:
1. Verify skill is in agent's skill list
2. Clear up agent instructions
3. Test skill individually first

---

## Authentication Problems

### OAuth token not working

**Cause**: Token expired or misconfigured

**Fix**:
1. Check token URL
2. Verify client ID and secret
3. Check scope if required
4. Look for clock sync issues

### SSO not working

**Cause**: Trust configuration issue

**Fix**:
1. Check IAS/IdP configuration
2. Verify trust relationship
3. Check user mapping

---

## Cloud Connector Issues

### Connection shows "unreachable"

**Cause**: Cloud Connector not connected

**Fix**:
1. Check if CC service is running
2. Verify subaccount connection in CC admin
3. Check firewall rules

### "No route to host" for on-prem

**Cause**: Virtual mapping issue

**Fix**:
1. Verify virtual host mapping
2. Check path exposure
3. Test internal connectivity

---

## Performance Issues

### Slow Joule responses

**Possible causes**:
- AI model overloaded
- Complex grounding
- Slow backend API

**Fix**:
1. Simplify grounding documents
2. Optimize backend API
3. Check AI Core status

### Destination timeout

**Cause**: Target system too slow

**Fix**:
1. Increase timeout in destination
2. Optimize target API
3. Check network latency

---

## General Tips

1. **Check BTP Cockpit logs** — Most errors appear there
2. **Use "Check Connection"** — Test destinations before use
3. **Start simple** — Test components individually
4. **Read error messages** — They're usually helpful
5. **Check SAP Notes** — For known issues and fixes

---

*[Back to Table of Contents](../content.md)*
