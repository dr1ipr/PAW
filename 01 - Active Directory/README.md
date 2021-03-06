## Shadow Groups
If you want to use Shadow Groups to aide in managing your devices, you will need to keep your PAW devices separate from your day-to-day devices.  This means a separate OU for your PAW devices and a separate OU for all your other workstations.  

What are Shadow Groups?

Shadow Groups are groups that mirror the membership of an Active Directory OU.  If you have ever administrated a Novel Netware network, you will recall that you can apply the membership of an OU to a network object's ACL.  Thus, giving access to an object based on the users in an OU.  Active Directory does not allow you to add OUs to ACLs.  Thus, if we wanted to replicate this behavior in AD, we need shadow groups.  With shadow groups you now have all members of each department in a group.  All departmental computers in their own groups.  All laptops in their own groups.  All Tablets in their own groups...

What else can you do with Shadow Groups?

* Apply GPO security filtering to shadow groups rather than **Authenticated Users**.  Now you can apply the GPO to a higher level OU, and have it apply to only certain child OUs without the need for complicated WMI filters.
* More effectively manage NPS 802.1x policies.  
* Quicker reporting for auditors.  All employees are in thier own group, filtering out things like service accounts, contacts, and contractors which are members of the **Domain Users** group.  Same for computers and the **Domain Computers** group.
* Rule the world

How are Shadow Groups managed?

A scheduled task runs on a regular interval and creates groups based on your Active Directory OU hierarchy.  It then takes the members of the OU and adds them as members of the group.  Lastly, it removes group members that may have moved to a different OU, keeping your group membership accurate.

## Recommended Active Directory Hierarchy
```
DOMAIN.COM
├── Domain Controllers
└── Company
    ├── Computers
    │   ├── Disabled-Computers - - - Will hold all disabled computer accounts
    │   └── Location A
    │       ├── PAW
    │       │   ├── Tier 0   - - - - Will hold Tier 0 PAWs (for domain admins)
    │       │   ├── Tier 1   - - - - Will hold Tier 1 PAWs (for server admins)
    │       │   └── Tier 2   - - - - Will hold Tier 2 PAWs (for helpdesk admins)
    │       ├── Servers
    │       │   ├── Tier 0   - - - - Will hold Tier 0 servers (but not DCs!)
    │       │   └── Tier 1   - - - - Will hold Tier 1 servers (most member servers)
    │       ├── Workstations - - - - Will hold all Computer accounts.  Feel free to organize your own hierarchy.  For this example, we use <Locale>\<Department>
    │       │   └── Location     - - - - Each office location will have its own OU
    │       │       └── Department   - - Each department will hold the computer accounts for that department
    │       └── VMs          - - - - All VMs, including your PAWs day-to-day VM
    ├── Groups
    │   └── Security Groups
    │       ├── PAW          - - - - All groups related to PAW management
    │       ├── Shadowgroups-Computers - - - Computer object's shadowgroups
    │       ├── Shadowgroups-Servers - - - - Server object's shadowgroups
    │       └── Shadowgroups-Users - - - - - User's object's shadowgroups
    └── Users
        ├── Employees        - - - - Will hold all Employee accounts.  Feel free to organize your own hierarchy.  For this example, we use <Locale>\<Department>
        │   └── Location     - - - - Each office location will have its own OU
        │       └── Department   - - Each department will hold the user accounts for that department
        ├── Disabled-Users   - - - - Will hold all disabled user accounts
        ├── ServiceAccounts  - - - - Will hold all service accounts, and special use accounts (like accounts that run scheduled tasks)
        └── PAW Accounts
            ├── Tier 0       - - - - Will hold Tier 0 user accounts (for domain admins)
            ├── Tier 1       - - - - Will hold Tier 1 user accounts (for server admins)
            └── Tier 2       - - - - Will hold Tier 2 user accounts (for helpdesk admins)
```
## Active Directory Permissions
The shadowgroup.ps1 script will be run by a standard user account which must be given the explicit permissions listed below.  Modify AD Advanced Security Permissions of the following OUs (should probably be scripted in the future...)

***COMPANY.COM\Company\Computers***
* ACL 1
  * Principal: **AD-Company-Computers--DeleteComputerObjects**
  * Type: **Allow**
  * Applies to: **Descendant Computer Objects**
  * Properties: **Write Name, and Write name (capitol and lower case N & n)**
* ACL 2
  * Principal: **AD-Company-Computers--DeleteComputerObjects**
  * Type: Allow
  * Applies to: **This object and all descendant Objects**
  * Permissions: **Delete Computer objects**
* ACL 3
  * Principal: **AD-Company-Computers--DeleteComputerObjects**
  * Type: **Allow**
  * Applies to: **Descendant Computer Objects**
  * Permissions: **Read all properties**

***COMPANY.COM\Company\Users\Employees***
* ACL 1
  * Principal: **AD-Company-Users--DeleteUserObjects**
  * Type: **Allow**
  * Applies to: **Descendant User Objects**
  * Properties: **Write Name, and Write name (capitol and lower case N & n)**
* ACL 2
  * Principal: **AD-Company-Users--DeleteUserObjects**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **Delete user objects**
* ACL 3
  * Principal: **AD-Company-Users--DeleteUserObjects**
  * Type: **Allow**
  * Applies to: **Descendant User Objects**
  * Properties: **Read all properties**

***COMPANY.COM\Company\Computers\Disabled-Computers***
* ACL 1
  * Principal: **AD-Company-Computers-DisabledComputers--CreateComputerObjects**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **Create Computer objects**
* ACL 2
  * Principal: **AD-Company-Computers-DisabledComputers--CreateComputerObjects**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **List contents, Read all properties, write all properties, read permissions**

***COMPANY.COM\Company\Groups\SecurityGroups\ShadowGroups-Computers***
* ACL 1
  * Principal: **AD-Company-Groups-ShadowGroupsComputers--Modify**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **Create Group objects, Delete Group objects**
* ACL 2
  * Principal: **AD-Company-Groups-ShadowGroupsComputers--Modify**
  * Type: **Allow**
  * Applies to: **Descendant Group objects**
  * Permissions: **Full control**

***COMPANY.COM\Company\Groups\SecurityGroups\ShadowGroups-Servers***
* ACL 1
  * Principal: **AD-Company-Groups-ShadowGroupsServers--Modify**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **Create Group objects, Delete Group objects**
* ACL 2
  * Principal: **AD-Company-Groups-ShadowGroupsServers--Modify**
  * Type: **Allow**
  * Applies to: **Descendant Group objects**
  * Permissions: **Full control**

***COMPANY.COM\Company\Groups\SecurityGroups\ShadowGroups-Users***
* ACL 1
  * Principal: **AD-Company-Groups-ShadowGroupsUsers--Modify**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **Create Group objects, Delete Group objects**
* ACL 2
  * Principal: **AD-Company-Groups-ShadowGroupsUsers--Modify**
  * Type: **Allow**
  * Applies to: **Descendant Group objects**
  *  Permissions: **Full control**

***COMPANY.COM\CompanyUsers\Disabled-Users***
* ACL 1
  * Principal: **AD-Company-Users-DisabledUsers--CreateUserObjects**
  * Type: **Allow**
  * Applies to: **This object only**
  * Permissions: **Create User objects**
* ACL 2
  * Principal: **AD-Company-Users-DisabledUsers--CreateUserObjects**
  * Type: **Allow**
  * Applies to: **This object and all descendant objects**
  * Permissions: **Full control**
## Users

### Each Domain Admin will have the following accounts:
* **Normal domain user account**: used for logging into the Tier 0 PAW.  Will escalate to local admin to do admin stuff. Also logs into the PAW VM to do day-to-day tasks.
* **Tier 0 Admin**: Member of domain admins, the normal domain account elevates to this account to admin stuff on Tier 0 servers.
* **Tier 1 Admin**: Used to allow the user to RDP to Tier 1 member servers using /RemoteCredentialGuard. Normal domain user also uses this to elevate certain remote management consoles (RSAT/Server Manager) to manage remote Tier 1 servers.
* **Tier 2 Admin (optional)**: If the user will ever administrate workstations, they will need this account.  Used to allow the user to RDP to remote workstations using /RemoteCredentialGuard. Normal domain user also uses this to elevate certain remote management consoles (MMC) to manage  remote workstations.
* **Local user account**: Used as a contingency for any lost domain trusts.  In other words, if you fubar the domain and you can no longer log in to your PAW, this is the account you would use.
* **Local administrator account**: This account will be managed by LAPS.  Also used for fixing domain trust issues.  You would login with the local user account and elevate to this account to do admin stuff.
* **Access to server LAPS accounts**.  They can use this if RDP with /RestrictedAdmin is too restrictive.

### Each server administrator will have:
* **Normal domain user account**: used for logging into the Tier 1 PAW.  Will escalate to local admin to do admin stuff. Also logs into the PAW VM to do day-to-day tasks.
* **Tier 1 Admin**: Used to allow the user to RDP to Tier 1 member servers using /RemoteCredentialGuard. Normal domain user also uses this to elevate certain remote management consoles (RSAT/Server Manager) to manage remote Tier 1 servers.
* **Local user account**: Used as a contingency for any lost domain trusts.  In other words, if you fubar the domain and you can no longer log in to your PAW, this is the account you would use.
* **Local administrator account**: This account will be managed by LAPS.  Also used for fixing domain trust issues.  You would login with the local user account and elevate to this account to do admin stuff.
* **Access to server LAPS accounts**.  They can use this if RDP with /RestrictedAdmin is too restrictive.

### Each Helpdesk user will have:
* **Normal domain user account**: used for logging into the Tier 1 PAW.  Will escalate to local admin to do admin stuff. Also logs into the PAW VM to do day-to-day tasks.
* **Tier 2 Admin**: Normal domain user uses this to elevate certain remote management consoles (MMC) to manage remote  workstations.
* **Local user account**: Used as a contingency for any lost domain trusts.  In other words, if you fubar the domain and you can no longer log in to your PAW, this is the account you would use.
* **Local administrator account**: This account will be managed by LAPS.  Also used for fixing domain trust issues.  You would login with the local user account and elevate to this account to do admin stuff.
* **Access to all workstation LAPS accounts**.  They can use this if RDP with RA is too restrictive.

***NOTE***: *Helpdesk should never use this with /RemoteCredentailGuard.  This is because if an RDP session is initiated to a compromised client that an attacker already controls, the attacker could use that open channel to create sessions on the user's behalf (without compromising credentials) to access any of the user’s resources for a limited time (a few hours) after the session disconnects. [(Source)](https://docs.microsoft.com/en-us/windows/access-protection/remote-credential-guard)*

## Groups
The following groups must be created in Company > Groups > SecurityGroups > RBAC-PAW.  The sub-bullet point are the members of the specified group.

**PAW-AllPAW-Computers** - Members of this group include all PAW Tier groups.  It is a collection of all PAW machines.
* PAW-Tier0-Computers
* PAW-Tier1-Computers
* PAW-Tier2-Computers

**PAW-BlockPowershell** - Members of this group are blocked from using PowerShell via GPO.
* PAW-Users

**PAW-BlockLocalLogon** - Members of this group are not literally blocked from logging in locally, but rather from running certain applications via the AppLocker GPO after they have logged in.
* PAW-Admins

**PAW-Azure-Admins** - Members of this group are permitted to connect to pre-identified cloud services via Privileged Access Workstations
* not sure yet.

**PAW-Tier0-Admins** - Members of this group are Tier 0 server admins.
* All members of the Company\Users\PAW Accounts\Tier 0 OU

**PAW-Tier0-Computers** - Members of this group are Tier 0 PAWs.  Used mainly for GPO filtering.
* All Tier 0 PAWs

**PAW-Tier0-Users** - Members of this group are tier 0 PAW users.  They can log into Tier 0 PAWs.  They are a normal user account on PAWs that use the Tier 0/1/2 Admin accounts to elevate certain tasks.
* All domain user accounts that need to log into Tier 0 PAWs

**PAW-Tier1-Admins** - Members of this group are Tier 1 server admins.
* All members of the Company\Users\PAW Accounts\Tier 1 OU

**PAW-Tier1-Computers** - Members of this group are Tier 1 PAWs.  Used mainly for GPO filtering.
* All Tier 1 PAWs  

**PAW-Tier1-Users** - Members of this group are tier 1 PAW users.  They can log into Tier 1 PAWs.  They are a normal user account on PAWs that use their Tier 1 Admin account to elevate certain tasks.
* All domain user accounts that need to log into Tier 0 PAWs

**PAW-Tier2-Admins** - Members of this group are Tier 2 server admins.
* All members of the Company\Users\PAW Accounts\Tier 2 OU

**PAW-Tier2-Computers** - Members of this group are Tier 2 PAWs.  Used mainly for GPO filtering.
* All Tier 2 PAWs

**PAW-Tier2-Users** - Members of this group are tier 2 PAW users.  They can log into Tier 2 PAWs.  They are a normal user account on PAWs that use the Tier 2 Admin accounts to elevate certain tasks.
* All domain user accounts that need to log into Tier 0 PAWs

**PAW-Users** - Members of this groups include all the Tier 0, 1, and 2 Users
* PAW-Tier0-Users
* PAW-Tier1-Users
* PAW-Tier2-Users

**PAW-Admins** - Members of this groups include all the Tier 0, 1, and 2 Admins
* PAW-Tier0-Admins
* PAW-Tier1-Admins
* PAW-Tier2-Admins

## Additional Resources
For more information on what accounts count as Tier 0, see [Microsoft's recommendations here](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material#T0E_BM).
