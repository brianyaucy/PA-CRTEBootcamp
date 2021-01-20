# Domain Enumeration

- [Domain Enumeration](#domain-enumeration)
  - [Tools](#tools)
  - [BloodHound](#bloodhound)
  - [Domain Enumeration](#domain-enumeration-1)
  - [Domain Policy Enumeration](#domain-policy-enumeration)
  - [Domain User Enumeration](#domain-user-enumeration)
  - [Domain Computer Enumeration](#domain-computer-enumeration)
  - [Domain User Group Enumeration](#domain-user-group-enumeration)
  - [Local Group Enumeration on machines](#local-group-enumeration-on-machines)
  - [GPO Enumeration](#gpo-enumeration)
  - [OU Enumeration](#ou-enumeration)
  - [ACL Enumeration](#acl-enumeration)

----

## Tools

To enumerate AD environment, there are 2 common tools:

**1. PowerView**

https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1

```
. .\PowerView.ps1
```

<br/>

**2. ActiveDirectory.psd1**

- https://docs.microsoft.com/en-us/powershell/module/addsadministration/?view=win10-ps
- https://github.com/samratashok/ADModule

```
Import-Module .\Microsoft.ActiveDirectory.Management.dll
Import-Module .\ActiveDirectory.psd1
```

<br/>

**3. Bloodhound (C# and PowerShell Collectors)**
   
https://github.com/BloodHoundAD/BloodHound

----

## BloodHound

BloodHound uses **Graph Theory** for providing the capability of mapping the shortest path for interesting thigns like Domain Admins.

<br/>

BloodHound has built-in queries for frequently used actions. It supports **custom Cypher queries**.

<br/>

The following steps assume you are using BloodHound on Kali:

**Step 1: Run ingestor/collector using PowerShell or C#**

```
. .\BloodHound-master\Ingestors\SharpHound.ps1
```

```
Invoke-BloodHound -CollectionMethod All
```

Note we can avoid tools like MS ATA by using:

```
Invoke-BloodHound -CollectionMethod All -ExcludeDomainControllers
```

A file will be generated.

<br/>

The remaining steps see [Lab - BloodHound](l00-Bloodhound.md)

----

## Domain Enumeration

**Get Current Domain**

- PowerView
  
```
Get-NetDomain
```

- AD Module
  
```
Get-ADDomain
```

<br/>

**Get object of another domain**

- PowerView

```
Get-Domain -Domain techcorp.local
```

- AD Module

```
Get-ADDomain -Identity techcorp.local
```

<br/>

**Get domain SID of the current domain**

- PowerView

```
Get-DomainSID
```

- AD Module

```
(Get-ADDomin).DomainSID
```

<br/>

---

## Domain Policy Enumeration

**Get domain policy of the current domain**

- PowerView

```
Get-DomainPolicyData
```

```
(Get-DomainPolicyData).systemaccess
```

<br/>

**Get domain policy of another domain**

- PowerView

```
(Get-DomainPolicyData -Domain techcorp.local).systemaccess
```

<br/>

---

## Domain User Enumeration

**Get a list of users in teh current domain**

- PowerView

```
Get-DomainUser
Get-DomainUser -Identity studentuser64
```

- AD Module

```
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity studentuser64 -Properties *
```

<br/>

Note:<br/>
- For DC, it doesn't matter if the default Domain Admin is renamed - always identified by the SID


<br/>

Note that properties of users in the current domain are very useful for situational awareness. For example, 

- it is abnormal that a user has `logoncount = 0` - this should be more like a decoy user;

- it is also interesting to see `pwdlastset` to be a very old date.

- PowerView

```
Get-DomainUser -Properties pwdlastset
```

- AD Module

```
Get-ADUser -Filter * -Properties * | Select -First 1 | Get-Member -MemberType *Property | Select Name
```

```
Get-ADUser -Filter * -Properties * | Select name, @{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```

<br/>

**Search for a particular string in a user's attributes**

- PowerView

```
Get-DomainUser -LDAPFilter "Description=*built*" | Select Name, Description
```

- AD Module

```
Get-ADUser -Filter 'Description -Like "*built*"' -Properties Description | Select Name, Description
```

<br/>

---

## Domain Computer Enumeration

**Get a list of computers in the current domain**

- PowerView

```
Get-DomainComputer | Select Name
```

```
Get-DomainComputer -OperatingSystem "Windows Server 2019 Standard"
```

```
Get-DomainComputer -Ping
```

- AD Module

```
Get-ADComputer -Filter * | Select Name
```

```
Get-ADComputer -Filter 'OperatingSystem -Like "*Windows Server 2019 Standard*"' -Properties OperatingSystem | Select Name, OperatingSystem
```

```
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```

```
Get-ADComputer -Filter * -Properties *
```

<br/>

---

## Domain User Group Enumeration

**Get all the groups in the current domain**

- PowerView

```
Get-DomainGroup | Select Name
```

```
Get-DomainGroup -Domain techcorp.local
```

- AD Module

```
Get-ADGroup -Filter * | Select Name
```

```
Get-ADGroup -Filter * -Properties *
```

<br/>

**Get all groups containing the word "admin" in the group name**

- PowerView

```
Get-DomainGroup *admin* | Select Name
```

- AD Module

```
Get-ADGroup -Filter 'Name -like "*admin*"' | Select Name
```

<br/>

**Get all members of the Domain Admins group**

- PowerView

```
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

- AD Module

```
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

<br/>

**Get the group membership for a user**

- PowerView

```
Get-DomainGroup -UserName studentuser64
```

- AD Module

```
Get-ADPrincipalGroupMembership -Identity studentuser64
```

<br/>

---

## Local Group Enumeration on machines

**Get all the local groups on a machine**

- Need Admin Privilege on non-dc machines

- PowerView

```
Get-NetLocalGroup -ComputerName us-dc
```

<br/>

**Get members of all local groups on a machine**

- Need Admin Privilege on non-dc machines
- PowerView

```
Get-NetLocalGroupMember -ComputerName us-dc
```

<br/>

**Get members of the local group "Administrator" on a machine**

- Need Admin Privilege on non-dc machines
- PowerView

```
Get-NetLocalGroupMember -ComputerName us-dc -GroupName Administrators
```

<br/>

---

## GPO Enumeration

**Group Policy** provides the ability to manage/changes configuration easily and centrally in AD. It allow configuration of:

- Security settings
- Registry-based policy settings
- Group policy preferences like startup/shutdown/logon/logoff scripts settings

<br/>

However, GPO can be abused for various attacks like privesc, backdoors, persistence, etc.

<br/>

**Get list of GPO in the current domain**

- PowerView

```
Get-DomainGPO
```

```
Get-DomainGPO -ComputerIdentity student64.us.techcorp.local
```

<br/>

**Get GPO(s) which use Restricted Group or groups.xml for interesting users**

- PowerView

```
Get-DomainGPOLocalGroup
```

<br/>

**Get users which are in a local group of a machine using GPO**

- Give interesting users to target
- PowerView

```
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity student64.us.techcorp.local
```

```
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity us-mgmt.us.techcorp.local
```

<br/>

**Get machines where the given user is a member of a specific group**

```
Get-DomainGPOUserLocalGroupMapping -Identity studentuser64 -Verbose
```

<br/>

---

## OU Enumeration

**Get OUs in a domain**

- PowerView

```
Get-DomainOU
```

- AD Module

```
Get-ADOrganizationalUnit -Filter * -Properties *
```

<br/>

**Get GPO applied on an OU**

- Read GPOname from gplink attribute from `Get-DomainOU`
- PowerView

```
Get-DomainGPO -Identity '{FCE16496-C744-4E46-AC89-2D01D76EAD68}'
```

<br/>

**Get users which are in a local group of a machine in any OU using GPO**

- PowerView

```
(Get-DomainOU).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping 2>$null
```

<br/>

**Get users which are in a local group of a machine in a particular OU using GPO**

- PowerView

```
(Get-DomainOU -Identity 'OU=Mgmt,DC=us,DC=techcorp,DC=local').distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping 2>$null
```

For older versions of PowerView, the below commands would also work:

```
Get-DomainGPOComputerLocalGroupMapping -OUIdentity 'OU=Mgmt,DC=us,DC=techcorp,DC=local'
```

```
Find-GPOComputerAdmin -OUName 'OU=Mgmt,DC=us,DC=techcorp,DC=local'
```

<br/>

---

## ACL Enumeration

**Access Control Model** enables control on the ability of a process to access objects and other resources in AD based on:

- Access Tokens (security context of a process - identity and privs of user)
- Security Descriptors (SID of the owner, Discretionary ACL (DACL) and System ACL (SACL))
- Ref: <br/>
https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-model

<br/>

![picture 28](images/ca5ea2583b1b7a2cede98ab92a0a15fb3ad2346cbed4358fd5432f9f81907647.png)  


<br/>

**Access Control List (ACL)** is a list of **Access Control Entries (ACE)**, which corresponds to individual permission or audits access. 

- i.e. Who has the permission and what can be done on an object?

ACLs are vital to security architecture of AD. There are 2 types of ACL:

- DACL: Define the permissions trustees (a user / group) have on an object
- SACL: Log success and failure audit messages when an object is accessed

<br/>

![picture 29](images/c1504cb18d0d341ede99982d65deb0a62a566594e6f03839b8d36e521b2fc48f.png)  

<br/>

**Get the ACLs associated with a specified object**

- PowerView

```
Get-DomainObjectAcl -Identity student64 -ResolveGUIDs
```

<br/>

**Get the ACLs associated with the specified LDAP path to be used for search**

- PowerView

```
Get-DomainObjectAcl -Searchbase "LDAP://CN=Domain Admins,CN=Users,DC=us,DC=techcorp,DC=local" -ResolveGUIDs -Verbose
```

Note:<br/>
- Active Directory Rights:<br/>
  https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?redirectedfrom=MSDN&view=dotnet-plat-ext-5.0
- Extended Rights:<br/>
  https://technet.microsoft.com/en-us/library/ff405676.aspx

<br/>

**Search for interesting ACEs**

- Use without `-ResolveGUIDs` for faster result
- PowerView

```
Find-InterestingDomainAcl -ResolveGUIDs
```

<br/>

**Get the ACLs associated with the specified path**

- PowerView

```
Get-PathAcl -Path "\\us-dc\sysvol"
```

<br/>

---

