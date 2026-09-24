# Module 14 — NFS

## Objective

Understand and configure NFS (Network File System) to share directories between Linux machines over a network.

## Core concepts

### What NFS is for

NFS lets one machine (the server) export a directory that other
machines (clients) can mount as if it were local. Common uses: shared
home directories across multiple machines, shared application data,
centralized storage for a small cluster.

Created by Sun Microsystems in 1985. Worth knowing the version
history since it explains some persistent quirks: NFSv2 was
synchronous only (slow — every write waited for server
acknowledgment); NFSv3 added async writes (faster, still the common
baseline); NFSv4 added strong security options, ACLs, and better
firewall-friendliness (a single port instead of the older
RPC-portmapper dance). Use NFSv4 for anything new unless you have a
specific reason not to.

### Server setup

```bash
sudo apt install nfs-kernel-server
```

Define exports in `/etc/exports`:

```
/srv/shared   192.168.1.0/24(rw,sync,no_subtree_check,root_squash)
```

- `rw` — read-write (use `ro` for read-only)
- `sync` — writes are committed before replying (safer, matches NFS
  spec)
- `no_subtree_check` — improves reliability, standard recommendation
  for most setups
- `root_squash` (the default, worth stating explicitly) — maps a
  connecting client's root user down to an unprivileged user
  (`nobody`) on the server, so a compromised or malicious client's
  root account doesn't get root access to your exported files. Only
  disable this (`no_root_squash`) when you specifically need it and
  understand the tradeoff.

```bash
sudo exportfs -ra          # reload exports after editing
sudo systemctl restart nfs-kernel-server
```

### Client setup

```bash
sudo apt install nfs-common
sudo mount 192.168.1.10:/srv/shared /mnt/shared
```

For a persistent mount, add an entry to `/etc/fstab` (as in module
05):

```
192.168.1.10:/srv/shared   /mnt/shared   nfs   defaults   0 0
```

**Mount options worth knowing** for handling a flaky or unreachable
server: `soft,timeo=5` fails fast after 5 tenths of a second of
retries instead of hanging forever (`hard`, the default, retries
indefinitely — usually correct for critical shares, but can hang a
client badly if the server disappears). `intr` allows a hung NFS
operation to be interrupted (e.g. with Ctrl-C) rather than blocking
uninterruptibly.

### Security considerations

NFS traditionally trusts the client to enforce user identity (UID/GID
matching), which is a real limitation — a client can claim to be any
UID unless you add Kerberos-based authentication (NFSv4 supports this
but it adds real setup complexity). For a lab or trusted internal
network this is usually an acceptable tradeoff; for anything exposed
more broadly, restrict exports tightly by IP/subnet and consider
NFSv4 with Kerberos.

### Checking what's exported/mounted

```bash
showmount -e <server-ip>    # from a client, see what a server exports
mount | grep nfs            # see current NFS mounts
nfsstat -s                  # server-side statistics
nfsstat -c                  # client-side statistics
```

### Automatic (on-demand) mounting with autofs

Rather than mounting every NFS share permanently at boot (module 05's
`/etc/fstab` approach), `autofs` mounts a share only when something
actually accesses it, and unmounts it after a period of inactivity —
useful when you have many possible shares and don't want them all
consuming resources or hanging boot if one server is unreachable.

```bash
sudo apt install autofs
```

```
# /etc/auto.master — maps a mount point to a map file
/home    /etc/auto.home

# /etc/auto.home — the indirect map itself
*        -soft,timeo=5    192.168.1.10:/home/&
```

The `*` matches any subdirectory name under `/home`, and `&` in the
source substitutes that matched name — so accessing `/home/blackcat`
automatically mounts `192.168.1.10:/home/blackcat`, on demand, with no
explicit per-user fstab entry needed.

## Hands-on lab

1. Set up an NFS server on one VM, exporting a test directory to
   your lab subnet only (not to the world).
2. From a second VM, mount that export and confirm you can read/write
   to it.
3. Create a file from the client, then confirm it's visible from the
   server directly.
4. Add the mount to the client's `/etc/fstab` for persistence, and
   verify it remounts correctly after a reboot.
5. Test a UID mismatch: create a user with the same username but a
   different UID on each machine, and observe how file ownership
   looks from each side.
6. From the client, try creating a file as root on the mounted share
   and check its ownership on the server — confirm `root_squash` is
   mapping it to an unprivileged user rather than server-side root.
7. Set up `autofs` for the same share instead of a static `fstab`
   entry, and confirm the share mounts only when you actually access
   it.

## Common pitfalls

- Exporting a directory with no IP/subnet restriction, making it available to anyone who can reach the server — always scope exports to your specific trusted network.
- Forgetting `exportfs -ra` after editing `/etc/exports` — the running server doesn't automatically pick up file changes.
- Assuming NFS enforces real user authentication the way a properly configured application would — plain NFSv3 trusts client-reported UIDs, which matters for anything beyond a fully trusted internal network.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Ubuntu NFS documentation](https://ubuntu.com/server/docs/service-nfs)
