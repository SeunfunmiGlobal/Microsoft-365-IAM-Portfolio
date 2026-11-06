# IAM Integration Portfolio Project

**Microsoft Entra ID + Okta + SailPoint Identity Security Cloud (ISC) + CyberArk**  
**Artifact type:** hands-on implementation blueprint / portfolio runbook  
**Status:** lab plan; replace all sample values and verify product-specific requirements before implementation.  
**UI note:** every UI map in this package is an **illustrative UI guide**, not a screenshot from your tenant. Labels, navigation, licensing, and features can vary by edition and release.

> **Safety boundary:** Build in a test tenant or isolated lab. Do not use production identities, real employee data, production secrets, or privileged accounts for initial experiments. Never put tenant IDs, client secrets, certificates, recovery codes, or access tokens in this page or public portfolio.

## 1. Project brief

Build a demonstrable IAM integration that separates authentication, provisioning, governance, and privileged access. The finished portfolio should show that you can plan integration direction, configure a controlled pilot, validate lifecycle events, interpret logs, and document rollback.

### Business scenario

A fictional organization, **Northstar Demo**, wants a controlled workforce identity lifecycle:

- Microsoft Entra ID is the cloud directory and sign-in policy platform for the lab.
- Okta imports selected Entra users through the documented Microsoft Graph import path and provides a separate inbound SAML federation demonstration.
- SailPoint ISC aggregates Entra identities and groups, correlates accounts, and models access governance. Writeback is a separate, optional phase.
- CyberArk demonstrates a product-specific workforce/privileged-access integration. Exact steps depend on the CyberArk product available in the lab.

These are **reference roles**, not a requirement to make every system authoritative or every connection bidirectional. Record the source of authority for each attribute and event before configuring sync.

### Portfolio objectives

- [ ] Document the target architecture and data-flow direction.
- [ ] Prepare a small pilot population and stable matching attributes.
- [ ] Demonstrate Entra → Okta user import without accidentally enabling reverse provisioning.
- [ ] Demonstrate Entra-to-Okta SAML trust and successful pilot sign-in.
- [ ] Connect SailPoint ISC to Entra, aggregate identities, and inspect correlation and entitlements.
- [ ] Identify the exact CyberArk product and implement only its supported integration path.
- [ ] Test create, update, group change, disable, restore/rehire, and sign-in outcomes.
- [ ] Capture sanitized evidence, a troubleshooting log, and rollback steps.

## 2. Architecture and data flows

### Intended flows

| Flow | Purpose | Mechanism | Guardrail |
|---|---|---|---|
| HR/test source → Entra | Workforce lifecycle (optional for this lab) | Existing connector/API or manual test setup | Do not claim HR integration unless actually built |
| Entra → Okta | Import users and optionally supported groups | Okta Microsoft 365 integration using Graph | Review import limitations; avoid broad auto-confirmation |
| Entra ↔ Okta trust | Authenticate Okta users using Entra | SAML 2.0 inbound IdP configuration | Federation is not user provisioning |
| Entra → SailPoint ISC | Identity and group aggregation | SailPoint Entra connector / Microsoft Graph | Aggregation does not itself enable writeback |
| SailPoint ISC → Entra | Optional governed provisioning | Connector provisioning policies | Separate change phase, approval, pilot, and rollback |
| Entra ↔ CyberArk product | Workforce identity, SSO, or access integration | Product-specific supported connector / SAML / OIDC / SCIM / API | Select exact product/version first |
| CyberArk privileged workflows | Vault accounts, safe membership, rotation, sessions | Product-specific PAM controls | Directory sync alone does not onboard or secure privileged accounts |

### Authority decisions worksheet

Complete before configuration. Example defaults are suggestions, not tenant facts.

| Data or decision | Lab authority | Write direction / notes |
|---|---|---|
| Worker status and employee attributes | Fictional HR dataset or manually managed lab data | HR → Entra only if an HR connector is actually configured |
| Entra account enabled state | Entra for this lab | Import/aggregate downstream; test target disable semantics |
| Stable correlation key | Entra object ID where supported; employee ID only if unique and stable | Keep login UPN separate from immutable correlation key |
| Email / UPN | Explicitly assigned lab source | Mutable; do not rely on email alone for privileged auto-linking |
| Authentication and MFA policy | Entra for the SAML pilot | Confirm interactions with Okta policies |
| Access requests and certifications | SailPoint ISC (if licensed/configured) | Governance does not equal raw group sync |
| Privileged accounts, safes, sessions | CyberArk product | Separate PAM workflow from workforce identity sync |

## 3. Lab scope, assumptions, and prerequisites

### Required decisions

- Entra tenant and licensing; admin role and pilot group owner.
- Okta org / Identity Engine and Microsoft Office 365 app integration availability.
- SailPoint ISC tenant, Entra connector type, supported authentication method, and exact Graph permission list.
- CyberArk product name, deployment model, version/region, and desired outcome.
- Approved test users and a rollback contact.

### Suggested pilot identities

Use fictional names and controlled test mailboxes only. Do not use real employee data.

| Test identity | Scenario | Expected use |
|---|---|---|
| `lab.alex` | Standard active user | Baseline import, aggregation, and SAML test |
| `lab.jordan` | Attribute-change test | Department/title update and correlation validation |
| `lab.taylor` | Disabled-user test | Verify disable/visibility behavior across platforms |
| `lab.morgan` | Group membership test | Add/remove one low-risk pilot group |
| `lab.admin` | Dedicated lab admin | Administrative setup only; exclude from broad auto-link/JIT tests |

Use a dedicated security group such as `IAM-Integration-Pilot`. Keep break-glass accounts, service accounts, shared mailboxes, guests, and production administrators out of the pilot unless explicitly approved.

### Baseline evidence to capture

- Sanitized architecture and authority matrix.
- Pilot roster with fictitious identifiers.
- Before/after counts and sample attributes.
- Entra sign-in/audit/provisioning logs; Okta import results and System Log; SailPoint aggregation/task results; CyberArk audit evidence where available.
- Test outcomes, issues, remediation, and rollback record.

## 4. Step-by-step implementation

## 4.1 Microsoft Entra ID — prepare the pilot and SAML enterprise app

### A. Prepare users and scope

1. Open the Microsoft Entra admin center.
2. Confirm whether test users are cloud-only or synchronized from on-premises AD. Avoid editing attributes mastered elsewhere.
3. Create a dedicated pilot security group and add only the fictional test users.
4. Review `id`, `userPrincipalName`, `mail`, `displayName`, `givenName`, `surname`, `employeeId`, `department`, `jobTitle`, `manager`, `accountEnabled`, and relevant group memberships.
5. Decide which attributes are required downstream. Document how nulls, renamed users, disabled users, and rehires should behave.

### B. Create the enterprise application for Okta SAML

1. Navigate to **Identity / Entra ID → Applications → Enterprise applications**. Portal labels may differ.
2. Select **New application → Create your own application**. Give it a clear lab name, such as `Okta-Inbound-SAML-Pilot`.
3. Open **Single sign-on → SAML**.
4. Download the Base64 signing certificate and record Entra Identifier and Login URL in your secure lab notes.
5. If temporary SAML values are needed to reach the setup page, replace them with Okta's actual Audience URI and ACS URL before testing.
6. Under **Users and groups**, assign only the pilot group where supported/required.
7. After configuring Okta (section 4.2), enter Okta's ACS URL as the Reply URL and Audience URI as the Identifier. Save.
8. Review **Attributes & Claims**. Set NameID to the agreed matching identifier and remove unnecessary claims.
9. Test sign-in with a pre-provisioned pilot user. Review Entra sign-in logs and verify Conditional Access/MFA behavior.

**Do not confuse this enterprise app's SAML trust with the Okta Graph import.** The SAML setup authenticates; it does not automatically synchronize users or groups.

## 4.2 Okta — import Entra users and configure inbound SAML

### A. Configure the Microsoft Graph import

1. In Okta Admin Console, open **Applications → Applications** (navigation can vary) and add/select the Microsoft Office 365 integration for the correct Microsoft tenant.
2. Open the app's **Provisioning → API Integration / Integration** settings.
3. Enable API integration and authenticate with an appropriately authorized Entra administrator.
4. Review requested Graph permissions and grant consent only through the lab's approved process.
5. Test API credentials and resolve errors before saving.
6. Configure import scope, username format, and matching attributes. Start with the pilot population; enable group import only if required and supported.
7. Run an initial full import. Inspect new, matched, skipped, and conflicting records before confirming. Resolve duplicates first.
8. Test a harmless attribute update and group change, then review the import results and Okta System Log.
9. Configure import schedules only after the pilot passes. Verify current vendor limitations for disabled/deleted users, unlicensed users, AD-synced groups, nested groups, and incremental imports.
10. Keep **Provisioning → To App** create/update/deactivate actions disabled unless Okta is explicitly approved to write back to Entra/Microsoft 365. Saving reverse provisioning scope may have immediate effects.

### B. Configure Entra as an inbound SAML IdP for Okta

1. In Okta, navigate to **Security → Identity Providers → Add Identity Provider → SAML 2.0 IdP**.
2. Set a clear name such as `Microsoft Entra - Workforce`.
3. Enter Entra's Identifier as the issuer, Entra's Login URL as the SSO URL, and upload the Entra Base64 signing certificate.
4. Configure incoming IdP username and **Match against** using the agreed Okta profile attribute. Prefer a stable, well-governed match; avoid privileged auto-linking based only on mutable email.
5. Keep JIT user creation disabled for the pre-provisioned-user pilot. If you later enable JIT, design duplicate handling, required attributes, group behavior, and offboarding first.
6. Save the IdP and copy the generated ACS URL and Audience URI.
7. Return to Entra and populate the SAML Identifier and Reply URL with those values. Save and verify NameID/claims.
8. Test SP-initiated sign-in with an existing pilot account. Confirm it links to the expected account and no unexpected user is created.
9. Inspect Okta System Log and Entra sign-in logs. Add routing rules only after successful testing; preserve an admin recovery path.

### Acceptance check

- [ ] Graph API credentials test succeeds.
- [ ] Initial import reconciles with the pilot roster.
- [ ] No duplicate accounts were silently auto-confirmed.
- [ ] A pilot sign-in succeeds through the expected SAML trust.
- [ ] Reverse provisioning remains off unless separately approved.

## 4.3 SailPoint Identity Security Cloud — connect Entra and govern access

> This section assumes SailPoint Identity Security Cloud with a Microsoft Entra SaaS connector. Another SailPoint product, connector, or guided onboarding flow may differ.

1. In Entra, create/configure the app registration and service principal required by the selected SailPoint connector.
2. Consult the current connector guide for the exact Microsoft Graph permissions needed for your chosen aggregation and provisioning capabilities. Grant only required permissions through approved admin consent.
3. Use the connector-supported credential method. Store secrets/certificates in a secure store and assign rotation and expiry owners.
4. In ISC, navigate to **Admin → Connections → Sources** (labels may vary), select **Create New Source**, and choose the Microsoft Entra SaaS connector.
5. Enter source name, tenant ID, application/client ID, and the required credential fields. Test the connection before saving.
6. Configure account/group schemas and only the attributes needed for correlation, identity profile, access governance, and approved provisioning. Preserve Entra object ID as the native stable account key where supported.
7. Configure aggregation filters for pilot users and groups. Enable group hierarchy or delta aggregation only when needed and supported.
8. Run initial aggregation and review task results, account totals, duplicates, orphan accounts, disabled users, memberships, and identity correlation.
9. Correct identity profile, correlation rules, and source precedence before making access decisions.
10. Define lifecycle states, access profiles/roles, certifications, request/approval rules, and leaver controls. Test joiner/mover/leaver cases.
11. If ISC must provision back to Entra, treat it as a separate change phase: define approved operations, write ownership, approvals, target policies, disable/delete semantics, and rollback. Pilot each operation separately.
12. Reconcile ISC task results with Entra audit logs and assign connector-failure owners.

### Acceptance check

- [ ] Connection test succeeds with least-privilege permissions.
- [ ] Aggregation totals and sample records match the pilot population.
- [ ] Correlation rules avoid duplicate or orphan identities.
- [ ] Access profiles/roles and approval workflows are documented.
- [ ] No provisioning writeback is enabled unintentionally.

## 4.4 CyberArk — product decision gate and integration

**Do not treat CyberArk as one universal console.** First identify the exact product: CyberArk Identity, Privilege Cloud/PAM Self-Hosted, Secure Cloud Access (SCA), Secure Infrastructure Access (SIA), or Endpoint Privilege Manager (EPM). The integration method and available screens depend on product, edition, deployment, region, and release.

### A. Record the product decision

- Product / edition: `________________`
- Deployment (SaaS / self-hosted): `________________`
- Region / version: `________________`
- Desired outcome: SSO / MFA / workforce user-group sync / governance / privileged account and session control
- Official product-specific Entra integration guide: `________________`
- Supported direction and mechanism: `________________`

### B. Product-specific configuration workflow

1. Find the current official integration guide for the exact CyberArk product and version.
2. Confirm whether Entra acts as an authentication IdP, identity source, or provisioning target; confirm supported SAML, OIDC, SCIM, Graph, directory connector, or agent options.
3. Create the Entra enterprise app or app registration only as directed by that guide. Record endpoints, issuer, client/tenant IDs, ACS/redirect URIs, claims, permissions, and credential requirements in a secure configuration record.
4. Grant least-privilege permissions through approved admin consent. Use a dedicated non-human identity where required and supported; protect and rotate credentials.
5. In the CyberArk console, open the product-specific identity, directory, federation, or connector settings described in the guide. Configure matching attributes and pilot scope.
6. For SSO, test sign-in, MFA, policy behavior, and emergency access. For lifecycle integration, test create, attribute update, group change, disable, and restore/rehire.
7. Validate role removal, local-account exceptions, active sessions, and audit events. Do not assume removing an Entra assignment deletes CyberArk vault accounts or revokes sessions.
8. Expand scope only after reconciliation and owner approval.

### C. Keep PAM operations distinct

Workforce identity sync does **not** itself discover/onboard privileged accounts, configure safe membership, rotate credentials, authorize sessions, or set up session monitoring. If the chosen CyberArk product includes PAM, document these as separate workflows with their own approvals and test evidence.

### Acceptance check

- [ ] Exact CyberArk product and official integration guide are recorded.
- [ ] Supported direction, permissions, and object types are confirmed.
- [ ] Pilot identity/group and access outcomes are verified.
- [ ] Privileged-account onboarding and session controls are separately tested if in scope.

## 5. End-to-end validation plan

Run only with lab accounts and low-risk access. Record expected and actual results plus evidence reference.

| Test ID | Action | Expected result | Evidence |
|---|---|---|---|
| T01 | Add new pilot user to Entra scope | User appears in Okta import and ISC aggregation as designed | Import/aggregation result |
| T02 | Change department/title | Only attributes with approved write direction change | Before/after sample |
| T03 | Change UPN/email (if safe) | Existing identity remains correctly correlated; no duplicate | Correlation and logs |
| T04 | Add/remove pilot group | Target memberships and governance reflect intended policy | Group membership comparison |
| T05 | Sign in through SAML | Correct existing Okta account is linked; expected MFA/CA applies | Entra and Okta sign-in logs |
| T06 | Disable test user | Access is disabled within agreed lab target; exceptions are recorded | Source/target status and timestamps |
| T07 | Restore/rehire test | Correct identity is restored or recreated without unsafe duplicate | Lifecycle logs |
| T08 | Remove user from scope | Behavior matches documented disable/delete policy | Provisioning/import evidence |
| T09 | Use unknown/duplicate match key | Manual review or error; no unsafe auto-link | Error or review evidence |
| T10 | Simulate expired connector credential in nonproduction, if safe | Failure is visible and recovery steps work | Sanitized failure/recovery log |
| T11 | Revoke a privileged pilot assignment | Workforce access, roles, sessions, and vault-account handling match product policy | CyberArk audit evidence |

### Go/no-go criteria

- No unexplained duplicates or mismatched accounts.
- All pilot create/update/group/disable/restore tests pass or have an approved exception.
- Federation uses the intended identifier and certificate; MFA/Conditional Access behavior is understood.
- Connector permissions, secrets, and owners are documented securely.
- Logs and alerts are accessible; rollback and break-glass paths are tested.
- IAM, platform, security, and data owners approve expansion.

## 6. Troubleshooting decision guide

| Symptom | First checks | Safe response |
|---|---|---|
| SAML Reply URL / audience error | Compare Entra Reply URL and Identifier with Okta ACS and Audience exactly | Correct values, retest with pilot; do not weaken policy to bypass the error |
| Signature/certificate failure | Active signing certificate, issuer, signing settings, rollover state | Confirm trust metadata and planned rollover window |
| Sign-in works but user does not link | NameID, incoming IdP username, Okta match attribute, duplicates | Stop broad auto-link; fix matching and test again |
| User missing from Okta import | Graph consent, scope, licensing, account state, import limitations | Check full import and current vendor notes |
| Duplicate user in target | Correlation key, UPN/email changes, import preview | Pause confirmations and reconcile before resuming |
| SailPoint aggregation works but no identity correlates | Native account key, identity profile, correlation rule, source precedence | Correct correlation before access certifications or provisioning |
| SailPoint provisioning fails | Connector permissions, required attributes, target policy, task error | Fix root cause; avoid repeated blind retries |
| CyberArk identity exists but privileged access fails | Exact product, role/group assignment, safe membership, policy, separate PAM onboarding | Validate the product-specific workflow and audit trail |
| Disable does not revoke access | Target disable semantics, scope/job state, local accounts, sessions | Follow documented manual revocation and incident procedure |
| Connector stalls/quarantines | Credential expiry, mappings, API throttling, repeated failures | Inspect logs, correct cause, then restart according to vendor guidance |

## 7. Security, privacy, and rollback

- Use least-privilege Graph/API permissions and record the consent approver.
- Prefer managed or certificate-based credentials where supported; otherwise use a vault, expiry alerts, and rotation procedure.
- Never store secrets, tokens, private keys, or recovery codes in Notion, screenshots, tickets, or public repositories.
- Keep emergency-access accounts outside the federation dependency path and test the recovery process.
- Separate admin identities from ordinary workforce users. Avoid broad JIT auto-linking for privileged identities.
- Keep sign-in, audit, import, aggregation, provisioning, and privileged-session logs; define retention and alert owners.
- Document deactivation versus deletion, retention, rehire, and manual access revocation.

### Rollback checklist

1. Stop or disable the affected import/provisioning/federation change using the product-supported method.
2. Preserve sanitized logs and the change record.
3. Restore the last known-good scope, mappings, claims, or certificate trust as applicable.
4. Reconcile any created/updated/disabled target accounts; do not bulk-delete without impact review.
5. Revoke unintended access and check local accounts/active privileged sessions.
6. Verify break-glass/admin access and confirm sign-in policies remain effective.
7. Record root cause, recovery evidence, owner, and approval before restarting.

## 8. Portfolio deliverables and evidence checklist

Create a folder with the following sanitized artifacts:

- [ ] `01-architecture-and-data-flows.png` — logical flow and trust boundaries.
- [ ] `02-authority-and-attribute-mapping.md` — source of truth and field mapping.
- [ ] `03-entra-configuration-evidence/` — sanitized SAML config, pilot scope, sign-in results.
- [ ] `04-okta-import-and-saml-evidence/` — import reconciliation and federation logs.
- [ ] `05-sailpoint-aggregation-and-governance/` — aggregation counts, correlation and access model.
- [ ] `06-cyberark-product-and-test-evidence/` — exact product, source guide, supported method, sanitized test results.
- [ ] `07-test-results.csv` — test ID, date, expected, actual, status, evidence path.
- [ ] `08-troubleshooting-and-rollback.md` — issues and recovery.
- [ ] `09-readme.md` — what was actually implemented, what was simulated, limitations, and lessons learned.

**Evidence rule:** if you did not configure a feature in a tenant, label it “planned,” “simulated,” or “not tested.” Do not present illustrative UI maps as screenshots or claim production experience based on a lab design.

## 9. Suggested README for a portfolio repository

> **IAM Integration Lab — Entra ID, Okta, SailPoint ISC, and CyberArk**
>
> This project demonstrates an identity integration design covering directory preparation, controlled user import, SAML federation, identity aggregation and governance, and a product-specific privileged-access integration. It includes a source-of-authority matrix, least-privilege planning, lifecycle test cases, troubleshooting, and rollback documentation. Screens and UI maps are illustrative unless explicitly marked as sanitized evidence captured from my own lab. The CyberArk implementation scope depends on the product and licensing available in the lab.

### Interview talking points

- Why SAML federation and lifecycle provisioning are separate.
- Why stable correlation keys matter more than mutable email addresses.
- How you prevented reverse provisioning from overwriting Entra data.
- How SailPoint aggregation differs from provisioning and access governance.
- Why CyberArk product identification is a prerequisite for exact steps.
- How disable, rehire, duplicate matching, logging, and rollback were tested.

## 10. Implementation tracker

| Workstream | Status | Owner | Notes / evidence |
|---|---|---|---|
| Authority matrix and architecture | Not started |  |  |
| Entra pilot and SAML enterprise app | Not started |  |  |
| Okta Graph import | Not started |  |  |
| Okta inbound SAML | Not started |  |  |
| SailPoint ISC Entra aggregation | Not started |  |  |
| ISC governance workflows | Not started |  |  |
| CyberArk product decision | Not started |  |  |
| CyberArk supported integration | Not started |  |  |
| End-to-end tests | Not started |  |  |
| Rollback and portfolio README | Not started |  |  |

## 11. Official documentation starting points

Use current vendor documentation for exact permissions, feature availability, and UI labels:

### Microsoft Entra ID
- [SCIM support in Entra ID](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/scim-support-in-entra-id)
- [How application provisioning works](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/how-provisioning-works)
- [Customize application attribute mappings](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/customize-application-attributes)
- [On-demand provisioning](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/provision-on-demand)
- [SAML claims customization](https://learn.microsoft.com/en-us/entra/identity-platform/saml-claims-customization)
- [SAML enterprise application setup](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-setup-sso)

### Okta
- [Import users to Office 365 using Microsoft Graph](https://help.okta.com/en-us/Content/Topics/Apps/Office365-Deployment/import-ms-graph.htm)
- [Make Entra ID an identity provider in Okta](https://help.okta.com/en-us/Content/Topics/Provisioning/azure/azure-identify-identity-provider.htm)
- [Create the Okta enterprise app in Entra](https://help.okta.com/en-us/Content/Topics/Provisioning/azure/azure-create-enterprise-app.htm)
- [Provision users to Office 365](https://help.okta.com/oie/en-us/content/Topics/apps/office365-deployment/provision-users.htm)

### SailPoint
- [Integrating SailPoint with Microsoft Entra SaaS](https://documentation.sailpoint.com/connectors/saas/msentraid/help/saas_connectivity/microsoft_entra_id/integrating_sailpoint_microsoft_entra_id.html)
- [Aggregation and filter settings](https://documentation.sailpoint.com/connectors/saas/msentraid/help/saas_connectivity/microsoft_entra_id/aggregation_settings.html)

### CyberArk
- [CyberArk product documentation portal](https://docs.cyberark.com/)
- [CyberArk Identity connector best practices](https://community.cyberark.com/s/article/Idira-Identity-Connector-Best-Practices)

> These references are starting points, not substitutes for the exact connector, product version, region, and licensing guide. Confirm the supported flow with your tenant administrator and vendor documentation.