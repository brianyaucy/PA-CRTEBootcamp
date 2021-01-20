# Domain Enumeration

- [Domain Enumeration](#domain-enumeration)
  - [Tools](#tools)
  - [BloodHound](#bloodhound)

----

## Tools

To enumerate AD environment, there are 2 common tools:

1. PowerView

- https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1

```
. .\PowerView.ps1
```

2. ActiveDirectory.psd1

- https://docs.microsoft.com/en-us/powershell/module/addsadministration/?view=win10-ps
- https://github.com/samratashok/ADModule

```
Import-Module .\Microsoft.ActiveDirectory.Management.dll
Import-Module .\ActiveDirectory.psd1
```

<br/>

3. Bloodhound (C# and PowerShell Collectors)
   
- https://github.com/BloodHoundAD/BloodHound

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

The remaining steps see [Lab - BloodHound](l00-Bloudhound.md)

----

