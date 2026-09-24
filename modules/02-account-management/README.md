# Module 02 — Account Management

## Objective

Understand how Linux manages users, groups, and privilege escalation — the foundation of every other access-control topic in this curriculum.

## Core concepts

### Users and UIDs

Every user has a numeric UID. UID 0 is always root. System accounts
(for services) typically use low UIDs; regular human users start at
1000 on most distros (older systems commonly started at 500 — check
`/etc/login.defs` for your system's actual starting point).

```bash
sudo useradd -m -s /bin/bash newuser   # -m creates home dir, -s sets shell
sudo passwd newuser                     # set password
sudo userdel -r newuser                 # -r also removes home dir
```

`useradd` is the modern one-liner, but it's worth knowing what it's
actually doing under the hood, since you'll occasionally need to do
this by hand (e.g. restoring an account from a backup):

```bash
sudo mkdir /home/newuser
sudo cp -a /etc/skel/. /home/newuser        # /etc/skel = template files every new home dir starts with
sudo chown -R newuser:newuser /home/newuser
sudo chmod 711 /home/newuser
sudo vipw                                    # safely hand-edit /etc/passwd (like visudo, but for passwd)
sudo passwd newuser
```

`/etc/skel` is worth a deliberate look — it's the template directory
copied into every new home directory. Adding a company-standard
`.bashrc` or a README there means every new account gets it
automatically.

**Finding orphaned files after deleting a user:**

```bash
sudo find / -nouser -xdev    # find files with no matching UID left on the system
```

`userdel foo` (without `-r`) leaves the home directory behind — this
command is how you find that debris later, or find leftover files
from any deleted account you don't remember cleaning up fully.

### Groups

Groups let you grant permissions to multiple users at once. Every
user has a primary group and can belong to additional secondary
groups.

```bash
sudo groupadd developers
sudo usermod -aG developers newuser     # -a = append, don't replace existing groups
groups newuser                          # show a user's groups
```

### Where account data actually lives

**`/etc/passwd`** — one line per user, world-readable by design (many
programs need to look up usernames/UIDs without root privilege — the
actual secret lives in `/etc/shadow` instead). Seven colon-separated
fields:

```
foo:x:1000:1000:Foo Bar:/home/foo:/bin/bash
```

| Field | Meaning |
|---|---|
| `foo` | Login name (case-sensitive, conventionally lowercase) |
| `x` | Placeholder — real password hash lives in `/etc/shadow` |
| `1000` | UID |
| `1000` | Default/primary GID |
| `Foo Bar` | GECOS field — full name and other info (`finger foo` reads this) |
| `/home/foo` | Home directory |
| `/bin/bash` | Login shell (must be listed in `/etc/shells` to be valid for some tools) |

**`/etc/shadow`** — one line per user, readable only by root. Nine
colon-separated fields:

```
foo:$6$randomsalt$hashvalue...:19800:0:99999:7:::
```

| Field | Meaning |
|---|---|
| Login name | Matches `/etc/passwd` |
| Encrypted password | The actual hash (`$6$` prefix = SHA-512) |
| Last change | Days since Jan 1 1970 the password was last changed |
| Min days | Minimum days between password changes |
| Max days | Maximum days before password must change |
| Warn days | Days before expiration to start warning the user |
| Inactive days | Days after expiration before the account is disabled |
| Expire date | Absolute account expiration date |
| Reserved | Unused |

```bash
sudo usermod -e 2027-06-26 foo   # set an account expiration date directly
```

**`/etc/group`** — one line per group, four colon-separated fields:

```
installers:x:1200:foo,bar
```

Group name, password placeholder (`x`, real group passwords are rare
and live in `/etc/gshadow`), GID, and a comma-separated member list
(no spaces).

### Disabling an account (without deleting it)

Sometimes you need to freeze an account rather than delete it —
someone's on leave, or you're investigating something and don't want
to destroy evidence by removing the account outright.

```bash
sudo usermod -L foo        # lock: prefixes the password hash with '!' — login blocked
sudo usermod -U foo        # unlock: reverses it
sudo usermod -e 2024-01-01 foo   # or set an expiration date in the past
```

A more permanent-feeling lock: change the login shell to something
that refuses interactive login entirely:

```bash
sudo usermod -s /usr/sbin/nologin foo
```

### sudo vs su

- `su` switches to another user entirely (traditionally root),
  requiring that user's password
- `sudo` runs a single command as another user (usually root), using
  *your own* password, and logs what was run — this is why `sudo` is
  generally preferred for auditability

Configure who can use `sudo` and for what via `/etc/sudoers`, always
edited with `visudo` (which validates syntax before saving — a
malformed sudoers file edited directly can lock out all sudo access).

**Beyond a single blanket rule**, `sudoers` supports aliases for
readable, maintainable rules across a team:

```
Host_Alias   WEBSERVERS = web1, web2
Cmnd_Alias   NETDEBUG = /usr/sbin/tcpdump, /usr/sbin/wireshark

blackcat   ALL=(root) ALL                  # full sudo, any host
bar     WEBSERVERS = NETDEBUG           # bar can only run those two commands, only on those hosts
```

This is the real-world version of the least-privilege sudo exercise
from this module's lab — instead of granting `ALL=(root) ALL` to
everyone (the single most common sudoers misconfiguration in
practice), scope both the commands and the hosts a rule applies to.

### PAM (Pluggable Authentication Modules)

PAM is the framework Linux uses to actually perform authentication —
it's why you can configure things like password complexity
requirements, account lockout after failed attempts, or 2FA, without
modifying every application individually. Covered conceptually here;
deep PAM configuration is beyond this module's scope.

## Hands-on lab

1. Create a new user with a home directory and bash as their shell.
2. Create a new group, add your new user to it.
3. Inspect `/etc/passwd` and `/etc/group` and find the lines
   corresponding to what you just created.
4. Use `visudo` to grant your new user passwordless `sudo` access to
   exactly one command (e.g. `systemctl restart nginx`) — practice
   the principle of least privilege rather than granting full sudo.
5. Switch to that user (`su - newuser`) and confirm they can run that
   one command with `sudo` but not others.
6. Lock your test user's account with `usermod -L`, confirm login is
   blocked, then unlock it with `usermod -U`.
7. Delete the account without the `-r` flag, then use
   `find / -nouser -xdev` to locate the orphaned home directory it
   left behind.

## Common pitfalls

- Editing `/etc/sudoers` directly with a text editor instead of `visudo` — a syntax error can break `sudo` entirely, including your ability to fix it.
- Forgetting `-m` when creating a user, resulting in no home directory.
- Confusing a user's primary group with secondary groups when troubleshooting permission issues — check both with `id <username>`.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Linux Journey — Grasshopper section](https://linuxjourney.com/) covers users/groups/permissions in depth
