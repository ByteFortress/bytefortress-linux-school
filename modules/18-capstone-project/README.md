# Module 18 — Capstone Project

## Objective

Design, build, and document a real system administration project that
draws on multiple modules from this repo — the difference between
having done 17 guided exercises and being able to build something
nobody handed you a script for.

## Format

Much lighter than an academic course: no page-count minimums, no
LaTeX. What matters is going through the same real arc as any actual
sysadmin project:

1. **Proposal** (a paragraph, not a page) — what problem are you
   solving, and roughly how?
2. **Build it.**
3. **Write it up** in a `README.md` for the project itself, covering:
   - **Introduction** — what problem this solves and why it matters
   - **Technical details** — how you actually built it, what you'd
     tell another admin trying to understand your setup
   - **Result** — does it work? How do you know? (Same "prove it"
     standard as every other module's hands-on lab)
   - **Lessons learned** — what surprised you, what you'd do
     differently
   - **Next steps** — what you'd add with more time
4. **Share it** in this repo's `SHOWCASE.md`.

## Project ideas

Organized by which modules they draw on most. Pick one, combine two,
or come up with your own — "any topic related to Linux system
administration" is a genuinely fine scope.

### Automation
- Fully automated server provisioning: given a fresh VM, one script
  (or an Ansible playbook, if you want to go beyond bash) gets it to
  a known-good, hardened, service-running state with zero manual
  steps. (Modules 02, 04, 09, 16)
- A "disaster recovery drill" script: simulate a server dying, then
  time and automate how fast you can stand up an identical
  replacement from your backups and provisioning scripts. (Modules
  09, 17, and this one)

### Monitoring and visualization
- A system health dashboard: collect uptime, load, process count,
  network throughput via cron/systemd timer, store it, and build a
  simple web front-end to display it over time instead of just
  current numbers. (Modules 06, 09 or 10, 11)
- Network topology visualization: discover and graph what's actually
  connected to what on your lab network, rather than just listing IPs
  in text. (Module 07)
- A disk usage analysis tool with visualization instead of raw `du`
  output. (Module 05)

### Security
- A log analyzer that parses auth/access logs for signs of brute-force
  attempts or other suspicious patterns, and alerts you. (Modules 16,
  09 or 10) — this is a lightweight, from-scratch version of what
  tools like Fail2ban or a SIEM do.
- A promiscuous-mode NIC detector across a group of machines — a
  classic "is someone sniffing traffic they shouldn't be" check.
  (Module 07, 16)
- A minimal Tripwire-style file integrity checker: hash a set of
  critical files, store the hashes, and alert on any change. (Modules
  16, 06 from `bytefortress-foundations-school` for the hashing
  concept)

### Web-facing tools
- A web front-end for user account management (create/disable/delete
  accounts through a web UI instead of the CLI). (Modules 02, 11)
- A web front-end for reviewing and adjusting your own procmail/Sieve
  mail filtering rules. (Modules 11, 13)

### Backup and file systems
- An incremental backup system built from scratch on top of `rsync`
  or `restic`, with your own retention policy and a scheduled job.
  (Module 17)
- Explore FUSE (Filesystem in Userspace) and build a minimal custom
  filesystem — even something simple like a filesystem that
  transparently compresses files is a genuinely deep exercise.
  (Module 05)

### Directory services and integration
- Full LDAP-backed authentication across two or more lab VMs, with a
  real client (`sssd`) configuration rather than just the server-side
  setup from module 15.
- Authentication integration between a Linux box and a Windows VM (if
  you have one) via LDAP or Samba.

### Minimal project (if you want something smaller in scope)

Find an existing piece of sysadmin-related open-source software,
install it, actually use it for a while, identify a real weakness or
missing feature, and either patch the code or write a wrapper script
that fixes the gap. Document what you found and what you built. This
is a legitimate, smaller-scope capstone — depth of understanding
matters more than size of build.

## What "done" looks like

You don't need every idea's stretch goals — you need one project,
actually working, that you can explain clearly to someone else and
that you'd be comfortable pointing to as evidence you can do real
sysadmin work. A working, well-documented small project beats an
ambitious, half-finished big one.

## Share it

Once it's working and written up, add it to this repo's
[`SHOWCASE.md`](../../SHOWCASE.md) — future learners deciding what to
build benefit enormously from seeing what others actually did.
