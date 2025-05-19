# Identity Protection Risk Based Conditional Access Policies

## Core Identity Protection Policies

### Require MFA for medium and high sign-in risk

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | Target Resources: All Resources  | Sign-in Risk: High and Medium | Require multifactor authentication | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment:
 
### Require password change for high user risk

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | Target Resources: All Resources  | User Risk: High | Require multifactor authentication and Change Password | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment: 

 ### Block high user risk on untrusted device.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | Target Resources: All Resources  | User Risk: High <br> Filter for devices: 'device.isCompliant -ne True -and device.trustType -ne "ServerAD"' | Block | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment: 

## Additional Identity Protection Policies

### Block all sign-in risk for security information registration

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | User actions: Register security information  | Sign-in Risk: High, Medium and Low | Block | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment:

 ### Block all sign-in risk for Admin Portals

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | Target Resources: Windows Azure Service Management API and Microsoft Admin Portals <br> Exclude: Office 365  | Sign-in Risk: High, Medium and Low | Block | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment:

### Block directory sync account all sign-in risk

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Roles: Directory Syncronization Account  <br /> Exclude: BreakGlass  | Target Resources: All Resources  | Sign-in Risk: High, Medium and Low | Block | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment:
