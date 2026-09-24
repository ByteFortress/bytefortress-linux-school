# Byte Fortress Linux School

![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)
![Modules](https://img.shields.io/badge/modules-18-blue.svg)
![Status](https://img.shields.io/badge/status-active-success.svg)

Part of the **Byte Fortress Learn** open project: a free, open-source,
deep dive into Linux system administration. This is the follow-on
track after [`bytefortress-foundations-school`](https://github.com/Wh1t3R4bbit/bytefortress-foundations-school)
— foundations gets you comfortable with basic Linux; this repo takes
you from "comfortable user" to "can actually administer a Linux
server."

> This repo grew out of an older course of mine
> (*CS101: Linux System Administration*) that I'm rebuilding and
> updating here. If you're seeing placeholder-style explanations in
> a module, that's a module still being expanded with fuller original
> content — the structure, labs, and external resources are solid
> either way.

## Who this is for

Anyone who's finished `bytefortress-foundations-school` (or already
has equivalent basic Linux/shell comfort) and wants to go deep on
actually running and administering Linux systems: accounts, services,
networking, servers, security, and automation.

## Prerequisites

- Comfortable with the Linux shell, basic permissions, and package
  management (covered in `bytefortress-foundations-school` module 02)
- A home lab VM environment ready to go (covered in
  `bytefortress-foundations-school` module 08)

## Repository structure

```
bytefortress-linux-school/
├── README.md                          ← you are here
├── LICENSE                            ← CC BY-SA 4.0
├── CONTRIBUTING.md
├── FAQ.md
├── GLOSSARY.md
├── PROGRESS.md
├── SHOWCASE.md
├── docs/
│   └── curriculum-map.md              ← full module list, sequencing, time estimates
└── modules/
    ├── 01-linux-basics/                    ← each 01-17 module has:
    │   ├── README.md                       │   concept, lab, pitfalls, further reading
    │   └── exercises.md                    │   extra practice exercises
    ├── 02-account-management/
    ├── 03-booting-and-shutdown/
    ├── 04-software-management/
    ├── 05-linux-filesystem/
    ├── 06-controlling-processes/
    ├── 07-basic-networking/
    ├── 08-network-management/
    ├── 09-bash-scripting/
    ├── 10-python-for-sysadmins/
    ├── 11-linux-and-the-web/
    ├── 12-dns/
    ├── 13-email-servers/
    ├── 14-nfs/
    ├── 15-directory-services/
    ├── 16-linux-security/
    ├── 17-backups/
    └── 18-capstone-project/          ← open-ended project ideas, README only (no exercises.md)
```

## How to use this repo

1. Read `docs/curriculum-map.md` for full sequencing.
2. Work through modules in order — later modules (networking,
   services, security) assume earlier ones (accounts, filesystem,
   process control).
3. Copy `PROGRESS.md` into your own fork to track completion.
4. Check `FAQ.md` if something's unclear, and `GLOSSARY.md` for quick
   term lookups.
5. Finish with module 18, the capstone — build one real project
   drawing on what you've learned, and share it in `SHOWCASE.md`.
6. Once you finish, `bytefortress-pwn-school` and future specialized
   tracks build directly on this foundation.

## Contributing

See `CONTRIBUTING.md`. This repo is actively being expanded with
original content — corrections, clearer explanations, translated
content, and new external resource links are all welcome.

## License

Licensed under **CC BY-SA 4.0** — free to use, fork, and remix
(including commercially), as long as you give credit and keep
derivative works open under the same license. See `LICENSE`.
