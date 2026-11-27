# Web Application Penetration Testing — Learning Reference

> **Status:** Study/reference guide. The checklist below is a learning roadmap; it is not a record of a completed assessment. Document hands-on results only in dated lab notes or appropriately sanitized engagement reports.

Use this material only on systems you own, intentionally vulnerable labs, CTFs, or targets covered by explicit written authorization. Follow the engagement scope, rules of engagement, rate limits, data-handling requirements, and stop conditions. Avoid destructive tests, persistence, uncontrolled automation, and unnecessary access to sensitive data.

## 1. Assessment workflow

A quality web assessment is evidence-driven and iterative. Adapt the steps to scope and risk instead of treating the list as a rigid checklist.

| Phase | Goal | Outputs |
|---|---|---|
| 0. Authorization & scope | Confirm permitted targets, test accounts, methods, time window, exclusions, and escalation contact | Scope notes and rules of engagement |
| 1. Reconnaissance | Understand the application, exposed routes, technologies, roles, and entry points | Asset/route inventory and technology notes |
| 2. Map functionality | Identify workflows, data, trust boundaries, and state changes | Application map, role matrix, workflow diagrams |
| 3. Test controls | Assess authentication, authorization, input handling, session controls, configuration, and logic | Test evidence and observations |
| 4. Validate findings | Reproduce safely, isolate root cause, evaluate realistic impact, and rule out false positives | Verified finding records |
| 5. Report | Communicate risk, evidence, and practical remediation | Prioritized report |
| 6. Retest | Verify fixes and assess residual risk | Retest status and closure notes |

## 2. Scope and safety checklist

Before testing, record:

- [ ] Written authorization and scope document are available.
- [ ] Target hosts, domains, IP ranges, applications, APIs, and environment are listed.
- [ ] Out-of-scope hosts, third-party services, and prohibited techniques are clear.
- [ ] Testing time window, rate limits, and monitoring expectations are understood.
- [ ] Test accounts and roles are provided or approved for creation.
- [ ] Data handling, evidence storage, redaction, and retention rules are understood.
- [ ] A contact and stop-testing process exist for instability or accidental sensitive-data exposure.
- [ ] Destructive, denial-of-service, social-engineering, persistence, or data-exfiltration tests are explicitly excluded unless separately authorized.

Do not assume that a hostname discovered during enumeration is in scope. Verify it against the authorization.

## 3. Reconnaissance and application mapping

Start with passive and low-impact discovery. Build an inventory rather than collecting a large list of endpoints without context.

### Inventory

Record, as applicable:

- Application entry points and authorized hostnames.
- Technologies and versions, with confidence and evidence.
- Public pages, login/logout, password reset, registration, and account recovery.
- API routes, request methods, content types, and version prefixes.
- Static assets, client-side routes, source maps, and exposed documentation.
- Roles, accounts, and access levels available for testing.
- Features that create, update, delete, export, upload, or approve data.
- External integrations, redirects, and trust boundaries.

### Useful tools

- **Browser developer tools:** inspect requests, responses, DOM, storage, JavaScript, and network activity.
- **Burp Suite / OWASP ZAP:** intercept, replay, compare, and organize authorized HTTP traffic.
- **Nmap:** service discovery only for in-scope hosts and at approved intensity.
- **ffuf / Gobuster:** route and content discovery with scoped wordlists and agreed rate limits.
- **Nuclei:** template-based checks; review the template, request, evidence, and false-positive potential before reporting.
- **Postman:** organize API requests, role-based test collections, and response comparisons.

Automation helps coverage, but a scanner result is a lead—not proof of vulnerability.

## 4. Map roles and workflows

Create a simple role matrix before testing access control.

| Function / object | Anonymous | User A | User B | Privileged role |
|---|---|---|---|---|
| View public content | Expected behavior | Expected behavior | Expected behavior | Expected behavior |
| View own private object | N/A / blocked | Allowed if designed | Allowed for own object | Depends on policy |
| View another user's object | Denied | Denied unless shared | Denied unless shared | Explicitly authorized |
| Modify account/profile | Denied or limited | Own object only | Own object only | Policy-dependent |
| Administrative action | Denied | Denied | Denied | Explicit permission |

For each workflow, note:
- Which role performs the action?
- What object or resource is affected?
- Which request changes state?
- Which server-side control is expected?
- What should happen when the action is attempted by a different role?

Use test accounts and objects you control. Do not access real users' private data to demonstrate a flaw.

## 5. Core testing areas

### Authentication and account lifecycle

- Registration and account activation requirements.
- Login error handling and account-enumeration signals.
- MFA enrollment, enforcement, recovery, and removal flows (within scope).
- Password-reset tokens: expiry, single use, invalidation, and binding to the intended account.
- Logout and session invalidation.
- Password change and reauthentication requirements for sensitive actions.
- Rate limits and lockout controls, using the agreed safe test volume.

### Authorization and access control

- Horizontal access: user-to-user object ownership boundaries.
- Vertical access: lower-privileged role attempting privileged functions.
- Server-side enforcement on every request, including APIs and background actions.
- Object identifiers in paths, query strings, forms, and JSON bodies.
- Hidden UI controls versus actual server-side authorization.
- Role or permission changes and stale sessions.
- Authorization checks for exports, downloads, bulk actions, and secondary endpoints.

**Key principle:** changing an identifier or hiding a button must not be the only access-control mechanism. Verify the server's decision using controlled accounts and owned test objects.

### Session and token security

- Cookie flags: `Secure`, `HttpOnly`, and appropriate `SameSite`.
- Session identifier rotation after login and privilege changes.
- Session expiration and logout behavior.
- Token storage, leakage through URLs/logs/referrers, and scope.
- JWT signature, algorithm allowlist, expiry, issuer, audience, and authorization claims—where JWTs are used.
- CSRF defenses for state-changing browser requests when applicable.

### Input handling and injection

Identify input locations: query parameters, path segments, headers, cookies, JSON/XML bodies, form fields, file names, and stored content.

Review for:
- SQL/NoSQL/command injection.
- Cross-site scripting (reflected, stored, DOM-based).
- Template injection and unsafe deserialization.
- Path traversal and unsafe file access.
- XML external entity (XXE) risks where XML processing exists.
- Unsafe redirects and URL parsing.
- Server-side request forgery (SSRF).

Use safe, non-destructive validation techniques in a lab or approved scope. Avoid commands or payloads that alter data, access unrelated files, reach internal services, or cause outages. Confirm the relevant input-to-sink path and the security control that failed.

### Business logic and workflow

Security bugs may arise when valid features are combined in an unintended way. Map the expected workflow and test, within scope:

- Whether actions can be repeated, skipped, reordered, or performed concurrently.
- Whether server-side limits, quotas, prices, and ownership are authoritative.
- Whether approval, payment, or verification states can be bypassed.
- Whether bulk operations enforce the same permissions as single-item operations.
- Whether state changes are atomic and consistent.
- Whether error or retry behavior creates an unintended state.

### File upload and download

- Allowed types, extension/content consistency, size limits, and server-side validation.
- Safe storage names and storage location.
- Whether uploaded content can be executed or interpreted.
- Access control on upload, preview, and download.
- Metadata, path, and content-disposition handling.
- Antivirus/content scanning and isolation where appropriate.
- Protection against unauthorized overwrite or retrieval.

Use benign test files and your own accounts. Never upload malware or test files designed to harm users unless a separately approved exercise specifically authorizes it.

### Configuration and deployment

- HTTPS/TLS and redirect behavior.
- Security headers and their applicability: HSTS, CSP, frame protections, content-type protections, and browser permissions policy.
- CORS policy: allowed origins, credential behavior, and whether policy matches the intended trust model.
- Debug mode, verbose errors, stack traces, and diagnostic endpoints.
- Exposed backups, configuration files, secrets, source maps, and directory listings.
- Default credentials and unnecessary services.
- Outdated components: verify versions and match them to vendor advisories; a version banner alone is not proof of exploitability.

### API and client-side areas

Use the dedicated [API Security guide](../API-Security/README.md) for API-specific assessment. In web testing, also review:
- API routes called by browser features and their authorization behavior.
- CORS and credentialed cross-origin requests.
- Client-side secrets (public configuration versus actual credentials).
- DOM sinks, client-side routing, and trust placed in local storage or URL values.
- Sensitive data returned but hidden by the interface.
- WebSocket authorization and message validation if used.

## 6. Finding validation and evidence

A reportable finding should have:

1. **Title:** weakness + affected component + consequence.
2. **Severity:** context-based risk, not just a scanner label.
3. **Affected endpoint/feature:** sanitized where needed.
4. **Prerequisites:** account role, access, or conditions.
5. **Expected behavior:** the control that should apply.
6. **Observed behavior:** what actually happened, with status codes and sanitized evidence.
7. **Reproduction:** minimal, repeatable steps using test accounts and owned data.
8. **Impact:** realistic security consequence and affected assets/data.
9. **Root cause:** the missing or ineffective control.
10. **Remediation:** specific change, plus broader prevention if useful.
11. **References:** relevant CWE, OWASP WSTG/ASVS, or vendor guidance.
12. **Retest:** verification of the fix and any residual issue.

Capture enough evidence to prove the behavior without collecting unrelated personal data. Redact cookies, tokens, credentials, private identifiers, and client information. Keep request/response evidence in approved storage.

## 7. Severity and prioritization

Use the client's or program's severity model when defined. Otherwise assess:

- **Impact:** confidentiality, integrity, availability, account/privilege impact, and business consequences.
- **Exploitability:** required access, user interaction, complexity, and reliability.
- **Exposure:** affected users, data, functions, and reachable environments.
- **Context:** compensating controls, asset importance, and whether the finding is independently exploitable.

Avoid severity inflation. Explain the scenario that makes the rating credible. A scanner finding, missing header, or version disclosure may be informational or low in one context and more serious in another; evidence and impact decide.

## 8. Report template

Copy this for each verified finding:

```markdown
# [Severity] Finding title

## Summary
One-paragraph description of the weakness.

## Affected component
Endpoint, feature, version, or role (sanitized).

## Preconditions
Required account, role, or state.

## Steps to reproduce
1. Use an authorized test account.
2. Perform the minimal safe action.
3. Record the observed result.

## Expected vs. observed
Expected security control and actual behavior.

## Evidence
Sanitized request/response, screenshot, or log.

## Impact
Realistic security and business consequences.

## Root cause
Why the control is missing or ineffective.

## Remediation
Specific fix and defense-in-depth suggestions.

## References
CWE / OWASP WSTG / ASVS / vendor advisory.

## Retest
Status, verification evidence, residual risk.
```

## 9. Learning progression

Suggested order for a practice lab:

- [ ] Map routes and application features.
- [ ] Learn how the application authenticates and maintains sessions.
- [ ] Build a two-role access-control matrix.
- [ ] Test ownership and role boundaries using your own accounts/objects.
- [ ] Review input handling and output encoding.
- [ ] Review file handling and configuration.
- [ ] Explore one business workflow and document its expected state transitions.
- [ ] Use a scanner only as a coverage aid; manually validate findings.
- [ ] Write a finding with reproduction, evidence, impact, and remediation.
- [ ] Retest the fix in the lab and record what changed.

Record completed lab work separately in [Daily Logs](../Daily-Logs/README.md). Do not mark checklist items complete until you have actually practiced and documented them.

## 10. Official references

- [OWASP Web Security Testing Guide (WSTG)](https://wstg.owasp.org/) — practical test methodology. The stable release and current development guide are available from the project; cite a versioned scenario in reports to make references reproducible.
- [OWASP WSTG stable guide](https://owasp.github.io/www-project-web-security-testing-guide/stable/) — stable guidance; v4.2 is the stable version while v5.0 is under development (check the project page for current status).
- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-project-application-security-verification-standard/) — security requirements and verification baseline.
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) — focused implementation and defense guidance.
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) — intentionally vulnerable practice application.
- [OWASP ZAP](https://www.zaproxy.org/) — web proxy and application testing tool.
- [Burp Suite documentation](https://portswigger.net/burp/documentation) — proxy-based web security testing.

---

`root@lordkraken:~/cybersecurity-journey/Web-Penetration-Testing$`
