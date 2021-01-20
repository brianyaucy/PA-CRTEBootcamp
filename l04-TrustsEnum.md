# Hands-on 4: Trusts Enumeration

- [Hands-on 4: Trusts Enumeration](#hands-on-4-trusts-enumeration)
  - [Tasks](#tasks)
  - [Enumerate all domains in the techcorp.local forest](#enumerate-all-domains-in-the-techcorplocal-forest)
  - [Map the trusts of the us.techcorp.local domain](#map-the-trusts-of-the-ustechcorplocal-domain)
  - [Map external trusts in techcorp.local forest](#map-external-trusts-in-techcorplocal-forest)
  - [Identify external trusts of us domain.](#identify-external-trusts-of-us-domain)
  - [Can you enumerate trusts for a trusting forest?](#can-you-enumerate-trusts-for-a-trusting-forest)

---

## Tasks

- Enumerate all domains in the techcorp.local forest. 
- Map the trusts of the us.techcorp.local domain. 
- Map external trusts in techcorp.local forest. 
- Identify external trusts of us domain. Can you enumerate trusts for a trusting forest?

---

## Enumerate all domains in the techcorp.local forest

Use AD Module.

```
(Get-ADForest).Domains
```

![picture 38](images/9d9dd88ab4d3f626b3d49fef0e5ae046585118c94c762e2341908e3a44836330.png)  

<br/>

---

## Map the trusts of the us.techcorp.local domain

Since the current domain is `us.techcorp.local`, simply run:

```
Get-ADTrust -Filter 'intraForest -ne $True' -Server (Get-ADForest).Name
```

![picture 39](images/9c0674df0668c5c4acbc83d8866e4546c280e81c0299ed1f5167468c6c607fd4.png)  

<br/>

---

## Map external trusts in techcorp.local forest

```
(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_}
```

![picture 40](images/5a3c3379cd68778314ff5f7044ad743a4a253339eedb4394639bd79ab12c71e5.png)  

<br/>

---

## Identify external trusts of us domain. 

```
Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server us.techcorp.local
```

![picture 41](images/0c8e98b969144223ab82110e9fb3f3dc61c2cf3a370baf19beadc134cd065c6e.png)  


<br/>

---

## Can you enumerate trusts for a trusting forest?

Since `us.techcorp.local` has **bi-directional trust** with `eu.local`, we can enumerate `eu.local` trust by:

```
Get-ADTrust -Filter * -Server eu.local
```

![picture 42](images/7a40c4fe72cac73fbf95021001e427feced852050fd2831d38c33629732614ca.png)  

<br/>

---