# Domain Enumeration

- [Domain Enumeration](#domain-enumeration)
  - [Tools](#tools)
  - [BloodHound](#bloodhound)
  - [Domain Enumeration](#domain-enumeration-1)
  - [<br/>](#)
  - [Domain Policy Enumeration](#domain-policy-enumeration)
  - [Domain User Enumeration](#domain-user-enumeration)
  - [Domain Computer Enumeration](#domain-computer-enumeration)
  - [Domain User Group Enumeration](#domain-user-group-enumeration)
  - [Local Group Enumeration on machines](#local-group-enumeration-on-machines)

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