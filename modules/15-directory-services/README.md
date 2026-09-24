# Module 15 — Directory Services

## Objective

Understand centralized identity management via LDAP — the modern replacement for the legacy NIS approach this module was originally built around.

## Core concepts

### Why centralized identity management matters

Managing separate local user accounts on every single server doesn't
scale past a handful of machines. A directory service centralizes
user/group data so every system authenticates against one source of
truth.

Three questions make the problem concrete — imagine a small network
of machines with no directory service at all:

1. You create user `foo` on one machine. Can `foo` log into a
   *different* machine on the network?
2. `foo` changes their password on one machine. Does that new
   password work on the other machines?
3. `foo` creates a file on one machine. Can `foo` access that file
   from another machine? (This third one is really module 14's
   problem — NFS — but it's the same underlying motivation: one
   source of truth instead of N disconnected copies.)

Without a directory service, the answer to 1 and 2 is "no" — you'd
need to manually create and update the account on every single
machine. That's the exact gap NIS, and later LDAP, were built to
close.

### NIS — legacy context only

NIS (Network Information System) was Sun's original answer to this
problem, distributing `/etc/passwd`-style data across a network. It's
considered obsolete for production use today: NIS transmits data
(including password hashes) without encryption and has weak
authentication. It's covered here only so the history/terminology
makes sense — do not deploy NIS in anything beyond a historical
curiosity lab.

**A bit of trivia that explains a lot of confusing old documentation:**
NIS was originally called "Sun Yellow Pages," and had to be renamed
for trademark reasons — but every NIS command still starts with `yp`
as a result: `ypserv` (the server daemon), `ypbind` (the client
daemon), `ypcat` (dump a NIS map's contents), `ypwhich` (show which
server you're bound to), `yppasswd` (change your password
NIS-wide). If you ever see a `yp*` command in old sysadmin
documentation, now you know why.

Mechanically, NIS worked as a master/slave model: a master server held
the authoritative data, and changes had to be explicitly pushed to
slave servers with `yppush` (or pulled with `ypxfr` from the slave
side) — there was no automatic real-time sync, which was itself a
source of the stale-data problems that pushed the industry toward
LDAP's more robust replication model.

### LDAP — the modern approach

LDAP (Lightweight Directory Access Protocol) stores directory
information (users, groups, and much more) in a hierarchical
structure and supports proper authentication and encryption (LDAPS/
StartTLS).

```bash
sudo apt install slapd ldap-utils
sudo dpkg-reconfigure slapd     # initial configuration wizard
```

### Basic LDAP concepts

- **DN (Distinguished Name)** — the unique path to an entry, e.g.
  `uid=jacob,ou=people,dc=example,dc=lab`
- **Entries** are organized in a tree, rooted at your domain
  (`dc=example,dc=lab` for `example.lab`)
- **Object classes** define what attributes an entry can/must have
  (e.g. `posixAccount` for a Unix user entry)

### Adding an entry (via LDIF)

```
dn: uid=blackcat,ou=people,dc=example,dc=lab
objectClass: inetOrgPerson
objectClass: posixAccount
uid: blackcat
cn: Blackcat
sn: Fortress
uidNumber: 2001
gidNumber: 2001
homeDirectory: /home/blackcat
```

```bash
ldapadd -x -D "cn=admin,dc=example,dc=lab" -W -f newuser.ldif
```

### Querying

```bash
ldapsearch -x -b "dc=example,dc=lab" "(uid=blackcat)"
```

### Connecting clients to authenticate against LDAP

This involves configuring `nsswitch.conf` and PAM (from module 02) to
check LDAP in addition to or instead of local files. On modern
systems this is done via `sssd` rather than raw LDAP client
configuration — `sssd` handles the caching, connection management,
and PAM/NSS integration for you.

```bash
sudo apt install sssd sssd-ldap
```

A minimal `/etc/sssd/sssd.conf`:

```ini
[sssd]
services = nss, pam
domains = example.lab

[domain/example.lab]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://192.168.1.10
ldap_search_base = dc=example,dc=lab
```

```bash
sudo chmod 600 /etc/sssd/sssd.conf   # sssd refuses to start if this file is world-readable
sudo systemctl enable --now sssd
```

Then point NSS at it in `/etc/nsswitch.conf` (`passwd: files sssd`,
same `files`-first-then-remote pattern as `/etc/hosts` resolution from
module 07), and enable `sssd` as a PAM source (`authselect enable-feature
with-sssd` on Red Hat-family systems, or `pam-auth-update` on Debian/
Ubuntu). This module's hands-on lab has you set this up and prove it
end to end.

## Hands-on lab

1. Install and initialize OpenLDAP (`slapd`) on a lab VM.
2. Create an LDIF file defining a new user entry and add it with
   `ldapadd`.
3. Query for that user with `ldapsearch`.
4. Add a group entry and add your user to it via LDIF.
5. Configure a second VM to authenticate against this LDAP server via
   `sssd` (install `sssd`, `sssd-ldap`, and configure `/etc/sssd/sssd.conf`
   to point at your LDAP server and search base, then
   `authselect` or `pam-config` to enable `sssd` as an auth source).
6. **The real proof it's working**: create a brand new user via LDAP
   (an `ldapadd` LDIF, as in step 2) on the server only. On the
   *client* VM, confirm `grep <username> /etc/passwd` finds
   **nothing** — the account doesn't exist locally at all — and then
   log in as that user anyway. If the login succeeds despite the
   account being absent from the client's own `/etc/passwd`, you've
   just proven centralized authentication actually works, not just
   that the LDAP server responds to queries.

## Common pitfalls

- Confusing LDAP's tree/DN structure with a normal filesystem path — it's hierarchical but the ordering and syntax rules (commas, RDN components) are LDAP-specific and easy to get wrong at first.
- Deploying NIS thinking it's 'simpler' for a real (non-lab) use case — its lack of encryption is a genuine security problem, not just an academic concern.
- Forgetting `objectClass` requirements — LDAP entries require the right object classes to accept certain attributes, and a missing one produces a cryptic schema violation error rather than a clear message.
- Leaving `/etc/sssd/sssd.conf` at default permissions — `sssd` silently refuses to start if the file is readable by anyone but root, and the resulting "why won't this start" confusion sends most people to the logs before they check `ls -l` on the config file itself.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [OpenLDAP documentation](https://www.openldap.org/doc/)
