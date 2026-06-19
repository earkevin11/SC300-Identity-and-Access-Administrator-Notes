# Microsoft Entra Application Proxy

> Readme: Integrate on‑premises applications with Microsoft Entra Application Proxy (formerly Azure AD Application Proxy).

## Table of Contents

- [TLDR](#tldr)
- [Source and further reading](#source-and-further-reading)
- [Why implement Microsoft Entra Application Proxy](#why-implement-microsoft-entra-application-proxy)
- [Core use cases](#core-use-cases)
- [Benefits (business and technical)](#benefits-business-and-technical)
- [How it works (high level)](#how-it-works-high-level)
- [Key concepts](#key-concepts)
- [When to use Application Proxy vs alternatives](#when-to-use-application-proxy-vs-alternatives)
- [Traditional scenarios (what orgs used before App Proxy)](#traditional-scenarios-what-orgs-used-before-app-proxy)
- [Limitations and considerations](#limitations-and-considerations)
- [Implementation checklist (quick)](#implementation-checklist-quick)
- [Practical use cases & scenarios](#practical-use-cases--scenarios)
- [Primary benefit and context](#primary-benefit-and-context)
- [TLDR (expanded with contrasts)](#tldr-expanded-with-contrasts)
- [Example resources and links](#example-resources-and-links)


<a id="tldr"></a>
## TLDR

Entra Application Proxy publishes on‑prem web apps securely through Microsoft Entra so users can access them with SSO and Conditional Access — instead of opening inbound firewall ports or granting broad VPN access.

<a id="source-and-further-reading"></a>
## Source and further reading

- Microsoft Learn: Integrate on-premises apps using Azure AD Application Proxy
  - https://learn.microsoft.com/en-us/training/modules/implement-monitor-integration-of-enterprise-apps-for-sso/4-integrate-premises-apps-use-azure-active-directory-application-proxy

<a id="why-implement-microsoft-entra-application-proxy"></a>
## Why implement Microsoft Entra Application Proxy

- Publish on‑prem web apps to remote users and partners without exposing the apps directly to the internet.  
- Provide SSO and centralized access controls (Conditional Access, MFA) for legacy apps that don’t natively support modern auth.  
- Reduce operational overhead compared to managing VPNs, reverse proxies, or complex network changes.

<a id="core-use-cases"></a>
## Core use cases

- Remote employees need access to internal web apps (intranets, HR portals, reporting tools).  
- Partners or external contractors require scoped access to specific web apps without issuing direct network access.  
- Legacy/internal apps that lack modern authentication but can be fronted by Entra for SSO and access control.  
- Rapidly publishing apps during migrations or cloud adoption pilots without rehosting or reengineering the app.

<a id="benefits-business-and-technical"></a>
## Benefits (business and technical)

- **Identity‑first access**: enforce Conditional Access policies, MFA, and device compliance for on‑prem apps.  
- **No inbound firewall openings**: connectors make outbound connections to Microsoft so you avoid opening inbound ports.  
- **Simpler operations**: fewer VPN tunnels and less network configuration; central policy in Entra.  
- **SSO for legacy apps**: support single sign-on and modern session controls even for apps without native modern auth.  
- **Auditability & compliance**: per-user sign-in logs and access events linked to identities for investigations and reporting.

<a id="how-it-works-high-level"></a>
## How it works (high level)

1. Install the **Application Proxy connector** on one or more Windows servers in the internal network.  
2. The connector establishes an outbound TLS connection to the Entra Application Proxy service in Microsoft’s cloud (no inbound firewall rule required).  
3. In Entra, register an enterprise application and configure internal URL and external URL mapping; set preauthentication mode (Azure AD or Passthrough).  
4. When a remote user requests the external URL, Entra evaluates Conditional Access and preauthentication; approved traffic is sent through the connector to the backend app.

<a id="key-concepts"></a>
## Key concepts

- **Connector**: on‑prem agent that creates outbound tunnels to Entra and proxies approved requests.  
- **Preauthentication**: Entra can preauthenticate users (Azure AD) before forwarding requests, or allow passthrough authentication to the app.  
- **External URL**: the published endpoint users access.  
- **Internal URL**: the backend app address reachable by the connector.

<a id="when-to-use-application-proxy-vs-alternatives"></a>
## When to use Application Proxy vs alternatives

- Use Application Proxy when you need to publish web apps quickly with Entra SSO and Conditional Access but want to avoid opening inbound firewall ports — instead of placing users on a VPN or rehosting apps.  
- Prefer GSA or network tunnels when you need network‑level connectivity (L4) or non‑HTTP protocols — Application Proxy is HTTP(S) focused.  
- Use Entra App Proxy over a traditional reverse proxy when you want cloud‑based preauthentication, centralized Conditional Access, and simplified connector‑managed outbound connectivity.

<a id="traditional-scenarios-what-orgs-used-before-app-proxy"></a>
## Traditional scenarios (what orgs used before App Proxy)

- Reverse proxies / WAP: configure an on‑prem reverse proxy and open inbound ports; maintain TLS certs and external DNS.  
- Publish directly via DMZ with NAT/firewall rules — higher exposure and operational burden.  
- Broad VPN access: give remote users network access to reach internal apps (coarse trust, higher blast radius).

<a id="limitations-and-considerations"></a>
## Limitations and considerations

- Application Proxy primarily supports **web (HTTP/HTTPS)** apps — not arbitrary TCP/UDP protocols.  
- For high throughput or large file transfers, evaluate connector placement, HA, and network capacity.  
- Choose preauthentication mode carefully: **Azure AD preauth** provides the best security posture but may require app adjustments.  
- Plan connector redundancy (multiple connectors and servers) and keep connectors updated.

<a id="implementation-checklist-quick"></a>
## Implementation checklist (quick)

- Identify apps suitable for App Proxy (HTTP/HTTPS, internal URL reachable by connector).  
- Prepare servers for connector installation (Windows Server, outbound TLS allowed).  
- Configure enterprise application in Entra and set external/internal URLs.  
- Configure Conditional Access (MFA, device compliance, session controls) targeting the published app.  
- Test in a pilot, monitor sign-ins and connector logs, then roll out.

<a id="practical-use-cases--scenarios"></a>
## Practical use cases & scenarios

### TL;DR
- Primary benefit: publish on‑prem web apps using identity‑first controls — SSO + Conditional Access + MFA — without opening inbound firewall ports or giving broad network/VPN access.

### 1) Remote employees need convenient, secure access
- Example: intranet, HR portal, BI/reporting dashboards.  
- Why: quick publish, SSO, enforce MFA/device compliance, no VPN client required for many users.

### 2) Contractors, partners, or vendors with scoped access
- Example: external accountants access payroll app for a fixed engagement.  
- Why: time‑bounded guest accounts, per‑app CA policies, avoid creating VPN accounts or broader network access.

### 3) Legacy apps lacking modern auth
- Example: older apps using forms auth or NTLM.  
- Why: App Proxy enables SSO patterns (preauth, passthrough, or KCD) so you can secure access without rewriting the app.

### 4) Migrations, pilots, and fast publishing
- Example: pilot cloud migration by publishing the legacy app via App Proxy while planning rehosting.  
- Why: minimal network changes, fast to deploy, controlled access.

### 5) Scoped admin or vendor access (least‑privilege)
- Example: external support requiring access to an internal admin console.  
- Why: give only the app endpoint, enforce strict CA policies and logging.

### 6) Replace fragile reverse proxies or DMZ publishes
- Example: aging WAP with exposed ports and cert maintenance.  
- Why: connectors make outbound TLS connections (no inbound holes), reduce cert/dns maintenance.

### When NOT to use App Proxy
- Non‑HTTP(S) protocols (RDP, SMB, arbitrary TCP/UDP) — use GSA or network tunnels.  
- Very high throughput/large file transfers without testing connector capacity.  
- Need full network-level connectivity for clients — use VPN/site‑to‑site for L3/L4 access.

<a id="primary-benefit-and-context"></a>
## Primary benefit and context

App Proxy’s core value is moving trust from “on the network” to “who the identity is and the state of the device.” Instead of saying “if you’re on the corporate IP you’re allowed,” you enforce per‑user, per‑device, per‑app controls in Entra (MFA, device compliance, session controls). This:

- Reduces attack surface (no inbound holes, no broad network access).  
- Centralizes enforcement and auditing via Entra (one place to require MFA, block risky sign‑ins, enable session controls).  
- Improves user experience (SSO, often no VPN client required).

<a id="tldr-expanded-with-contrasts"></a>
## TLDR (expanded with contrasts)

- App Proxy grants **identity‑first SSO and Conditional Access** for on‑prem web apps — instead of opening inbound ports and relying on network perimeter controls.  
- App Proxy publishes HTTP(S) apps via **outbound connectors** — instead of requiring VPN tunnels or DMZ NAT.  
- App Proxy enforces preauthentication in Entra — instead of leaving authentication solely to legacy app forms or NTLM-only flows.

<a id="example-resources-and-links"></a>
## Example resources and links

- Microsoft Learn module: Integrate on-premises apps using Azure AD Application Proxy — https://learn.microsoft.com/en-us/training/modules/implement-monitor-integration-of-enterprise-apps-for-sso/4-integrate-premises-apps-use-azure-active-directory-application-proxy  
- Entra Application Proxy overview: https://learn.microsoft.com/en-us/azure/active-directory/app-proxy/

*Last updated: 2026 — Notes for integrating on‑prem apps with Microsoft Entra Application Proxy*
