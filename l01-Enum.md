# Hands-On 1: Basic Enumeration

- [Hands-On 1: Basic Enumeration](#hands-on-1-basic-enumeration)
  - [Tasks](#tasks)
  - [Preparation - Import AD Module](#preparation---import-ad-module)
  - [Enumerate Users](#enumerate-users)
  - [Enumerate Computers](#enumerate-computers)
  - [Enumerate Domain Admins](#enumerate-domain-admins)
  - [Enumerate Enterprise Admins](#enumerate-enterprise-admins)
  - [Enumerate Kerberos Policy](#enumerate-kerberos-policy)

---

## Tasks

Enumerate following for the us.techcorp.local domain: 

- Users 
- Computers 
- Domain Administrators 
- Enterprise Administrators 
- Kerberos Policy

<br/>

---

## Preparation - Import AD Module

To import AD Module:

```
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll; Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

<br/>

---

## Enumerate Users

```
Get-ADUser -Filter * -Properties *
```

```
Get-ADUser -Filter * -Properties CN, Description, PrimaryGroup, LastLogonDate, pwdLastSet | Select CN, Description, PrimaryGroup, LastLogonDate, pwdLastSet
```

![picture 19](images/a03a0b1922653906391dd22571c2c38113ad5e8772dcef33dc2ac3acb8ae1be7.png)  


<br/>

---

## Enumerate Computers

```
Get-ADComputer -Filter * -Properties DNSHostName | Select DNSHostName
```

![picture 18](images/0c78d5a1548155f4bbd8ec59d582b4896f387dc9ea8c335a849568ca8f901eb3.png)  


<br/>

---

## Enumerate Domain Admins

- To get information of the `Domain Admins` group:

```
Get-ADGroup -Filter 'Name -like "*Domain Admins*"' -Properties *
```

![picture 17](images/7b1e8a2dd3952150fc0999a4f10f4876e90ade22a5c87d453fb778471157b4f4.png)  


- To get members in the `Domain Admins` group:

```
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

![picture 16](images/920b79692266e95c3e9d8937b50ada44d0ffa01baddc6410cca24527ad572995.png)  


<br/>

---

## Enumerate Enterprise Admins

- To get information of the `Enterprise Admins` group:

```
Get-ADGroup -Identity "Enterprise Admins" -Properties * -Server techcorp.local
```

![picture 15](images/e4625dbe5ae92edb1a91188e32b1800ee6a5f4ee74156cc6f9f9ef78704a4f65.png)  


- To get members in the `Enterprise Admins` group:

```
Get-ADGroupMember -Identity "Enterprise Admins" -Server techcorp.local -Recursive
```

![picture 14](images/f8f6e631414f0aace54ec72a43bf63e58706c182032b81c9c2e0dad9c34e7345.png)  


<br/>

---

## Enumerate Kerberos Policy

To enumerate Kerberos Policy, we need to use **PowerView**.

First use **InviShell**:

```
cd C:\AD\Tools\InviShell; .\RunWithRegistryNonAdmin.bat
```

<br/>

Then import PowerView:

```
cd ..; . .\PowerView.ps1
```

![picture 13](images/7895a861b7d5a085e477a718280da58818db9a213b83e84d05aa7de1b2fb030e.png)  


<br/>

To get Kerberos Policy:

```
(Get-DomainPolicyData).KerberosPolicy
```

![picture 12](images/233cc614eca2fcceceeb0beb4800a309c1fa85c2c9406c8bac165f481e6f8c27.png)  


<br/>

---