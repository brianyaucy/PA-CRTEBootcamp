# Domain Enumeration

- [Domain Enumeration](#domain-enumeration)
  - [Tools](#tools)

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

