# Hands-on 26: Cross Forest Attacks - MSSQL Servers

- [Hands-on 26: Cross Forest Attacks - MSSQL Servers](#hands-on-26-cross-forest-attacks---mssql-servers)
  - [Tasks](#tasks)
  - [Enumeration](#enumeration)

---

## Tasks

Get a reverse shell on a `db-sqlsrv` in `db.local` forest by abusing database links from `us-mssql`.

<br/>

---

## Enumeration

Import `PowerUpSQL.ps1` using InviShell:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\PowerUpSQL-master\PowerUpSQL.ps1
```

![picture 17](images/364cb35bfb53920774829881908b441c299842f4cf0a9e9e9a38c7d1cf582a01.png)  


<br/>

Then 