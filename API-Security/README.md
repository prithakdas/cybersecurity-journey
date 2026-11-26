# API Security — Learning Notes & Testing Checklist

> **Status:** Study/reference notes, not a record of completed testing. Hands-on practice should be logged separately after work in an authorized lab.

## Learning objectives

- Understand HTTP requests, responses, headers, JSON, authentication, and API schemas.
- Map endpoints and identify roles and data objects.
- Test authorization separately from authentication.
- Practice with Postman and Burp Suite.
- Recognize and report the OWASP API Security Top 10 (2023).

## 1. Request anatomy

A REST request typically contains a method, path, headers, body, and response. Identify client-controlled values versus server-generated values and server-side checks.

```http
GET /api/v1/profile HTTP/1.1
Host: api.example.test
Accept: application/json
Authorization: Bearer <LAB_TOKEN>
```

This is a placeholder request, not a real target or credential.

## 2. Repeatable assessment workflow

### A — Scope and inventory
- Confirm written scope, allowed hosts, accounts, data, and restrictions.
- Identify base URLs, versions, routes, methods, and content types from permitted documentation, browser traffic, or an OpenAPI specification.
- Identify authentication and roles (anonymous, regular, privileged).
- Inventory routes; flag undocumented, deprecated, duplicate, or debug endpoints.
- Do not probe outside approved scope.

### B — Capture a baseline
| Field | Record |
|---|---|
| Route/method | e.g. `GET /api/v1/profile` |
| Purpose | Intended operation |
| Authentication | Required, optional, public |
| Role/owner | Which identity is allowed |
| Inputs | Path, query, headers, body |
| Response | Status, schema, sensitive fields |
| Evidence | Redacted request/response |

Start with expected requests so later comparisons have a baseline.

### C — Authorization (high priority)
Authentication asks **who is signed in?** Authorization asks **what may that identity do?**

Use two lab accounts you control and, where provided, different roles. Verify:
- Users can access their own objects, but not another user's objects unless explicitly allowed.
- Access checks cover reads, updates, deletes, exports, and nested resources—not only the UI.
- Regular users cannot call functions reserved for administrators.
- Authorization is server-side, not based only on hidden fields or client-side visibility.
- Anonymous requests are denied where authentication is required.

Use test data only. Avoid changing/deleting another user's data. Confirm suspected issues with the smallest safe read-only action and document expected versus observed authorization.

### D — Property and input handling
- Review which JSON properties each user can read or write.
- Compare documented schemas with accepted fields.
- Protect server-controlled properties (role, ownership, account state, price, balance, permissions).
- Observe validation for types, lengths, ranges, required/unexpected fields, and content types.
- Check errors for stack traces, internal paths, and sensitive values.
- Keep tests non-destructive and within defined limits.

### E — Authentication and tokens
- Review login, registration, password reset, logout, and session renewal.
- Check token expiry, refresh, revocation, and storage.
- Verify JWT signatures and server-side claims; decoded payloads are not inherently trustworthy.
- Confirm sensitive operations require authentication and authorization.
- Do not test credential attacks or lockout behavior beyond explicit scope.

### F — Business logic, rate limits, and resources
- Understand workflows and state transitions.
- Check whether operations can be repeated or performed out of order.
- Review limits and whether they apply to relevant actions/identities.
- Check pagination, upload size, batch size, timeouts, and expensive operations within lab limits.
- Avoid high-volume, stress, financial, or irreversible tests without explicit authorization.

### G — Configuration and inventory
- Review TLS, CORS, allowed methods, content types, security headers, debug behavior, and error handling.
- Compare documented versions with observed routes.
- Identify obsolete endpoints, debug routes, and inconsistent controls across versions.
- Validate external API responses, set timeouts, and handle redirects carefully.

### H — Evidence and reporting
- Save only the minimum request/response needed; redact tokens, cookies, keys, personal data, and unrelated records.
- Describe expected and observed behavior, impact, affected role/object.
- Provide safe reproduction within the authorized lab.
- Retest fixes and record the result.

## 3. OWASP API Security Top 10 (2023)

| ID | Risk | Review question |
|---|---|---|
| API1 | Broken Object Level Authorization (BOLA) | Does each request verify access to the specific object? |
| API2 | Broken Authentication | Are identity and tokens handled securely? |
| API3 | Broken Object Property Level Authorization | Are sensitive fields protected and writable fields allowlisted? |
| API4 | Unrestricted Resource Consumption | Are resource, payload, and request limits appropriate? |
| API5 | Broken Function Level Authorization | Are operations restricted to permitted roles? |
| API6 | Unrestricted Access to Sensitive Business Flows | Can a workflow be abused at scale or out of order? |
| API7 | Server-Side Request Forgery (SSRF) | Can user-controlled destinations trigger unsafe server requests? |
| API8 | Security Misconfiguration | Are TLS, CORS, errors, and defaults configured safely? |
| API9 | Improper Inventory Management | Are versions, hosts, routes, and environments accounted for? |
| API10 | Unsafe Consumption of APIs | Are third-party responses, redirects, and integrations validated and bounded? |

Refer to the official OWASP material for detailed threat models, test guidance, and mitigations.

## 4. Postman + Burp Suite

Postman organizes requests, environments, variables, auth, and repeatable tests. Burp can intercept and inspect requests during controlled testing.

### Same-machine lab setup
1. Start Burp and confirm the proxy listener is running on the intended interface and port.
2. In Postman Desktop, open **Settings → Proxy** (labels can vary by version).
3. Enable custom proxy settings for HTTP and HTTPS.
4. Set the proxy host/port to the Burp listener. If both apps are on one machine, a common lab setting is `127.0.0.1:8080`; use the actual listener values.
5. Send a Postman request and confirm it appears in Burp's Proxy history.
6. For HTTPS interception, trust the Burp CA only in the lab client environment; remove that trust when no longer needed.
7. Use Postman environment variables for lab URLs and tokens. Never commit real secrets.

### VM / WSL / host split
- `127.0.0.1` inside a guest may refer to the guest, not the host. Use a host address reachable from that environment when necessary.
- Configure the Burp listener on the appropriate reachable interface and restrict access with a firewall.
- Do not expose the listener to the public internet or an untrusted network.
- Verify network connectivity, routing into Burp, and TLS trust as separate steps.
- Avoid conflicting system/custom proxy settings. Check Postman's settings and `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY`.

Official Postman references: [Proxy settings](https://learning.postman.com/docs/getting-started/installation/settings/proxy-settings/) and [Configure a proxy](https://learning.postman.com/docs/getting-started/installation/proxy/).

## 5. Practice checklist

Future lab work—not a claim that these tasks are already completed:
- [ ] Build a small REST API lab and identify documented routes.
- [ ] Capture a baseline request.
- [ ] Route Postman requests through Burp and verify in HTTP history.
- [ ] Create two test accounts with different ownership/roles.
- [ ] Inventory endpoints, methods, inputs, authentication, authorization, and response fields.
- [ ] Review BOLA, function-level, and property-level authorization using lab data.
- [ ] Review token/session lifecycle and server-side validation.
- [ ] Review rate limits, pagination, batches, and resource limits within lab bounds.
- [ ] Review errors, CORS, TLS, API versions, and third-party integrations.
- [ ] Write a finding with redacted evidence, impact, remediation, and retest.
- [ ] Record actual practice in a dated daily log.

## 6. Finding template

```markdown
### [Finding title]
**Affected endpoint:** [METHOD / route]
**Severity:** [Severity and rationale]
**Affected role:** [Test role/account]
**Precondition:** [Required access/state]

**Description** — [Security weakness]

**Evidence** — [Minimal redacted request/response]

**Impact** — [What can be achieved with stated access?]

**Remediation** — [Specific server-side control]

**Retest** — [Result, date, and evidence]
```

## References
- [OWASP API Security Top 10 — 2023](https://api-security.owasp.org/editions/2023/en/0x00-header/)
- [OWASP API Security Project](https://owasp.org/www-project-api-security/)
- [Postman proxy settings](https://learning.postman.com/docs/getting-started/installation/settings/proxy-settings/)
- [Postman configure a proxy](https://learning.postman.com/docs/getting-started/installation/proxy/)
