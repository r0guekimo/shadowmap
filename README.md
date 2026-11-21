# voidscope

A personal recon and bug bounty methodology: the commands, tools and workflow I use during authorized security testing. It is a working reference that I update as my process evolves.

```
> id
uid=1000(r0guekimo) groups=bugbounty,recon

> status
[+] Living methodology, updated as the workflow evolves
[+] Built for authorized testing only (bug bounty programs, VDPs)
[+] Recon-first: passive enumeration before anything active
```

## Contents

- [Overview](#overview)
- [The workflow](#the-workflow)
- [How to use this repo](#how-to-use-this-repo)
- [Scope and ethics](#scope-and-ethics)
- [Contributing](#contributing)

## Overview

Good bug bounty results usually come from a repeatable process, not from random tool runs. This repo keeps that process in one place so each engagement starts from the same checklist: map the attack surface passively first, then probe actively, then test for vulnerabilities.

## The workflow

The full pipeline lives in [`recon-workflow.md`](recon-workflow.md), in this order:

| # | Stage | Goal |
|---|-------|------|
| 1 | Subdomain enumeration | Build the list of hosts in scope |
| 2 | Active probing and port scanning | Find which hosts are alive and what they expose |
| 3 | Historical and content discovery | Recover old URLs, files and endpoints |
| 4 | Directory and endpoint fuzzing | Find hidden paths and routes |
| 5 | JavaScript analysis | Extract endpoints, parameters and secrets from client code |
| 6 | Parameter discovery | Find inputs worth testing |
| 7 | Vulnerability testing | XSS, SQLi, SSRF, LFI/RFI, open redirect |
| 8 | Misconfiguration and add-ons | WAF, CMS, S3 buckets, exposed `.git` |

## How to use this repo

1. Confirm the target is in scope (program page, VDP or signed engagement).
2. Work through `recon-workflow.md` from stage 1, staying passive until stage 2.
3. Keep notes per target outside this repo, so no target data ends up in it.
4. Respect the program's rate limits and rules at every stage.

## Scope and ethics

Everything here assumes explicit authorization: an active bug bounty program, a vulnerability disclosure policy, or a signed engagement. Nothing in this repo is meant for use against systems without permission. You are responsible for how you use it.

## Contributing

Suggestions and corrections are welcome. Open an issue or a pull request with a short explanation of what you would change and why.

---

<sub>🔻 r0gue_kimo</sub>