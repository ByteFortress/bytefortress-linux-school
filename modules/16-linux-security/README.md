# Module 16 — Linux Security

## Objective

Apply real hardening practices to a Linux system: mandatory access control, auditing, and systematic review against a known-good baseline.

## Core concepts

### The hardening mindset

Security hardening is about reducing attack surface and limiting
blast radius, not achieving some mythical "fully secure" state. Every
module before this one already touched on hardening (SSH key-only
auth, least-privilege sudo, scoped NFS exports, restrictive
firewalls) — this module pulls those threads together and adds
systematic tools.

### Network architecture: why segmentation matters

A useful progression to reason through, each step fixing the previous
one's obvious problem:

1. **A web server and a database server, both directly on the
   internet, no firewall.** Anyone can reach either directly —
   obviously wrong.
2. **Add a single firewall in front of both.** Better, but the
   firewall is now an all-or-nothing gate — if it needs to let web
   traffic through to the web server, that same open path often
   reaches the database server too, since they're on the same
   network segment.
3. **Introduce a DMZ (demilitarized zone)** — the web server sits in
   its own network segment, reachable from the internet, while the
   database sits in a separate internal segment reachable only from
   the web server, never directly from the internet.
4. **Layer additional firewalls between each zone** — internet → DMZ
   firewall → DMZ → internal firewall → internal network — so
   compromising the web server doesn't automatically hand over the
   database too.

The general principle: **a compromised system should be a dead end,
not a stepping stone.** Every zone boundary you add is one more place
an attacker has to break through, and one more place you get a chance
to detect them.

### How security actually gets compromised

Worth being blunt about where real incidents come from, roughly in
order of how often they actually matter:

- **Social engineering** — a majority of real-world security
  incidents involve a person being tricked or making a mistake, not a
  sophisticated technical exploit. User education is a genuine
  security control, not a soft afterthought.
- **Configuration errors** — an account with no password, a service
  left at its default credentials, an overly permissive firewall
  rule.
- **Software vulnerabilities** — buffer overflows (see
  `bytefortress-pwn-school` for the deep dive) and, just as commonly
  in web applications, unsanitized user input reaching a system call:

  ```php
  system("/bin/cat " . $_POST["filename"]);
  ```

  If `$_POST["filename"]` isn't validated, a user submitting
  `report.txt; rm -rf /` (or worse) turns a simple "display a file"
  feature into arbitrary command execution. This exact class of bug
  (unsanitized input reaching a shell) is why input validation and
  parameterized/escaped calls are non-negotiable in anything that
  touches user input — a single unescaped concatenation like this one
  is a genuinely common real-world root cause.

### Mandatory Access Control: SELinux and AppArmor

Standard Linux permissions (module 02) are discretionary — the
resource owner decides who can access it. Mandatory Access Control
adds a system-wide policy that constrains programs regardless of
file ownership, containing what a compromised process can actually
do.

- **SELinux** (Red Hat-based distros) — label-based, very granular,
  steeper learning curve
- **AppArmor** (Ubuntu/Debian-based) — path-based profiles, generally
  considered easier to reason about

```bash
# AppArmor
sudo aa-status                      # see enforced/complain profiles
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx

# SELinux
sestatus
getenforce                          # Enforcing / Permissive / Disabled
```

### Auditing and log review

```bash
sudo apt install auditd
sudo auditctl -w /etc/passwd -p wa -k passwd_changes   # watch a file for writes/attribute changes
sudo ausearch -k passwd_changes                          # review triggered events
```

Beyond `auditd`, regular review of `journalctl`, `/var/log/auth.log`
(failed logins, sudo usage), and application-specific logs is a core
ongoing admin habit, not a one-time setup task. Classic log locations
worth knowing even as `journalctl` has become the modern default entry
point:

| Log | Contains |
|---|---|
| `/var/log/auth.log` (Debian) / `/var/log/secure` (Red Hat) | Authentication attempts, sudo usage |
| `/var/log/syslog` / `/var/log/messages` | General system messages |
| `/var/log/mail.log` | Mail server activity |
| `/var/log/wtmp` (via the `last` command, not readable directly) | Login history |

```bash
last -f /var/log/wtmp          # login history
lastlog                        # most recent login per user
tail -f /var/log/syslog        # follow a log live
grep "Failed password" /var/log/auth.log   # hunt for brute-force attempts
watch -n 5 'who'                # re-run a command every 5 seconds — handy for live monitoring
```

**Auditing what's actually listening on the network** — an
unexpectedly open port is one of the highest-value things to catch
early:

```bash
ss -tlnp                        # modern replacement for `netstat -ta`, shows listening ports + owning process
sudo lsof -i :PORT              # identify exactly which process owns a specific port
```

If a service turns up that you don't recognize or don't need running,
disable it (`sudo systemctl disable --now <service>`) — every
unnecessary listening service is attack surface for no benefit.

**Checking for accounts with no password set** (a genuinely common
misconfiguration on systems that were set up in a hurry):

```bash
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
```

An empty second field means no password hash at all — any account
this returns needs immediate attention.

### Benchmark-driven hardening

Rather than guessing what to harden, industry benchmarks give you a
checklist:

- **CIS Benchmarks** — detailed, distro-specific hardening guides,
  free to read (registration required)
- **Lynis** — an open-source auditing tool that scores your system
  against common hardening criteria and explains each finding

```bash
sudo apt install lynis
sudo lynis audit system
```

### iptables — the mechanics behind ufw

Module 07 covered `ufw` as the friendly frontend; here's what it's
actually driving underneath, worth knowing since you'll encounter raw
`iptables` rules constantly in older documentation and existing
production systems.

**Three default chains** apply to every packet: `INPUT` (incoming),
`OUTPUT` (outgoing), `FORWARD` (passing through, e.g. on a router —
see module 07's router lab). Each rule has a **target**: `ACCEPT`,
`DROP` (silently discard), `REJECT` (discard with a response),
`LOG`, or jump to a custom chain.

A real, complete default-deny ruleset — flush everything, default to
DROP, then explicitly allow only what's needed:

```bash
sudo iptables -F                        # flush all existing rules
sudo iptables -P INPUT DROP              # default deny incoming
sudo iptables -P FORWARD DROP            # default deny forwarding
sudo iptables -P OUTPUT ACCEPT           # usually fine to allow outgoing

sudo iptables -A INPUT -i lo -j ACCEPT                                    # always allow loopback
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT     # allow replies to our own connections
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT   # allow new SSH connections
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT        # allow ping
sudo iptables -A INPUT -j LOG --log-prefix "DROPPED: "                    # log everything else before...
sudo iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited       # ...rejecting it
```

**Default-deny (whitelist what you need) is the right posture** for
anything internet-facing — the alternative (default-accept, blocklist
specific bad things) means anything you forgot to block gets through.

```bash
sudo iptables -L -v          # view current rules and hit counters
sudo iptables -t nat -L -v   # view the NAT table specifically
```

NAT and transparent redirection (tying back to module 11's Squid
content):

```bash
# Redirect outbound port 80 into a local caching proxy (transparent caching)
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j REDIRECT --to-ports 3128

# Standard NAT/masquerading for a router forwarding a private subnet's traffic
sudo iptables -t nat -A POSTROUTING -o eth0 -s 10.0.0.0/24 -j MASQUERADE
```

### TCP wrappers: /etc/hosts.allow and /etc/hosts.deny

An older, simpler access-control layer than a full firewall, still
supported by some services (notably via `libwrap`):

```bash
cat /etc/hosts.deny
# ALL: ALL

cat /etc/hosts.allow
# sshd: 192.168.1.0/255.255.255.0
```

The pattern — deny everything by default, then explicitly allow
specific services from specific networks — is the same default-deny
philosophy as the iptables ruleset above, just at the
application-wrapper level instead of the packet-filter level. Modern
setups generally rely on the firewall for this instead, but you'll
still encounter `hosts.allow`/`hosts.deny` in the wild.

### A tour of other security tools worth knowing

- **`nmap`** — port scanning and OS fingerprinting (only ever scan
  systems you own or are authorized to test):
  ```bash
  nmap -sT target-ip          # basic TCP connect scan
  nmap -O -sV target-ip        # attempt OS and service version detection
  ```
- **John the Ripper** — password hash cracking, primarily used
  defensively to audit whether your own users' passwords are weak
  enough to crack quickly.
- **GnuPG (GPG)** — the open-source implementation of PGP, used to
  encrypt files, sign commits/releases, and verify the authenticity
  of downloaded software.
- **Kerberos** — a network authentication protocol that lets services
  and users prove their identity to each other without repeatedly
  transmitting passwords — the enterprise-grade authentication layer
  LDAP setups often pair with.

### Putting it together

A realistic hardening pass touches: SSH config (module 08), firewall
rules (module 07), least-privilege accounts (module 02), MAC policy
enforcement (this module), and a scheduled audit tool run — treat
this module as the checklist that ties the whole curriculum's
security-relevant choices together.

## Hands-on lab

1. Check whether AppArmor or SELinux is active on your distro by
   default, and review the status of the profiles/policies already
   loaded.
2. Install and run Lynis, review its report, and pick three
   recommendations to actually implement.
3. Set up an audit rule watching a sensitive file (e.g. `/etc/passwd`
   or `/etc/sudoers`) for changes, then trigger it deliberately and
   confirm it logs.
4. Review `/var/log/auth.log` (or `journalctl -u ssh`) for any failed
   login attempts — even on a lab VM with no real exposure, this
   builds the habit of actually looking.
5. Write and apply a default-deny `iptables` ruleset (flush, default
   DROP, explicitly allow loopback/established/SSH), verify SSH still
   works, then verify a non-allowed port is actually blocked from
   another machine.
6. Run `ss -tlnp` on your VM, identify every listening service, and
   disable at least one you don't actually need.
7. Check your own `/etc/shadow` for any accounts with an empty
   password field using the `awk` one-liner above (there shouldn't be
   any on a properly set-up system — confirm that's the case).

## Common pitfalls

- Running Lynis or a CIS benchmark and implementing every single recommendation blindly — some hardening steps have real usability tradeoffs; understand each one before applying it, especially anything affecting remote access.
- Setting SELinux/AppArmor policies too restrictively without testing, breaking a legitimate service, then disabling the whole subsystem out of frustration instead of fixing the specific policy — 'permissive'/'complain' mode exists for exactly this iterative tuning.
- Treating a one-time audit as 'done' — hardening and log review are ongoing practices, not a checkbox you tick once.
- Setting `iptables` default policy to DROP before an explicit ACCEPT rule for your own SSH connection is in place — exactly like the ufw lockout risk in module 07, order matters: add the allow rule for your own access first, change the default policy last.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) and [Lynis](https://cisofy.com/lynis/)
