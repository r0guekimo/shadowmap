# voidscope

Personal recon and bug bounty methodology — a working reference of the commands, tools, and workflow I use during authorized security testing engagements.

```
> id
uid=1000(r0guekimo) groups=bugbounty,recon

> status
[+] Living methodology — updated as the workflow evolves
[+] Built for authorized testing only (bug bounty programs, VDPs)
[+] Recon-first: passive enumeration before anything active
```

## What's in here

`recon-workflow.md` — the full pipeline, in order:

1. Subdomain enumeration
2. Active probing & port scanning
3. Historical & content discovery
4. Directory & endpoint fuzzing
5. JavaScript analysis
6. Parameter discovery
7. Vulnerability testing (XSS, SQLi, SSRF, LFI/RFI, open redirect)
8. Misconfiguration & recon add-ons (WAF, CMS, S3, exposed `.git`)

## Scope & ethics

Everything here assumes explicit authorization — an active bug bounty program, a vulnerability disclosure policy, or a signed engagement. Nothing in this repo is intended for use against systems without permission.

---

<sub>🔻 r0gue_kimo</sub>
