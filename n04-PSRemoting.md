# PowerShell Remoting

- [PowerShell Remoting](#powershell-remoting)
  - [Introduction](#introduction)
  - [One-to-One PS Remoting](#one-to-one-ps-remoting)
  - [One-to-Many PS Remoting](#one-to-many-ps-remoting)
  - [Invoke-Command](#invoke-command)
  - [Tradecraft](#tradecraft)

---

## Introduction

Think of** PowerShell Remoting (PSRemoting)** as **psexec** on steroids but much more silent and super fast! It leverages **Windows Remote Management (WinRM)**, which is MS's implementation of WS-Management.

<br/>

It is enabled by default on Server 2012 onwards with a firewall exception. WinRM listens by default on **tcp/5985 (HTTP)** and **tcp/5986 (HTTPS)**.

<br/>

PSRemoting is the recommended way to manager Windows Core servers. For Desktop machine, you have to enable PSRemoting `Enable-PSRemoting` with admin privilege. 

<br/>

The remoting process runs as a high integrity process (i.e. you get an elevated shell).

<br/>

---

## One-to-One PS Remoting

One-to-One PSSession:

- Is interactive
- Runs in a new process (**wsmprovhost**)
- Is stateful

<br/>

Useful cmdlets:

```
New-PSSession
```

```
Enter-PSSession
```

<br/>

---

## One-to-Many PS Remoting

Also known as **Fan-out Remoting**. But it:

- Is Non-interactive
- Executes commands in parallel

<br/>

Useful cmdlets:

```
Invoke-Command
```

<br/>

---

## Invoke-Command

`Invoke-Command` run commands and scripts on:

- multiple remote computers
- in disconnected sessions (v3)
- as background job and more.

<br/>

It is also the best thing in PowerShell for passing the hashes, using credentials and executing commands on multiple remote computers. You may use `-Credentials` parameter to pass username / password (secure).

<br/>

**Execute commands or scriptblocks**

```
Invoke-Command -Scriptblock {Get-Process} -ComputerName (Get-Content <list of servers>)
```

<br/>

**Execute scritps from files**

```
Invoke-Command -FilePath C:\scripts\Get-PassHashes.ps1 -ComputerName (Get-Content <list of servers>)
```

<br/>

**Execute locally loaded function on the remote machines**

```
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list of servers>)
```

- To pass arguments (only positional arguments can be passed):

```
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list of servers>) -ArgumentList
```

<br/>

**Execute Stateful commands**

```
$sess = New-PSSession -ComputerName server1

Invoke-Command -Session $sess -ScriptBlock {$Proc = Get-Process}

Invoke-Command -Session $sess -ScriptBlock {$Proc.Name}
```

<br/>

---

## Tradecraft

PowerShell remoting supports the system-wide transcripts and deep scriptblock logging. We can use `winrs` in place of PSRemoting to evade the logging, while still reap the benefit of 5985 allowed between hosts:

```
winrs -r:<hostname> -u:<domain>\<user> -p:<password> <command>
```

- https://pentestlab.blog/2018/05/15/lateral-movement-winrm/

<br/>

We can also use COM objects of WSMan COM object:

- https://github.com/bohops/WSMan-WinRM

<br/>

---