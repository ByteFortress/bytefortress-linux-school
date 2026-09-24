# Curriculum Map — Byte Fortress Linux School

18 modules, based on the original *CS101: Linux System
Administration* course outline, rebuilt and updated for this open
project. Sequenced so each builds on the last.

| # | Module | What you'll learn | Primary external resource | Est. time |
|---|--------|--------------------|-----------------------------|-----------|
| 01 | Linux Basics | Unix/Linux history, distros, man pages, core commands, text editors | [Linux Journey](https://linuxjourney.com/) | 4-6 hrs |
| 02 | Account Management | Users, groups, `sudo`, `/etc/passwd` & `/etc/shadow`, PAM basics | [Linux Journey — Grasshopper](https://linuxjourney.com/) | 4-6 hrs |
| 03 | Booting and Shutdown | BIOS/UEFI → bootloader (GRUB) → kernel → init, systemd targets | [Arch Wiki — systemd](https://wiki.archlinux.org/title/Systemd) | 3-5 hrs |
| 04 | Software Management | Package managers (`apt`/`dnf`), repositories, building from source | Distro package manager docs | 3-5 hrs |
| 05 | Linux Filesystem | FHS in depth, mounting, filesystem types, disk partitioning, LVM basics | [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html) | 5-7 hrs |
| 06 | Controlling Processes | `ps`/`top`, signals, `cron`/`systemd timers`, managing services | [Linux Journey — Processes](https://linuxjourney.com/) | 4-6 hrs |
| 07 | Basic Networking | Interface config, routing, `iptables`/`nftables`/`ufw` firewalls | Arch Wiki networking pages | 5-7 hrs |
| 08 | Network Management | SSH hardening, remote administration, network troubleshooting tools | [SSH Academy](https://www.ssh.com/academy/ssh) | 4-6 hrs |
| 09 | Bash Scripting | Variables, conditionals, loops, functions, real automation scripts | [ShellCheck](https://www.shellcheck.net/), [Bash Guide](https://mywiki.wooledge.org/BashGuide) | 6-8 hrs |
| 10 | Python for SysAdmins | Automating admin tasks with Python instead of bash | [Automate the Boring Stuff](https://automatetheboringstuff.com/) (free) | 6-8 hrs |
| 11 | Linux and the Web | Running a web server (Apache/Nginx), virtual hosts, TLS certs, a MySQL-backed page, Squid caching | [Nginx docs](https://nginx.org/en/docs/), [Let's Encrypt](https://letsencrypt.org/) | 7-9 hrs |
| 12 | DNS | Running your own DNS server, zone files, records, troubleshooting | [BIND9 docs](https://bind9.readthedocs.io/) | 4-6 hrs |
| 13 | Email Servers | SMTP fundamentals, running Postfix, spam/auth basics (SPF/DKIM) | [Postfix docs](http://www.postfix.org/documentation.html) | 4-6 hrs |
| 14 | NFS | Sharing filesystems across a network, exports, mount options | [Ubuntu NFS docs](https://ubuntu.com/server/docs/service-nfs) | 3-4 hrs |
| 15 | Directory Services | Centralized user/auth management — LDAP fundamentals, sssd client auth (NIS as legacy context) | [OpenLDAP docs](https://www.openldap.org/doc/) | 6-8 hrs |
| 16 | Linux Security | Hardening, SELinux/AppArmor, auditing, log review, iptables deep dive | [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) | 7-9 hrs |
| 17 | Backups | Backup strategies, `rsync`, `tar`, testing restores | [restic docs](https://restic.net/) | 3-5 hrs |
| 18 | Capstone Project | Design, build, and document a real sysadmin project drawing on prior modules | Your own build | 10-20+ hrs |

**Total: roughly 85-125 hours** — meaningfully deeper than
`bytefortress-foundations-school`, matching the original course's
8-10 week pace.

## A note on module 15 (Directory Services)

The original course covered NIS (Network Information System) for
centralized account management. NIS is legacy and considered insecure
by modern standards (unencrypted, weak authentication). This module
covers LDAP as the modern equivalent, with NIS discussed for
historical/conceptual context only — not as a recommended technology
to deploy.

## Sequencing logic

- **01-04** — the operational basics: using the system, managing
  accounts, understanding boot, installing software.
- **05-06** — deeper system internals: filesystem and process
  management.
- **07-08** — networking, from configuration to secure remote access.
- **09-10** — automation, first in bash then in Python.
- **11-15** — running actual network services (web, DNS, email, file
  sharing, directory services) — the "you can now run real
  infrastructure" arc.
- **16-17** — security hardening and backups, the two things every
  admin needs and most skip until it's too late.
- **18** — the capstone: one real project, drawing on whichever prior
  modules fit what you decide to build.

## After this repo

- **`bytefortress-pwn-school`** — binary exploitation (this repo
  deepens the Linux/process/memory context pwn-school assumes)
- A future **cloud infrastructure track** (this repo's service-running
  modules translate directly)
