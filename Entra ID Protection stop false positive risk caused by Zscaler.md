# Entra ID Protection stop false positive risk caused by Zscaler

_**False positives will clear up as the model recognizes the MFA prompts.  Meaning this is not necessary, one concern that has been communicated to me is man in the middle from a proxy may be missed.**_

Also it is recommended by Zscaler to consider doing a direct to the Microsoft M365 logon urls so that the appropriate conditional access policy is applied.  [best practices for microsoft365 and zscaler](https://www.zscaler.com/resources/white-papers/best-practices-for-microsoft365-and-zscaler.pdf)

Also source anchor ip  - [Source IP Anchoring Configuration Guide for Office 365 Conditional Access](https://help.zscaler.com/zia/source-ip-anchoring-config-guide-office-365#:~:text=How%20to%20enable%20and%20configure%20Source%20IP%20Anchoring,using%20a%20source%20IP%20address%20of%20your%20choice.)

But maybe you are concerned with the volume of false positives.

If you are reading this your organization probably has a conditional access policy in place that only requires multifactor authentication from untrusted networks.  The issue is when an organization leverages solutions like ZScaler or Umbrella it looks like the user is traveling all over the United  States or World.  Unfortunately Entra ID Protection struggles with this and usually starts flagging the sign-in risk which could end up either blocking the user or repeatedly prompts the user to provide MFA and if you trust the location the MFA requirement usually goes away, which makes it seem like not a good solution.

## My recommendation: 
_if you want to do what is truly recommended all sign-ins should require a trusted or compliant device and some sort of multifactor authentication. Which means trusted locations are rarely used by conditional access policies._

## Here is how I have solved this with multiple organizations.

# Using this list of IP egress, create a named location.

Zscaler Egress IP ranges and Future Data Centers (18)
https://config.zscaler.com/zscaler.net/cenr

1. Navigate to the Named Locations Blade
2. Create a IP ranges locations
3. Name it something like "Zscaler Egress IP ranges"
4. Select Mark as trusted location
5. Enter in all the subnets from the Zscaler site.
6. Then select create

In Entra ID the workbook section has a new workbook called Impact analysis of risk-based access policies, if you scroll down at the bottom it will show current list of IP addresses where multiple users are coming from. The ASN registered to Zscaler is 62044 and 53813.

## Now create a conditional access policy that requires MFA from the Zscaler Egress IP ranges named location.
1. Navigate to the conditional access policies blade in the Entra ID portal.
2. Create a new policy
3. Name something like "require MFA from Zscaler Egress IP ranges" 
4. Users: Select all users, may want to exclude Guest if you currently are not requiring guest to MFA.
5. Target Resources: Select all cloud apps
6. Conditions: Locations - select locations - Zscaler Egress IP ranges
7. Grant: Require multifactor authentication
8. Place the policy in report only, test it then follow your change request process and get it enabled.
	
Next up will be to set back up the conditional access policy that requires MFA when a medium or high sign-in risk is detected.

I'm also being told that enabling this might be an answer.
[Understanding Source IP Anchoring](https://help.zscaler.com/zia/understanding-source-ip-anchoring)
