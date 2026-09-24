# Module 05 — Linux Filesystem

## Objective

Go deeper on the Linux filesystem: mounting, filesystem types, disk partitioning, and basic LVM — beyond the FHS overview from foundations.

## Core concepts

### The Filesystem Hierarchy Standard, in more depth

Beyond `/home`, `/etc`, `/var` (covered in foundations), know:

- `/opt` — optional/third-party software packages
- `/srv` — data for services this system provides
- `/mnt`, `/media` — mount points for manually and auto-mounted
  filesystems respectively
- `/proc`, `/sys` — virtual filesystems exposing kernel/process
  information (not real files on disk)

### Mounting

```bash
lsblk                          # list block devices
sudo mount /dev/sdb1 /mnt/data # mount a device to a mount point
sudo umount /mnt/data          # unmount
```

For mounts that should persist across reboots, add an entry to
`/etc/fstab` — a misconfigured fstab entry can prevent a system from
booting normally, so edit carefully and test with `mount -a` before
rebooting.

**When `umount` refuses because the device is "busy":**

```bash
sudo fuser -mv /mnt/data     # show every process currently using files on that mount
sudo lsof +D /mnt/data        # alternative: list open files under that path
```

Kill or close whatever's shown before retrying `umount` — this is one
of the most common real-world mounting frustrations, and `fuser` is
the direct answer to "why won't this unmount."

### Common filesystem types

- **ext4** — the long-standing Linux default, mature and reliable
- **XFS** — good for large files and high-performance workloads (RHEL
  default)
- **Btrfs** — supports snapshots and built-in RAID-like features
- **FAT32/exFAT/NTFS** — for interoperability with Windows/removable
  media

### Partitioning and LVM basics

Traditional partitioning (via `fdisk`/`parted`) creates fixed-size
partitions.

```bash
sudo fdisk -l              # list all disks and their partitions
sudo fdisk /dev/sda        # enter fdisk's interactive menu for a specific disk
```

Inside the interactive menu: `m` (help/menu), `p` (print current
partition table), `n` (create a new partition), `d` (delete a
partition), `t` (change a partition's type ID), `w` (write changes to
disk — **nothing is actually applied until you press `w`**, so it's
safe to explore with `p` and `m` first).

LVM adds a flexible layer on top of raw partitions:

```
Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV)
```

This lets you resize storage without the rigid constraints of
traditional partitions — e.g. extending a logical volume across
multiple physical disks without repartitioning everything.

### Disk usage tools

```bash
df -h          # filesystem-level usage
du -sh /path   # directory-level usage
ncdu           # interactive disk usage explorer (install separately)
```

### File types (there are more than "file" and "directory")

Linux recognizes 7 file types, visible as the first character of
`ls -l`'s permission string:

| Symbol | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device (e.g. `/dev/tty`) |
| `b` | Block device (e.g. `/dev/sda`) |
| `s` | Unix socket |
| `p` | Named pipe (FIFO) |

```bash
ln -s /path/to/original /path/to/link   # create a symbolic link
```

### Special permission bits

Beyond the standard read/write/execute:

- **setuid** — an executable runs with its *owner's* privileges
  rather than the invoking user's (e.g. `passwd` is setuid root, so a
  normal user can update `/etc/shadow` through that one controlled
  path)
- **setgid** on a file — same idea for group; on a *directory*
  (covered in module 04), new files inherit the directory's group
- **sticky bit** — on a shared directory (classically `/tmp`), only a
  file's owner (or root) can delete/rename it, even if others have
  write access to the directory

```bash
chmod u+s /path/to/binary    # set setuid
chmod g+s /path/to/dir        # set setgid
chmod +t /path/to/dir         # set sticky bit
ls -l /usr/bin/passwd         # look for 's' in the owner execute position
```

### umask — controlling default permissions

`umask` sets which permission bits are *removed* from the default
when a new file or directory is created.

```bash
umask          # show current umask, e.g. 0022
umask 027      # set a stricter default (owner: full, group: read-only, others: nothing)
```

### Safely handling filenames with spaces or special characters

A classic Linux gotcha: piping `find` output into a command that
splits on whitespace will break on filenames containing spaces. The
safe pattern uses null-terminated output:

```bash
find /var/www -size +4M -print0 | xargs -0 ls -l
find . -name "*.tmp" -print0 | xargs -0 rm
```

`-print0` and `xargs -0` together use a null byte (which can never
appear in a filename) as the separator instead of whitespace — always
reach for this pattern when scripting anything that processes `find`
results.

## Hands-on lab

1. Attach a second virtual disk to your VM (most hypervisors
   support this easily).
2. Partition it with `fdisk` or `parted`.
3. Format it with ext4: `sudo mkfs.ext4 /dev/sdb1`.
4. Create a mount point and mount it manually.
5. Add a persistent entry to `/etc/fstab`, then test it with
   `sudo mount -a` (safer than rebooting to find out it's broken).
6. Reboot and confirm it mounted automatically.
7. Find every setuid binary on your system with
   `find / -perm -4000 -type f 2>/dev/null` and pick two to research —
   explain why each one needs to run with elevated privileges.
8. Deliberately hold a mount point busy (e.g. `cd` into it in one
   terminal), attempt to `umount` it from another, observe the
   failure, then use `fuser -mv` to identify what's holding it open.

## Common pitfalls

- Writing a bad `/etc/fstab` entry and rebooting without testing — this can drop you into an emergency shell. Always `mount -a` first to catch errors safely.
- Formatting the wrong device by mistyping `/dev/sdX` — always double-check with `lsblk` before any destructive disk operation.
- Confusing `df` (filesystem-level) with `du` (directory-level) when troubleshooting 'disk full' issues — a filesystem can show full via `df` while `du` on visible files doesn't add up, often due to deleted-but-still-open files.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html) for the FHS itself; Arch Wiki's LVM page for LVM depth
