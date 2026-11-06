# Microsoft 365 & IAM Portfolio

**Oluwaseun Olasode | Modern Workplace Engineer | Microsoft 365, Entra ID & Identity Security**

This repository contains technical portfolio documentation, architecture decisions, implementation plans, test scenarios, troubleshooting guidance, and security controls focused on Microsoft cloud administration and Identity & Access Management (IAM).

## Portfolio areas

- Microsoft Entra ID: identity lifecycle, enterprise applications, SAML SSO, Conditional Access concepts, and identity governance foundations.
- Microsoft Intune & Microsoft 365: endpoint administration, device compliance, and modern workplace operations.
- IAM / IGA / PAM: identity source-of-authority, provisioning boundaries, access governance, privileged access controls, and joiner-mover-leaver (JML) scenarios.
- Automation: operational runbooks and opportunities for PowerShell-driven administration.

## Featured project

### IAM integration design: Entra ID + Okta + SailPoint ISC + CyberArk

The featured runbook describes a controlled fictional-organization lab design covering:

1. Entra ID pilot preparation and SAML enterprise application setup.
2. Entra-to-Okta Microsoft Graph user import and inbound SAML federation.
3. SailPoint Identity Security Cloud aggregation, correlation, and governance.
4. A CyberArk product decision gate and product-specific integration approach.
5. End-to-end lifecycle testing, troubleshooting, security guardrails, and rollback.

**Project status:** This is an implementation blueprint / lab plan. Individual tasks are not represented as completed tenant implementations unless accompanied by clearly labelled evidence. The UI references in the source material are illustrative guides, not screenshots from a live tenant. CyberArk steps must be selected for the exact product, edition, and version.

[Open the IAM integration runbook](docs/iam-integration-runbook.md)

## Repository structure

```text
.
├── README.md
└── docs/
    └── iam-integration-runbook.md
```

## Evidence and security standards

- Use fictional identities and isolated test tenants for demonstrations.
- Never publish tenant IDs, real user data, client secrets, certificates, access tokens, recovery codes, or privileged account details.
- Clearly distinguish **implemented**, **tested**, **simulated**, **planned**, and **not tested** work.
- Include sanitized logs and screenshots only when they are genuinely captured from a lab and safe to disclose.
- Do not describe a design document as a production deployment or claim a test passed without evidence.

## About

I focus on Microsoft 365 administration, Microsoft Entra ID, Intune, endpoint management, identity security, and practical IAM/IGA/PAM learning. This portfolio is intended to demonstrate structured technical thinking, safe implementation practices, and clear operational documentation.

## Contact

For professional enquiries, please use the contact details on my LinkedIn profile: [Oluwaseun Olasode](https://www.linkedin.com/in/olosode-oluuwaseun-3a98b99b/).
