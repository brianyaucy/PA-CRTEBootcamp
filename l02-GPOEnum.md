# Hands-On 2: GPO Enumeration

- [Hands-On 2: GPO Enumeration](#hands-on-2-gpo-enumeration)
  - [Tasks](#tasks)
  - [Enumerate Restricted Groups from GPO](#enumerate-restricted-groups-from-gpo)
  - [Enumerate Membership of the restricted groups](#enumerate-membership-of-the-restricted-groups)
  - [List all the OUs](#list-all-the-ous)
  - [List all the computers in the Students OU](#list-all-the-computers-in-the-students-ou)
  - [List the GPOs](#list-the-gpos)
  - [Enumerate GPO applied on the Students OU](#enumerate-gpo-applied-on-the-students-ou)

----

## Tasks

Enumerate following for the `us.techcorp.local` domain: 

- Restricted Groups from GPO
- Membership of the restricted groups 
- List all the OUs 
- List all the computers in the Students OU. 
- List the GPOs 
- Enumerate GPO applied on the Students OU.

---

## Enumerate Restricted Groups from GPO

Use PowerView:

```
Get-DomainGPOLocalGroup
```

![picture 20](images/4288821213985e36f3144c6bdabd78f965a7844200328f2c534e7ec942d935b4.png)  

<br/>

---

## Enumerate Membership of the restricted groups 

```
Get-DomainGroupMember us\machineadmins
```

![picture 21](images/5635f4eb2a78d5910d11db02104d52aec1e08f10f2268a76abdc09734df99e02.png)  


<br/>

---

## List all the OUs 

Use PowerView:

```
Get-DomainOU | Select DistinguishedName
```

![picture 22](images/32cd388d36d29d818e130f72c1f499883fa657e3636d9250ab546776f91efafd.png)  

<br/>

---

## List all the computers in the Students OU

```
(Get-DomainOU -Identity 'OU=Students,DC=us,DC=techcorp,DC=local').distinguishedname | %{Get-DomainComputer -SearchBase $_} | Select dnshostname
```

![picture 23](images/7e3e2e45aa0b7bcfa8eba7f4745ee3e8801600d314fbda397e0f280aee8d2ada.png)  

<br/>

---

## List the GPOs

```
Get-DomainGPO
```

![picture 24](images/a34333a4aaf73bdbd51b0da7cfd7b081aa242fd99440bce2a62ec9cd37924121.png)  

<br/>

---

## Enumerate GPO applied on the Students OU

First get the `gplink` attribute of the Student OU:

```
Get-DomainOU | Where-Object {$_.name -Like "*Student*"}
```

![picture 26](images/eda670cc554c2b74bb3360aa04797d47d5430a8e035c44d9b0e4c08c84106341.png)  

- `{FCE16496-C744-4E46-AC89-2D01D76EAD68}`

<br/>

Then list the GPO applied on the Students OU:

```
Get-DomainGPO -Identity '{FCE16496-C744-4E46-AC89-2D01D76EAD68}'
```

![picture 27](images/83777f9889a4d70667e00ea5e13f481687a2d599bffc5648b32d2b5fa9c7abe3.png)  


<br/>

----

