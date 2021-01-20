# Enumerations

- [Enumerations](#enumerations)
  - [Active Directory - Components](#active-directory---components)
  - [Tools](#tools)
  - [PowerShell Detections](#powershell-detections)
  - [Execution Policy](#execution-policy)

---

## Active Directory - Components

**Schema**

- Define objects and their attributes

**Query and index machanism**

- Provide searching and publication of objects and their properties

**Global Catalog**

- Contain information about every object in the directory

**Replication Services**

- Distribute information across domain controllers

<br/>

----

## Tools

To enumerate AD environment, there are 2 common tools:

1. PowerView

```
. .\PowerView.ps1
```

2. ActiveDirectory.psd1

```
Import-Module .\Microsoft.ActiveDirectory.Management.dll
Import-Module .\ActiveDirectory.psd1
```

----

## PowerShell Detections

There are 4 possible PowerShell detections:

**1. System-wide transcriptions**

- `C:\Transcripts\<date>\PowerShell_transcript.<host>.xxxxxx.xxxxx`

<br/>

**2. Scriptblock logging**

- Windows Event ID 4104

<br/>

**3. Anti-Malware Scan Interface (AMSI)**

- Signature-based detection
- Can be bypassed easily
- https://amsi.fail/

<br/>

**4. Constrained Language Mode (CLM)**

- Integrated with Applocker and WDAC (Device Guard)
- Disable almost all interesting commands (e.g. `.NET`) and common Red Team powershell scripts
- The only one works in CLM will be the official AD module

<br/>

----

## Execution Policy

- **This is NOT a security measure!**
- Just to prevent user from accidently executing scripts
- Methods to bypass:

```
powershell -ep bypass
```

```
powershell -c <command>
```

```
powershell -encodedcommand $env:PSExecutionaPolicyPreference="bypass"
```

- Also see:<br/>
https://www.netspi.com/blog/entryid/238/15-ways-to-bypass-the-powershellexecution-policy

<br/>

----

