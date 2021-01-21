# Abusing MS Product Trusts

- [Abusing MS Product Trusts](#abusing-ms-product-trusts)
  - [Abusing Trusts of MS Products](#abusing-trusts-of-ms-products)
  - [Privilege Escalation - MS Exchange](#privilege-escalation---ms-exchange)
  - [Mailbox Permissions](#mailbox-permissions)

---

## Abusing Trusts of MS Products

Multiple products from Microsoft are very well integrated in an AD environment. Products like **SQL Server** and **Exchange** integrate with AD and therefore become a part of the domain trust.

Let's have a look at Exchange.

<br/>

## Privilege Escalation - MS Exchange

**Exchange 2019** provides multiple exciting opportunities for privilege escalation. More so if the split permissions are not used.

We are going to discuss Exchagne 2019 abuse in 2 categories:

1. Multiple groups with very interesting permissions Exchange - Exchange Servers, Exchagne Trusted Subsystem, Exchange Windows Permissions, etc. The groups are added in a new container `Microsoft Exchange Security Groups`.
2. The ability to read mailboxes of other users (due to misconfigured permissions).

<br/>

## Mailbox Permissions

Mailboxes, of course, contains a lot of interesting information.

In an Enterprise, mailboxes are often delegated to other users. This is more applicable to high value targets (think of CEO, C_O, ...)

Even without explict delegation, users who are a part of the `Domain Admins`, `Enterprise Admins`, `Organization Management`, `Exchange Trusted Subsystem` and `Exchange Servers Group` have, by default, **FULL ACCESS** to mailboxes of all the users.

We will use **MailSniper** for enumerating and accessing mailboxes.

- https://github.com/dafthack/MailSniper

<br/>

We can enumerate all mailboxes using the following command:

```
Get-GlobalAddressList -ExchHostname us-exchange -Verbose -UserName us\studentuser64 -Password <password>
```

<br/>

Next, look at those mailboxes where our current user has access:

```
Invoke-OpenInboxFinder -EmailList C:\AD\Tools\emails.txt -ExchHostname us-exchange -Verbose
```

<br/>

Once we have identified mailboxes where we can read emails, use the following command to read emails - this looks for terms like `pass`, `creds`, `credentials` from the top 100 emails:

```
Invoke-SelfSearch -Mailbox <email> -ExchHostname us-exchange -OutputCsv .\mail.csv
```

- Alternatively, using `Exchange Manager (Organization Management)` or `Exchange User (Exchange Trusted Subsystem)` privileges also allows us to read the emails!

<br/>

