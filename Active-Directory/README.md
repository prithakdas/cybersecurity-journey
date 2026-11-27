# Active Directory Security — Assessment & Learning Notes

> **Status:** Reference and study checklist. This page is not evidence of a new hands-on lab. Record actual exercises separately in dated logs. Use only authorized domains, accounts, and lab environments.

## Objectives

- Understand core identity objects and relationships in a Windows domain.
- Build a consistent, scope-aware assessment workflow.
- Review privilege, delegation, Group Policy, authentication, and trust risks.
- Use graph analysis to explain identity relationships and prioritize remediation.
- Communicate findings with evidence and defensive recommendations.

## 1. Scope and safety

Before collection or testing, document:
- Approved domain(s), hosts, network ranges, test accounts, and time window.
- Permitted collection methods and tools; whether credentialed testing is allowed.
- Rules for sensitive data, account lockout, changes, and proof of impact.
- Emergency contacts and stop conditions.

Prefer read-only inventory and configuration review. Do not attempt destructive actions, password spraying, credential extraction, persistence, or privilege escalation outside explicit written scope.

## 2. Domain and identity inventory

| Area | Questions / evidence to collect |
|---|---|
| Domain and forest | Domain/forest names, functional level, sites, domain controllers, trusts |
| Users | Enabled/disabled status, stale accounts, privileged identities, service identities |
| Groups | Nested membership, privileged groups, ownership, membership review |
| Computers | Server/workstation inventory, unsupported OS, stale computer objects |
| Authentication | Kerberos/NTLM use, MFA coverage where applicable, policy configuration |
| Services | Service accounts, ownership, password management, SPN inventory |
| DNS | Zones, delegation, records, dynamic update policy, exposed internal naming |
| Shares | Share permissions and NTFS ACLs, sensitive data exposure, access groups |
| Group Policy | Linked GPOs, edit rights, scope, settings, exceptions, credential exposure |
| Trusts | Direction/type, scope, protective settings, business need |

Keep an inventory source, collection date, and known gaps. Avoid exporting more personal or sensitive data than needed.

## 3. Privileged access review

Review high-privilege groups and equivalent delegated roles, including:
- Domain Admins
- Enterprise Admins (where applicable)
- Built-in Administrators
- Account Operators, Server Operators, Backup Operators, and other delegated groups as relevant
- Groups or users granted administrative rights through GPO, local groups, or ACLs

For each privileged identity, verify:
- A documented business owner and purpose.
- Membership is minimal and reviewed regularly.
- Separate administrative accounts are used for privileged work.
- Strong authentication and secure administrative workstations are required where supported.
- Routine accounts are not unnecessarily privileged.
- Membership changes are logged and reviewed.

## 4. Permissions, ACLs, and delegation

Review directory permissions and delegation for unintended control paths. Look for:
- Excessive write or modify rights over users, groups, computers, OUs, and GPOs.
- Broad principals granted sensitive permissions.
- Unintended inheritance or broken inheritance that hides access.
- Delegated rights that allow sensitive account/group changes.
- Unreviewed ownership and control of critical objects.
- Local administrator access broader than necessary.

For each concern, record the object, permission, principal, intended role, and why the effective access is excessive. Validate the interpretation in a test environment or with the system owner before making changes.

## 5. Group Policy review

Inventory GPOs and examine:
- Who can create, edit, link, or delete each GPO?
- Which OUs and systems receive it?
- Does it add unprivileged identities to local groups with elevated access?
- Are security settings consistent with the organization's baseline?
- Are old, unlinked, conflicting, or overly broad policies present?
- Are credentials, secrets, or reversible-password settings exposed in policy files?
- Are exceptions documented, approved, and time-bound?

Changes should be tested in a staging OU or representative lab before deployment. Preserve a rollback plan.

## 6. Authentication and service-account posture

Review policy and configuration rather than attempting credential attacks:
- Identify where legacy authentication is still required and whether its use is monitored.
- Check password and lockout policy applicability; note fine-grained policy differences.
- Review service identities, ownership, lifecycle, privileges, and password management.
- Verify supported protections for privileged authentication and sensitive accounts.
- Examine whether authentication events are centrally collected and alertable.
- Review Kerberos-related configuration and service-account risk using approved inventory and defensive guidance.

Do not collect or publish real password hashes, tickets, tokens, or secrets in this repository.

## 7. Trusts and cross-domain relationships

Document each trust relationship and its business purpose:
- Trust direction and type.
- Domains/forests involved and owner.
- Scope and access expectations.
- Applicable protective settings and review date.
- Whether the trust is still required.
- Monitoring and change-control requirements.

Assess cross-domain access as a permissions and configuration review. Do not attempt trust abuse or access to a separate domain unless explicitly authorized.

## 8. Graph-based relationship analysis

BloodHound Community Edition models identity relationships as a graph. In an authorized lab or approved assessment, use graph analysis to understand:
- Membership and nested-group relationships.
- Delegated permissions and control edges.
- Relationships between identities, computers, and directory objects.
- Potentially unintended paths to sensitive roles.
- Whether an observed path is technically valid, in scope, and supported by evidence.

Treat a graph path as a lead to validate—not automatically a confirmed vulnerability. Explain the relevant relationship chain, affected objects, permissions, preconditions, and defensive changes. Protect collected data because directory relationship exports can be sensitive.

## 9. Monitoring and detection review

Confirm logging and alerting covers relevant events, such as:
- Privileged group membership changes.
- Account creation, disablement, and sensitive account modifications.
- GPO creation, linking, editing, and changes to privileged settings.
- Directory ACL and delegation changes.
- Authentication anomalies and use of legacy protocols.
- Domain-controller security and configuration changes.

Review event coverage, retention, access controls, alert routing, and whether test events are visible to the monitoring team.

## 10. Reporting template

Use a concise finding structure:

**Finding title:** [Title]  
**Affected object(s):** [Redacted identifiers as needed]  
**Severity:** [Level and rationale]  
**Precondition:** [Required access/context]

**Description:** [Observed configuration or permission]

**Evidence:** [Minimal redacted evidence]

**Impact:** [Specific, plausible consequence]

**Remediation:** [Least-privilege/configuration change]

**Validation:** [Safe verification and retest result]

Describe an observed weakness separately from a theoretical risk or graph-derived lead. Redact user identifiers, domain details, secrets, and sensitive paths before publishing.

## 11. Practice checklist — future authorized labs

- [ ] Define lab scope and test identities.
- [ ] Map domain, forest, OUs, sites, and trust relationships.
- [ ] Inventory users, groups, computers, and service accounts.
- [ ] Review privileged group membership and delegated administration.
- [ ] Review directory ACLs and object ownership.
- [ ] Review GPO scope, permissions, and security settings.
- [ ] Examine authentication policy and service-account lifecycle.
- [ ] Review trusts and cross-domain access.
- [ ] Use BloodHound to visualize and validate relationships.
- [ ] Review logging for relevant identity and configuration events.
- [ ] Produce a redacted assessment note with remediation and retest.
- [ ] Log only tasks actually performed in a dated daily entry.

## References

- [Microsoft — Best practices for securing Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory)
- [Microsoft Defender for Identity — Group Policy security posture assessments](https://learn.microsoft.com/en-us/defender-for-identity/modified-unprivileged-accounts-gpo)
- [SpecterOps — BloodHound documentation](https://github.com/SpecterOps/bloodhound-docs)
- [SpecterOps — BloodHound Community Edition](https://specterops.io/bloodhound-community-edition/)

---

[Back to repository overview](../README.md)
