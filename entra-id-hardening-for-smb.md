# Entra ID Hardening: Baseline Expectations for SMB Environments

This document outlines the identity and access controls that should be in place before any organization considers its Microsoft 365 and Entra ID environment adequately hardened. These are not aspirational targets. They are the minimum baseline. Most SMB environments are missing at least half of them, and adversaries know it.

The recommendations below are organized by domain. Within each section, items are ordered by impact and practicality.

---

## Where to Start: Recommended Sequencing

No environment completes this list overnight, and attempting everything in parallel usually means nothing gets done well. The sequencing below produces the most immediate risk reduction and ensures each layer supports the one that follows.

1. Confirm licensing and disable Security Defaults if Entra ID P1 or P2 is present.
2. Block legacy authentication protocols.
3. Enforce MFA for all users with no network-location exclusions.
4. Review and repair conditional access policies, starting with exclusion lists.
5. Configure and verify emergency access (break glass) accounts.
6. Activate PIM for tier-zero roles and convert permanent assignments to eligible.
7. Audit application permissions and service principal privileges.
8. Lock down Entra ID default settings for consent, guest access, app registration, and tenant creation.
9. Enforce device compliance and enable VBS and Credential Guard.
10. Verify logging coverage and configure alerts.

The controls in this document are not ceiling targets. They are the floor. Once they are in place and verified, the conversation about what comes next becomes more productive.

---

## Security Defaults vs. Conditional Access

This is the most common configuration mistake in SMB tenants, and it is worth addressing before anything else because getting it wrong undermines every control that follows.

Security Defaults are a set of baseline identity controls Microsoft enables automatically in new tenants without Entra ID P1 or P2 licensing. They were designed to provide minimum protection at no additional cost. They require MFA registration for all users, block legacy authentication, and enforce MFA for administrators. For a tenant with no other security investment, they are better than nothing.

They are not, however, adequate for any environment running business-critical workloads in 2025 and beyond. Security Defaults do not support named locations, device compliance requirements, risk-based policies, or per-application enforcement. They cannot be tuned. They cannot enforce phishing-resistant authentication methods. They do not support the granular Conditional Access policies that modern threat scenarios require.

More importantly, Security Defaults and Conditional Access are mutually exclusive. Enabling both produces unpredictable enforcement behavior. When Conditional Access policies are active, Security Defaults must be off.

- If the tenant has Entra ID P1 or P2 licensing (included in Microsoft 365 Business Premium, E3, E5, and several other SKUs), disable Security Defaults and manage authentication enforcement exclusively through Conditional Access policies. Security Defaults were designed for tenants with no premium identity licensing. Once P1 or P2 is in scope, they are the wrong tool for the job.
- Confirm the licensing state before disabling Security Defaults. Disabling them without functional Conditional Access policies in place removes all baseline enforcement and leaves the tenant unprotected.
- Verify the current state in the Entra ID portal under Properties. If Security Defaults show as enabled and Conditional Access policies also exist, resolve the conflict before proceeding with any other hardening work.

---

## Legacy Authentication Blocking

Legacy authentication protocols do not support modern MFA. An attacker who identifies a mailbox or account that accepts legacy authentication can bypass every Conditional Access policy in the tenant, because those protocols authenticate before Conditional Access has a chance to evaluate the session.

The protocols in scope are SMTP AUTH (when used for client submission outside of approved relay scenarios), POP3, IMAP, Exchange ActiveSync with basic credentials, and any application using basic authentication against Exchange Online or Entra ID directly.

- Before blocking, use the Sign-in Logs workbook in the Entra ID portal to identify accounts and applications currently authenticating via legacy protocols. Filter sign-in logs by client app and look for entries showing "Exchange ActiveSync," "Other clients," "IMAP," or "POP3." Blocking without this review will generate immediate helpdesk tickets.
- Create a Conditional Access policy targeting all users, all cloud apps, and the "Other clients" client app condition, with Grant set to Block. This is the most direct and auditable way to enforce the block.
- Address any legitimate exceptions before enforcing. Multifunction printers and legacy line-of-business applications that use SMTP AUTH for outbound mail are the most common. Migrate them to Modern Authentication or dedicated relay connectors. They are not a valid reason to leave legacy auth open for the entire user population.
- After enforcement, monitor sign-in logs for blocked legacy auth attempts. A spike in blocks after rollout is expected and informational. Persistent blocks from known applications indicate a migration gap, not a policy problem.

---

## Multi-Factor Authentication

- Enforce MFA for every user account with no exceptions based on network location. A Conditional Access policy that skips MFA for users on the corporate network is not an MFA policy; it is a partial one. Assume breach applies to the internal network.
- Eliminate SMS-based MFA as the primary method. Push users toward Microsoft Authenticator with number matching, or toward phishing-resistant methods (Windows Hello for Business, FIDO2 security keys, passkeys) as soon as device refresh allows.
- Do not configure MFA exclusion groups for convenience. Emergency access accounts are the only legitimate carve-out, and those accounts should be monitored with alerts on every use.
- Require phishing-resistant MFA specifically for any account assigned a privileged role. Authenticator push approval is not sufficient for Global Administrator, Privileged Role Administrator, or equivalent tier-zero roles.

---

## Emergency Access (Break Glass) Accounts

Break glass accounts are a pair of cloud-only accounts held outside normal identity governance processes specifically so that administrative access to the tenant can be recovered if every other privileged account becomes inaccessible. A locked-out global administrator with no break glass account has no recovery path without engaging Microsoft Support, which is slow and not guaranteed.

Configure these accounts correctly or they will not work when needed.

- Create exactly two break glass accounts. Use cloud-only accounts on the tenant's `*.onmicrosoft.com` domain. Do not synchronize them from on-premises Active Directory and do not federate them through any external identity provider.
- Assign Global Administrator permanently and actively to both accounts. These are the only two accounts in the tenant that should hold permanent active Global Administrator rights outside of PIM.
- Exclude both accounts from all Conditional Access policies, including MFA policies. The purpose of these accounts is emergency recovery; if they are subject to MFA policies that depend on systems that may be unavailable, they cannot serve that purpose.
- Use long, randomly generated passphrases (24 characters or more). Store credentials offline in a physically secured location, such as a sealed envelope in a fireproof safe. Do not store them in a password manager that is itself tied to the tenant.
- Configure an alert in Microsoft Sentinel, Microsoft Defender XDR, or the Entra ID portal that fires on any sign-in from either account. A break glass sign-in outside of a documented emergency is an incident.
- Test both accounts at least annually. Verify that the credentials work and that the alert fires.

---

## Conditional Access Policy Review

Conditional Access is where most environments fail silently. The policy title looks right; the configuration does not hold up under scrutiny. Review every policy against the following:

- Open each policy and audit the exclusion list. A policy excluding a group of 200 users for "compatibility" is not a meaningful control.
- Verify that policies apply to all users, not a subset assumed to cover everyone.
- Implement a named location policy that blocks or requires additional verification from countries where the organization has no presence. For most SMBs, this is every country outside the United States.
- Configure step-up MFA requirements for applications handling sensitive data. HR systems, finance platforms, and administrative portals should require MFA on every session regardless of device compliance state. This is the operational foundation of zero trust applied to a real SMB environment.
- Require MFA for device registration and Entra join through Conditional Access, not just device settings. The two controls behave differently under edge cases; the Conditional Access policy is the authoritative one.

---

## Microsoft Secure Score

Microsoft Secure Score is a dashboard in the Microsoft Defender portal that measures the security posture of a Microsoft 365 environment against a set of recommended actions. It assigns a numeric score and breaks recommended actions down by category, effort level, and licensing requirement.

It is a useful prioritization tool. It is not a compliance certification and it does not reflect risk in a nuanced way. A tenant can have a high Secure Score and still have a misconfigured Conditional Access exclusion that undermines everything. Use it as a starting point for identifying gaps, not as a measure of overall security maturity.

- Review Secure Score early in an engagement to get a quick read on which recommended actions are missing. The "Identity" category is most relevant to the controls in this document.
- Filter recommended actions by licensing tier to avoid acting on recommendations that require licenses not in scope.
- Do not treat Secure Score improvements as the goal. Treat the underlying controls as the goal. The score follows from correct configuration; chasing the number independently leads to checkbox behavior.
- Use the comparison view to see how the tenant's score compares to similarly sized organizations. This is useful context for client conversations about relative risk posture.

---

## Entra ID Default Settings

Microsoft ships several defaults that are too permissive for any environment running business-critical workloads. These should be changed during initial tenant configuration and verified on existing tenants.

- **User application registration:** Disable the ability for non-admin users to register applications. When a user registers an application, they become its owner. If that application later receives highly privileged Graph API permissions, a standard user now controls a privileged service principal. Assign the Application Developer role to specific individuals who legitimately need to register apps.
- **Tenant creation:** Disable the ability for non-admin users to create new tenants. Unmanaged tenants created by employees are invisible to the parent organization, may accumulate sensitive data and integrations, and are difficult to recover if the creator leaves. Microsoft now provides tenant governance discovery in Entra; use it to identify what already exists.
- **Application consent:** Set user consent to disabled or, at minimum, restrict it to Microsoft-verified publishers only. Illicit consent grant attacks are not new, and the default permitting broad user consent continues to be exploited. Admin consent workflows should be configured so requests are reviewed rather than silently denied.
- **Guest user access restrictions:** Set guest access to the most restrictive option, which limits guests to reading their own directory objects only. Guests should not be able to enumerate users, groups, or other directory resources.
- **Guest invitations:** Disable the ability for guest users to invite additional guests. Only specific administrator roles should hold this capability.

---

## External Sharing Controls

SharePoint Online, OneDrive for Business, and Microsoft Teams all have sharing settings that operate independently of Conditional Access and Entra ID guest controls. Misconfigured sharing settings are one of the most common paths to unintended data exposure in M365 environments, and they are frequently overlooked in identity-focused reviews.

- Set the tenant-level external sharing policy in the SharePoint Admin Center to "Existing guests" or "Only people in your organization" unless the business has a documented and specific need for broader sharing. The "Anyone with the link" setting allows unauthenticated access to any document shared with that link type, with no audit trail tied to an identity.
- Understand the distinction between "Anyone with the link" sharing and authenticated external sharing. The former requires no sign-in and cannot be revoked by removing a guest account. The latter creates an auditable guest identity. Only allow unauthenticated link sharing if there is a clear business justification and a defined review process.
- Verify that OneDrive sharing settings match the SharePoint tenant-level policy. OneDrive has its own sharing controls and can be configured more permissively than the tenant default if not explicitly locked down.
- Review Microsoft Teams external access (federation with other Teams tenants) and guest access (guests in Teams channels) separately. External access allows real-time communication with users in other tenants; guest access allows those users to participate in channels and access shared files. Both should reflect deliberate policy decisions, not defaults.
- If Microsoft Purview sensitivity labels are in scope, configure label policies to restrict external sharing for labels applied to sensitive content. This provides a data-aware enforcement layer that SharePoint-level settings alone do not offer.

---

## Privileged Role Management

This is the area where organizations feel confident and are most commonly wrong. Role membership reviews that look only at active assignments miss the full picture.

Two terms are important here. An **active** role assignment means the account holds that role right now and can exercise its permissions immediately. An **eligible** role assignment means the account can request activation of that role through PIM but does not hold it continuously. Eligible assignments are the correct state for all privileged accounts except emergency access accounts.

- **Use Privileged Identity Management (PIM) for all privileged roles.** Every role assignment should be eligible, not permanently active. The only accounts that should hold permanent active Global Administrator rights are the two emergency access accounts. Every other privileged user activates their role on demand through PIM.
- **Configure approval-based activation for tier-zero roles.** For Global Administrator, Privileged Role Administrator, and Privileged Authentication Administrator, require a second person to approve activation. This means a compromised account cannot self-elevate without triggering an approval request that alerts the team.
- **Query both active and eligible members when auditing role assignments.** These require separate API calls. A membership report showing zero permanent Global Administrators is incomplete if it does not also surface who holds eligible assignments.
- **Treat Application Administrator and Cloud Application Administrator as tier-zero.** These roles can add credentials to existing service principals. If a service principal holds `Directory.ReadWrite.All` or `RoleManagement.ReadWrite.Directory`, the application administrator effectively controls those permissions. This escalation path is the most commonly overlooked gap in otherwise well-configured environments.
- **Provision separate administrative accounts.** Privileged role assignments should be on dedicated accounts, not the accounts used for daily work. At minimum, these accounts should use a separate MFA registration and, ideally, should be cloud-only accounts not synchronized from on-premises Active Directory.

---

## Identity Protection Risk Policies

Entra ID Identity Protection (available with Entra ID P2 licensing, included in Microsoft 365 Business Premium and E5) evaluates every sign-in and user account for behavioral and contextual risk signals. It surfaces two types of policy: sign-in risk policies, which evaluate the risk of a specific authentication event, and user risk policies, which evaluate whether an account itself shows signs of compromise over time.

Neither policy type does anything useful without action thresholds configured. Review and configure both.

- **Sign-in risk policy:** Set the sign-in risk threshold to Medium and above. The recommended response is to require MFA. High-risk sign-ins (impossible travel, anonymous IP, token anomalies) should trigger MFA at minimum and can be configured to block access entirely depending on the organization's tolerance.
- **User risk policy:** Set the user risk threshold to High. The recommended response is to require a secure password reset. A user risk flag at high confidence means the account shows signs of credential compromise; forcing a password reset with MFA verification breaks an attacker's persistence on that account.
- Exclude break glass accounts from both policies. These accounts must remain accessible regardless of risk signals.
- Review the Risky Users and Risky Sign-ins reports in the Entra ID portal regularly. Identity Protection surfaces detections that warrant investigation even when automated policy responses are not triggered.
- Risky sign-in and risky user events should flow into the SIEM. In Sentinel, the Entra ID data connector surfaces these as security incidents that can be triaged alongside other alerts.

---

## Application Permissions Audit

Overprivileged service principals are the single most consistent finding in Entra assessments, including environments that have handled everything else correctly. This should be treated as a recurring audit item, not a one-time task.

Understanding what these permissions actually grant is necessary context before reviewing them. `Directory.ReadWrite.All` allows an application to read and write every object in the directory, including users, groups, and roles, without any user interaction. `RoleManagement.ReadWrite.Directory` allows an application to read and modify role assignments, meaning it can make itself or any other identity a Global Administrator. `AppRoleAssignment.ReadWrite.All` allows an application to grant itself or other applications any permission in the tenant. Any service principal holding one or more of these permissions has effective control over the entire tenant and should be treated accordingly.

- Enumerate all service principals and their assigned Graph API permissions. Pay specific attention to `Directory.ReadWrite.All`, `RoleManagement.ReadWrite.Directory`, `AppRoleAssignment.ReadWrite.All`, and any permission granting the ability to modify its own or others' permissions.
- Any service principal holding tier-zero Graph permissions should be treated with the same scrutiny as a Global Administrator account. Review the owner, the credential expiry, and the application registration behind it.
- Review service principals created by your licensing provider or prior managed service provider. A service principal with full read-write permissions deployed by a vendor for support access is not inherently malicious, but it is a standing privileged account that the organization often does not know exists. Identify it, understand it, and verify it is still necessary.
- Ensure that alerts exist for new high-privilege application consent grants. In Sentinel, the Entra ID audit log connector surfaces consent events; alert rules targeting `Add app role assignment to service principal` with high-privilege permission values are a reasonable starting point. A consent event at that tier should never be invisible.

---

## Device Security

- Enable Virtualization Based Security (VBS) and Credential Guard via Intune configuration profile. These controls protect the Primary Refresh Token (PRT) using the device's Trusted Platform Module (TPM), making token extraction significantly harder. This capability has been available since Windows 10 and TPM 2.0 hardware is standard on any device manufactured in the last several years.
- Enforce TPM 2.0 as a device compliance requirement in Intune. Devices without a TPM cannot protect the PRT in the same way and should not be granted access to organizational resources under the same policy terms.
- Disable the ability for standard users to join their own devices to Entra. Use Autopilot for provisioning so devices arrive in a managed state. If Autopilot is not yet in scope, scope device join to a specific administrative account or group through device settings.
- Require compliant device status as a Conditional Access condition for access to sensitive applications. Compliance status without enforcement at the access layer is a reporting tool, not a control.

---

## Administrative Access Hygiene

- Do not administer Entra ID from the same browser session used for general web browsing. Tokens stored in a browser profile can be extracted if that session is compromised. The minimum acceptable practice is a dedicated browser profile used exclusively for administration. A cloud-based admin VM or Azure Virtual Desktop is better. A physically separate device used only for privileged access is the correct long-term target.
- Global admins and other tier-zero role holders should authenticate with accounts that are not synchronized from on-premises Active Directory. A cloud-only account on an `@*.onmicrosoft.com` domain, with its own MFA registration independent of any federated identity system, limits the blast radius if the on-premises environment is compromised.
- Review Granular Delegated Admin Privileges (GDAP) relationships and remove any legacy Delegated Admin Privileges (DAP) relationships that remain. DAP grants partner organizations Global Administrator and Helpdesk Administrator standing access to the tenant. GDAP allows time-limited, role-scoped access. Any MSP or licensing partner that has not migrated to GDAP should be asked when they intend to do so.

---

## Logging and Monitoring

- Verify that Unified Audit Log ingestion is active and that sign-in logs, audit logs, and risky user and risky sign-in events are flowing to your SIEM or security operations platform. Licensing tier determines which log sources are available; confirm what you have before assuming coverage.
- Configure Microsoft Sentinel native connectors for Entra ID if Sentinel is in scope. The Entra ID connector maps available log sources automatically and reduces configuration complexity. The specific tables populated include `SigninLogs`, `AuditLogs`, `AADRiskyUsers`, and `AADUserRiskEvents`.
- Ensure alerts are configured for the following at minimum: activation of any tier-zero role, emergency access account sign-in, new application consent granted, new service principal created with high-privilege permissions, and new device registered outside of the Autopilot workflow. In Sentinel, these are implemented as Scheduled Analytics Rules against the Entra ID log tables. In Microsoft Defender XDR, equivalent alerts can be configured under Custom Detection Rules.
- Do not assume logs are being ingested because the diagnostic setting is configured. Verify that records are appearing in the SIEM on a regular interval and that alert rules are firing on test events.
