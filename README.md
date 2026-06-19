# App registrations, Application objects, and Service principals — Entra ID (SC-300 notes)

> Notes: differences between application objects and service principals in Microsoft Entra ID, how they relate, why you register apps, benefits, and traditional alternatives.

## Table of Contents

- [Overview](#overview)
- [Application object (what it is)](#application-object)
- [Service principal (what it is)](#service-principal)
- [How they relate — lifecycle and mapping](#relationship)
- [Purpose of implementing an app registration in Entra ID](#purpose)
- [Benefits of app registrations](#benefits)
- [Traditional approaches and what changed](#traditional-approaches)
- [Practical guidance & checklist for app registration](#practical-guidance)
- [Resources and further reading](#resources)

---

## Overview {#overview}

When you register an application in Microsoft Entra (Azure AD), you create two related concepts:

- An *application object* — the global template that describes the app's identity, available permissions, and configuration.  
- A *service principal* — the tenant-specific instance used to represent and grant access to the application inside a given tenant.

TLDR: Application object = app definition (one per app across Entra), service principal = tenant-specific runtime identity (one per tenant where the app is used).

---

## Application object {#application-object}

What it is:
- The application object is the global programmatic description of an application stored in the Microsoft Entra directory of the app publisher’s home tenant. It contains metadata such as the app's name, redirect URIs, API permissions (scopes), certificates/secrets (credential definitions), logo, and other configuration.
- There is typically one application object per application (per publisher) — it’s the reusable definition that can be provisioned into multiple tenants.

Where it lives:
- In the publisher's Entra tenant (the tenant that registered the app) as the canonical definition.

Common properties on the application object:
- Application (client) ID — stable GUID identifying the application definition.  
- Exposed scopes and app roles — API permissions this app exposes.  
- Redirect URIs, logout URIs, implicit settings.  
- Owners and publisher metadata.

TLDR: The application object is the centralized app definition (the template) that describes how the app behaves and which permissions it requests or exposes.

---

## Service principal {#service-principal}

What it is:
- A service principal is the representation (instance) of the application in a specific tenant where it runs or is consumed. It contains the runtime configuration used for authentication and authorization in that tenant.
- When an application object is used in a tenant different from the publisher tenant (multi-tenant app), Entra creates a service principal in the consumer tenant. For single-tenant apps, the publisher tenant has both the application object and a service principal representing that app.

Where it lives:
- In each tenant that uses or consents to the app — service principals are tenant-scoped.

Common properties on the service principal:
- Object ID (service principal id) — tenant-scoped GUID.  
- AppId — references the application object's client ID.  
- Role assignments, permissions granted, consent records, and local configuration (e.g., conditional access applied to the service principal).  

TLDR: The service principal is the tenant-specific instance that grants the app an identity in that tenant and stores consent/permission state.

---

## How they relate — lifecycle and mapping {#relationship}

- Create an app registration → Entra creates an application object in the publisher tenant.  
- When the same application is used in another tenant, a corresponding service principal is created in the consumer tenant (via consent or admin provisioning).  
- The application object’s client (app) ID is referenced by each service principal via its AppId property.  
- You can think of the application object as the code-level identity; the service principal is the runtime identity used by the tenant to grant access and apply policies.

Important lifecycle notes:
- Deleting the application object removes the global definition; service principals in other tenants remain but lose their link if the app is unpublished.  
- Credential rotation should be managed on the application object (publisher tenant) and consumers must acquire updated credentials or rely on federated flows.

TLDR: App object = template; service principals = per-tenant instances created from that template, holding consent and local settings.

---

## Purpose of implementing an app registration in Entra ID {#purpose}

Why admins and developers register apps:
- To enable OAuth2/OpenID Connect authentication for applications (user or daemon flows).  
- To centrally declare and expose APIs (scopes) and app roles that other apps and services can request.  
- To manage credentials (client secrets, certificates) used by applications to authenticate.  
- To control authorization and consent (who can access the app and which permissions are granted).  

Practical operational purposes:
- Integration with Microsoft identity platform for SSO, tokens, and managed identities.  
- Support for daemon/service-to-service authentication (client credentials flow).  
- Enabling Conditional Access and session controls for app sign-ins.

TLDR: App registrations let you give applications a first-class identity in Entra so they can authenticate, request tokens, and participate in centralized access control and consent flows.

---

## Benefits of app registrations {#benefits}

Security and control:
- Centralized credential management (rotate secrets/certificates).  
- Apply Conditional Access and risk policies to app sign‑ins or service principals.  
- Least‑privilege by scoping permissions to the minimum required (scopes/app roles).

Operational & development benefits:
- Standard OAuth2/OpenID Connect flows across apps — simpler integration and SSO.  
- Reuse definitions (one app object, many service principals) for multi‑tenant SaaS scenarios.  
- Auditability — sign-in logs, consent records, and permission grants are tracked in Entra.

Governance benefits:
- Tenant admins control which third‑party or publisher apps receive consent (admin consent vs user consent).  
- Ability to discover and inventory enterprise app service principals for compliance and lifecycle management.

TLDR: App registrations provide security, governance, and operational benefits — centralized identity, auditable consent, and policy enforcement.

---

## Traditional approaches (what organizations used before modern app registrations) {#traditional-approaches}

- Hard-coded credentials: storing application passwords or service account credentials in config files, scripts, or AD accounts (high risk).  
- VPN/Network trust: assuming network location equals trust — apps accessible only from on-prem networks or via VPN instead of identity.  
- Reverse-proxies and AD FS federation: organizations used AD FS or reverse proxies to publish applications and handle SSO, often with complex infrastructure.  
- Service accounts in on‑prem AD: using managed service accounts or AD user accounts for applications, which led to elevated lateral‑movement risk if credentials leaked.

TLDR: Before app registrations, teams relied on network trust, service accounts, or custom SSO stacks — less centralized, harder to audit, and more risky.

---

## Practical guidance & checklist for app registration {#practical-guidance}

Checklist for admins before registering an app:
1. Determine app type: single-tenant, multi-tenant, or external (multi-tenant SaaS).  
2. Choose authentication flow: authorization code, client credentials, device code, etc.  
3. Define least-privilege permissions and app roles; avoid over-scoping.  
4. Decide ownership and secret/certificate lifecycle: who rotates credentials, expiration policy.  
5. Plan conditional access and logging: which policies should apply and monitoring requirements.  
6. Test in a development tenant and validate consent flows (user vs admin consent).  
7. Onboard to inventory/governance: record app object IDs and service principal IDs for audits.

TLDR: Plan app type, auth flow, permissions, credential lifecycle, and governance before registering; test in dev and enforce least-privilege.

---

## Resources and further reading {#resources}

- Microsoft Learn: Plan your line-of-business application registration strategy  
  https://learn.microsoft.com/en-us/training/modules/implement-app-registration/2-plan-your-line-business-application-registration-strategy
- Microsoft docs: Application objects and service principals  
  https://learn.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals

*Last updated: 2026 — SC-300 notes*
