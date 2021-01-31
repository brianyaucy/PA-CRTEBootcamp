# Cross Forest Attacks - Foreign Security Principals & ACLs

- [Cross Forest Attacks - Foreign Security Principals & ACLs](#cross-forest-attacks---foreign-security-principals--acls)
  - [Foreign Security Principals (FSP)](#foreign-security-principals-fsp)
  - [FSP Enumeration](#fsp-enumeration)
  - [ACLs](#acls)
  - [ACLs enumeration](#acls-enumeration)

---

## Foreign Security Principals (FSP)

A **Foreign Security Principal (FSP)** represents a Security Principal in a external forest trust or special identities (like `Authenticated Users`, `Enterprise DCs` etc.).

Only SID of a FSP is stored in the **Foreign Security Principal Container** which can be resolved using the trust relationship. 

FSP allows external principals to be added to domain local security groups. Thus, allowing such principals to access resources in the forest. **Often, FSPs are ignored, mis-configured or too complex to change/cleanup in an enterprise making them ripe for abuse.**

<br/>

## FSP Enumeration

Let's enumerate FSPs for the `db.local` domain using the reverse shell we have there.

- PowerView

```
Find-ForeignGroup -Verbose 

Find-ForeignUser -Verbose
```

- AD Module

```
Get-ADObject -Filter {objectClass -eq "foreignSecurityPrincipal"}
```

<br/>

## ACLs

Access to resources in a forest trust can also be provided without using FSPs using ACLs.

Principals added to ACLs do NOT show up in the `ForeignSecurityPrinicpals` container as the container is populated only when a principal is added to a domain local security group.

<br/>

## ACLs enumeration

Let's enumerate ACLs for the `dbvendor.local` domain using the reverse shell we have on `db.local`:

- PowerView

```
Find-InterestingDomainAcl -Domain dbvendor.local
```

<br/>