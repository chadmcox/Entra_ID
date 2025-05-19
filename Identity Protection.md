#Identity Protection

## Core Identity Protection Policies

### Require MFA for medium and high sign-in risk

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | User actions: All Resources  | Sign-in Risk: High and Medium | Require multifactor authentication | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment:
 
### Require password change for high user risk

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | User actions: All Resources  | User Risk: High | Require multifactor authentication and Change Password | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment: 

 ### Block high user risk on untrusted device.

| Users | Cloud Apps or Actions | Conditions | Grant | Session |
| --------------------- | --------------------- | --------------------- | --------------------- | --------------------- |
| Include: All users  <br /> Exclude: BreakGlass  | User actions: All Resources  | User Risk: High, Filter for devices: 'device.isCompliant -ne True -and device.trustType -ne "ServerAD"' | Block | Sign-in Frequency: EveryTime |  

 **Prereq:** NA

 **Comment: 
