# Module 08 — Network Management

## Objective

Get comfortable with secure remote administration via SSH and the tools you'll reach for constantly when managing systems remotely.

## Core concepts

### SSH fundamentals

SSH (Secure Shell) is how you'll administer almost every Linux server
you don't have physical access to.

```bash
ssh user@host                    # basic connection
ssh -p 2222 user@host             # non-default port
ssh -i ~/.ssh/mykey user@host     # specify a private key
```

### Key-based authentication (the right way to do this)

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"   # generate a keypair
ssh-copy-id user@host                                # install your public key on the server
```

Once key-based auth works, disable password authentication entirely
in `/etc/ssh/sshd_config` (`PasswordAuthentication no`) — this closes
off the entire category of brute-force password attacks against SSH.

### Hardening sshd_config

Common hardening steps in `/etc/ssh/sshd_config`:

```
PermitRootLogin no
PasswordAuthentication no
Port 2222                    # obscurity, not real security, but reduces log noise from bots
AllowUsers admin_user
```

After any change: `sudo systemctl restart sshd` — and **keep your
current session open** while testing a new connection in a second
terminal, so you can revert if something's wrong.

### Remote file transfer

```bash
scp file.txt user@host:/remote/path/       # copy a file over SSH
rsync -avz localdir/ user@host:/remote/dir/   # sync directories, more efficient for repeated transfers
```

### Convenience without sacrificing security: ssh-agent and ~/.ssh/config

If your private key has a passphrase (it should), typing it every
single connection gets old fast. `ssh-agent` holds your decrypted key
in memory for your session so you only enter the passphrase once:

```bash
eval "$(ssh-agent -s)"    # start the agent for this shell session
ssh-add ~/.ssh/id_ed25519  # unlock and cache your key — prompts for passphrase once
ssh-add -l                  # confirm which keys are currently loaded
```

Most desktop Linux environments start an agent automatically at
login; on a server or minimal environment you may need the above
manually per session.

**`~/.ssh/config`** eliminates repetitive connection flags entirely —
instead of remembering the right user, port, key, and options for
every host:

```
Host lab
    HostName 192.168.1.10
    User jacob
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    ForwardX11 yes
```

```bash
ssh lab    # equivalent to: ssh -p 2222 -i ~/.ssh/id_ed25519 -X jacob@192.168.1.10
```

`ForwardX11 yes` here means you never need to remember the `-X` flag
for that host again — genuinely handy if you regularly run graphical
tools over SSH.

### SSH tunneling (brief mention)

SSH can forward ports (`-L`, `-R`) to securely tunnel other traffic
through an encrypted connection — useful for accessing a remote
service that's only bound to localhost on the far end. Worth knowing
exists; depth is beyond this module.

## Hands-on lab

1. Generate an SSH keypair and copy your public key to your lab
   server VM.
2. Confirm key-based login works, then disable password
   authentication in `sshd_config` (keeping your current session open
   as a safety net).
3. Change SSH's listening port to something non-default, restart
   sshd, and connect using the new port.
4. Use `rsync` to sync a local directory to your lab server, make a
   change locally, and sync again — observe that only the changed
   file transfers.
5. Set a passphrase on your SSH key (if it doesn't have one already),
   confirm you're prompted for it on every connection, then start
   `ssh-agent`, `ssh-add` your key, and confirm you're no longer
   prompted for the rest of your session.
6. Create a `~/.ssh/config` entry for your lab server with a short
   nickname, the right port, and your identity file, then connect
   using just `ssh <nickname>`.

## Common pitfalls

- Disabling password authentication before confirming key-based login actually works — always test the new method in a second terminal before closing your working session.
- Forgetting to update your firewall rule (module 07) after changing SSH's port — the new port needs to be explicitly allowed.
- Using `scp` for repeated syncs of large directories instead of `rsync` — `rsync` only transfers changes, dramatically faster for anything beyond a one-off copy.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [SSH Academy](https://www.ssh.com/academy/ssh) — thorough, vendor-neutral SSH reference
