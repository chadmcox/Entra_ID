# Identity Protection
* This does require Azure AD Premium 2
* Navigate to the  [Identity Protection Pane](https://portal.azure.com/#blade/Microsoft_AAD_IAM/IdentityProtectionMenuBlade/Overview)

## Notes
Clear all old user risk prior to enabling any user risk based policy:  
* [Scripts I wrote to help with this](https://github.com/chadmcox/Azure_Active_Directory/tree/master/Identity%20Protection)
* Old user risk could lead to unexpected behavior when users signs-in.  Ideally all low and medium risk will be cleaned up via script and all high should be reviewed and manually remediated.
* Starting March 31st, 2024, all "low" risk detections and users in Microsoft Entra ID Identity Protection that are older than 6 months will be automatically aged out and dismissed. This will allow customers to focus on more relevant risk and provide a cleaner investigation environment.

Make sure all trusted networks have been identified and created in a trusted name location
* This stops the majority of false positives
* Use the workbook: [Workbook: Impact analysis of risk-based access policies](https://learn.microsoft.com/en-us/entra/id-protection/workbook-risk-based-policy-impact) to identify non trusted networks that are used by multiple users.  Work with network team to make sure they should be trusted.
* If using Zscaler or some other proxy solution with shared IP egresses, refer to [Entra ID Protection stop false positive risk caused by Zscaler](https://github.com/chadmcox/Azure_Active_Directory/wiki/Entra-ID-Protection-stop-false-positive-risk-caused-by-Zscaler).

When using federated authentication it is possible to leverage multifactor authentication (supported by Okta, Ping, ADFS and others) to remediate sign-in risk.
* [Set federatedIdpMfaBehavior to enforceMfaByFederatedIdp](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-migrate-mfa-server-to-mfa-with-federation#set-federatedidpmfabehavior-to-enforcemfabyfederatedidp)
* [#AzureAD Identity Protection adds support for federated identities! - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/azuread-identity-protection-adds-support-for-federated/ba-p/244328)

To self remediate sign-in risk
* For cloud authentication user must be registered for Azure MFA
* Review Workbook - Impact analysis of risk-based access policies / Risky sign-ins remediated by multifactor authentication - shows users that remediated a sign-in risk by providing MFA.

To self remediate user risk
* Self service password reset and password write back need to be enabled for either cloud authentication or federated authentication.
* Review Workbook - Impact analysis of risk-based access policies / User risk remediated by password reset - shows if any users have changed password that reset a user risk
* MFA does not remediate user risk and could make it so that users are prompted over and over.  Consider using a block over requiring MFA.

## Things you should know!!
* Do not combine user risk policies and sign-in risk policy into the same risk based conditional access policy.  
* When combined it is not an **or** its **and**, which means both risk types have to be meet.
* MFA will not remediate a user risk, the user will probably get bombarded with prompts.
* Password change will not remediate a sign-in risk

# Identity protection recommended configuration
## Legacy identity protection policies
* User risk policy - should be Disabled
* Sign-in risk policy - should be Disabled

These are set to retire according to article May 2024 will no longer be able to enable and by July 2024 will no longer be able to modify.
[April's what's new to Entra Id](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/what-s-new-in-microsoft-entra/ba-p/3796391)

[Migrate risk policies to Conditional Access](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies#migrate-risk-policies-to-conditional-access)

## Identity protection MFA registration policy
* Assignments
  * Include: All Users
  * Exclude: Breakglass, Exclusion Group
* Controls
  * Require Azure AD MFA registration
* Enforce policy: On

## Identity protection users at risk detected alerts
* Add users that need to be notified
* Alert on user risk level at or above: Recommended High

## Identity protection weekly digest
* Add users that need to be notified
* Send weekly digest emails: Yes

## Identity protection settings
This is something you should use if you have password hash sync enable, included for federated authentication.
* Allow on-premises password change to reset user risk, select enable
* [Remediate User Risks in Microsoft Entra ID Protection Through On-premises Password Changes](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/remediate-user-risks-in-microsoft-entra-id-protection-through-on/ba-p/3773129)
* If enabled Review Workbook - Impact analysis of risk-based access policies / User risk remediated by on-premise password change - _shows the number of users that have leveraged this feature to remediate a user risk._

## Workbook - Impact analysis of risk-based access policies
* Requires Entra ID logs to be integrated with log analytics.
* This workbook provides information based on successful sign-ins for specific scenarios.
* Provides who is not being protected by a RBCA.
* Provides information about remediation or block results from potential RBCA.
* Provides untrusted networks that are being accessed by multiple users.

# Recommended risk based conditional access policies
This is my current set of recommended risk based conditional access policies. I have written several KQL queries to run in either the log analytics attached to Entra ID or Defender XDR. These queries will help make decisions on certain policies.
* [RBCA Impact](https://github.com/chadmcox/Azure_Active_Directory/tree/master/Log%20Analytics/RBCA%20Impact)
* [Identity Protection Queries](https://github.com/chadmcox/Azure_Active_Directory/tree/master/Log%20Analytics/Identity%20Protection%20Queries)
* [Defender XDR KQL](https://github.com/chadmcox/Sentinel-and-Defender-Queries/tree/main/Defender)


---

### High user risk require password change
* Use when self service password reset and password write back is enabled on Entra ID tenant. Including in federation environments
* **Should only be allowed from trusted or compliant devices.**
* User must be registered and enabled for sspr or they are blocked.
* Recommended by CIS
* [Common Conditional Access policy: User risk-based password change](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-policy-risk-user)

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users  <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | User Risk: high | Require password change | Sign-in frequency: Every time |  

Review Workbook - Impact analysis of risk-based access policies
* Impact summary of recommended risk-based access policies / High risk users not prompted for password change - Shows the number of users not being prompted to change password with a RBCA.
* Impact details: User risk scenarios / High risk users not being prompted to change password by risk-based access policy - shows list of users that would have been impacted if a RBCA was in place

---

### High user risk block all cloud apps when using untrusted or not compliant devices
* Should be used when leveraging the require password change policy. This policy will make sure users not on trusted or compliant devices will be blocked from changing their password.
* Requires for someone to research and respond to high users risk as they come in. (Usually Security Operations)
* Allow on-premises password change to reset user risk will help with user risk clean up.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users  <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | User Risk: high <br /> Device Filter: Include All Devices, <br /> Exclude isCompliant = True or <br /> TrustType = Microsoft Entra hybrid Joined | Block | Sign-in frequency: Every time |  

---

### Medium and high sign-in risk require MFA all cloud apps
* Should be used when users are registered for MFA while using cloud authentication.
* Is supported when using a federated IDP like Ping, Okta, ADFS, or Duo when enforceMfaByFederatedIdp is set
* Recommended by CIS
* [Common Conditional Access policy: Sign-in risk-based multifactor authentication](https://learn.microsoft.com/en-us/entra/identity/conditional-access/howto-conditional-access-policy-risk)

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users  <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | Sign-in Risk: high, Medium | Require Multifactor | Sign-in Frequency: Every time |  

Review Workbook - Impact analysis of risk-based access policies
* Impact summary of recommended risk-based access policies / High risk sign-ins not remediated using multifactor authentication  - Shows the number of users not being prompted to MFA by a RBCA.
* Impact details: Sign-in risk policy scenarios / High risk sign-ins not self-remediating using multifactor authentication by risk-based access policy - shows list of users that would have been impacted if RBCA was in place
---

### Low, medium and high sign-in risk block register security information
* **Use when wanting to provide most protection to the tenant.**
* Should be set even with federated authentication to protect poorly managed in cloud accounts like shared mailboxes.
* Goal is to prevent a user from being able to register security information with any kind of sign-in risk.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users   <br /> Exclude: Guest, BreakGlass  | User actions: <br /> Register Security Information  | Sign-in Risk: low, <br /> medium, <br /> high | Block | Sign-in frequency: Every time |  

---

### Low, medium and high sign-in risk block Microsoft management endpoints
* **Use when wanting to provide most protection to the tenant.**
* If any chance there is a sign-in risk when a users is trying to manage a component of Microsoft 365 or Azure they should be blocked.
* Will need to work with a peer to clear the risk.
* [Here is the list of applications affected by this policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-cloud-apps#microsoft-admin-portals)

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users <br /> Exclude: BreakGlass  | Include: Microsoft Admin Portals, <br /> Windows Azure Service Management API | Sign-in Risk: low, <br /> medium, <br /> high | Block | Sign-in frequency: Every time |  

---

### Low, medium and high sign-in risk block highly privileged role members
* **Use when wanting to provide most protection to the tenant.**
* **Privileged Role Members** = Directory Roles (Application Administrator,Authentication Administrator,Cloud Application Administrator,Conditional Access Administrator,Exchange Administrator,Global Administrator,Helpdesk Administrator,Hybrid Identity Administrator,Password Administrator,Privileged Authentication Administrator,Privileged Role Administrator,Security Administrator,SharePoint Administrator,User Administrator)
* ***Include the directory sync account - Directory Role (Directory Sync Account) or create a separate conditional access policy***
* Will need to work with a peer to clear the risk.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: Role - privileged roles   <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | Sign-in Risk: low, medium, high | Block | Sign-in frequency: Every time |  
 
---

### Low, medium and high user risk block highly privileged role members
* **Use when wanting to provide most protection to the tenant.**
* **Privileged Role Members** = Directory Roles (Application Administrator,Authentication Administrator,Cloud Application Administrator,Conditional Access Administrator,Exchange Administrator,Global Administrator,Helpdesk Administrator,Hybrid Identity Administrator,Password Administrator,Privileged Authentication Administrator,Privileged Role Administrator,Security Administrator,SharePoint Administrator,User Administrator)
* ***Include the directory sync account - Directory Role (Directory Sync Account) or create a separate conditional access policy***
* Will need to work with a peer to clear the risk.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: Role - privileged roles   <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | User Risk: low, medium, high | Block |  Sign-in frequency: Every time |  

---

### Low, medium and high sign-in risk block Microsoft Intune Enrollment
* **Use when wanting to provide most protection to the tenant.**
* Goal is to prevent a user from being able to register a device with any kind of sign-in risk.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users   <br /> Exclude: Guest, BreakGlass  | Include: Microsoft Intune Enrollment  | Sign-in Risk: low, <br /> medium, <br /> high | Block | Sign-in frequency: Every time |  

---

### High user risk block all cloud apps
* Use when self service password reset and password write back is not enabled on Entra ID tenant.
* **Use when wanting to provide most protection to the tenant.**
* Requires for someone to research and respond to high users risk as they come in. (Usually Security Operations)
* Allow on-premises password change to reset user risk will help with user risk clean up.
* Recommended by CISA

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users  <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | User Risk: high | Block | Sign-in frequency: Every time |  

Review Workbook - Impact analysis of risk-based access policies
* Impact summary of recommended risk-based access policies / High risk users not being blocked - Shows the number of users not being blocked by a RBCA.
* Impact details: User risk scenarios / High risk users not being blocked by risk-based access policy - shows list of users that would have been impacted if a RBCA was in place

---

### High sign-in risk block all cloud apps
* Should be used when users are not registered for MFA while using cloud authentication.
* **Use when wanting to provide most protection to the tenant.**
* Recommended by CISA

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All Users  <br /> Exclude: BreakGlass  | Include: All Cloud Apps  | Sign-in Risk: high | Block | Sign-in frequency: Every time |  

Review Workbook - Impact analysis of risk-based access policies
* Impact summary of recommended risk-based access policies / High risk sign-ins not being blocked - Shows the number of users not being blocked by a RBCA.
* Impact details: Sign-in risk policy scenarios / High risk sign-ins not being blocked by risk-based access policy - shows list of users that would have been impacted if RBCA was in place

---

# References
* [Entra ID Protection – Common Microsoft 365 Security Mistakes Series](https://campbell.scot/entra-id-protection-common-microsoft-365-security-mistakes-series/)
* [New developments in Microsoft Entra ID Protection](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/new-developments-in-microsoft-entra-id-protection/ba-p/4062701)
* [Advice for incident responders on recovery from systemic identity compromises](https://www.microsoft.com/en-us/security/blog/2020/12/21/advice-for-incident-responders-on-recovery-from-systemic-identity-compromises/)
* [DEV-0537 criminal actor targeting organizations for data exfiltration and destruction](https://www.microsoft.com/en-us/security/blog/2022/03/22/dev-0537-criminal-actor-targeting-organizations-for-data-exfiltration-and-destruction/)
* [Token tactics: How to prevent, detect, and respond to cloud token theft](https://www.microsoft.com/en-us/security/blog/2022/11/16/token-tactics-how-to-prevent-detect-and-respond-to-cloud-token-theft/)
* [What is Identity Protection? | Microsoft Entra ID](https://youtu.be/1REQYdZ6364?si=pnSdFnMkBSp0AZOQ )
* [How to use Identity Protection | Microsoft Entra ID](https://youtu.be/zvCMpkOwRPs?si=_7lgcuUwBq83lJJJ)
* [What’s new with Microsoft Entra ID Protection](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/what-s-new-with-microsoft-entra-id-protection/ba-p/3773132)
* [The refreshed Azure AD Identity Protection is now generally available](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/the-refreshed-azure-ad-identity-protection-is-now-generally/ba-p/1002916)
* [Identity Protection alerts now available in Microsoft 365 Defender](https://techcommunity.microsoft.com/t5/microsoft-defender-xdr-blog/identity-protection-alerts-now-available-in-microsoft-365/ba-p/3660997)
* [Microsoft Entra ID Protection documentation](https://learn.microsoft.com/en-us/entra/id-protection/)
* [Examine Azure Identity Protection](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-identity-protection/)
* [More coverage to protect your identities](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/more-coverage-to-protect-your-identities/ba-p/2365685)
* [Plan an Identity Protection deployment](https://learn.microsoft.com/en-us/entra/id-protection/how-to-deploy-identity-protection)
* [Four major Azure AD Identity Protection enhancements are now in public preview](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/four-major-azure-ad-identity-protection-enhancements-are-now-in/ba-p/326935)
* [Azure AD Mailbag: Identity protection](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/azure-ad-mailbag-identity-protection/ba-p/1257350)
* [Holistic compromised identity signals from Microsoft](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/holistic-compromised-identity-signals-from-microsoft/ba-p/2365683)
* [Empowering SOCs with Azure AD Identity Protection in Microsoft 365 Defender](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/empowering-socs-with-azure-ad-identity-protection-in-microsoft/ba-p/2365675)
* [Introducing diagnostic settings for Identity Protection — August identity updates](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/introducing-diagnostic-settings-for-identity-protection-august/ba-p/2464365)
* [Announcing Improved Identity Protection Signal Quality and Visibility](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/announcing-improved-identity-protection-signal-quality-and/ba-p/2464410)
* [Extend the reach of Azure AD Identity Protection into workload identities (This requires additional license)](https://techcommunity.microsoft.com/t5/microsoft-entra-blog/extend-the-reach-of-azure-ad-identity-protection-into-workload/ba-p/2365666)
