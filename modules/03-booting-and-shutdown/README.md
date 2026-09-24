# Module 03 — Booting and Shutdown

## Objective

Understand what actually happens between pressing power and reaching a login prompt, and how to control system state safely.

## Core concepts

### The boot sequence, roughly

1. **Firmware (BIOS/UEFI)** — hardware initialization, then hands off
   to a bootloader
2. **Bootloader (GRUB)** — loads the Linux kernel into memory
3. **Kernel initialization** — sets up hardware, mounts the root
   filesystem
4. **Init system (systemd on most modern distros)** — starts every
   other process and service in the correct order

**What's actually happening at step 1, mechanically:** on legacy
BIOS systems, the firmware reads the first 512 bytes of the boot
disk — the Master Boot Record (MBR) — which contains just enough code
to locate and load the real bootloader (GRUB) from a specific disk
partition. UEFI systems replace this with a more capable boot
process (the EFI System Partition), but the concept — firmware hands
off to a small loader, which hands off to the real bootloader — is
the same idea either way.

### Recovering a system that won't boot normally

If a system won't boot due to a filesystem problem or a bad config,
you drop into rescue/single-user mode (see the target table below).
Two commands you'll actually use once you're there:

```bash
mount -o remount,rw /       # root often mounts read-only in rescue mode; this makes it writable
fsck -y /dev/sda1           # check and auto-repair filesystem errors on the affected partition
```

This is the real reason rescue/single-user mode exists — it gives you
a minimal environment where you can fix the thing that's preventing a
normal boot, without the normal boot process itself being able to
run.

### systemd targets (the modern replacement for runlevels)

| Target | Roughly equivalent to | Purpose |
|---|---|---|
| `poweroff.target` | Runlevel 0 | Shut down |
| `rescue.target` | Runlevel 1 | Single-user/maintenance mode |
| `multi-user.target` | Runlevel 3 | Normal operation, no GUI |
| `graphical.target` | Runlevel 5 | Normal operation with GUI |
| `reboot.target` | Runlevel 6 | Reboot |

```bash
systemctl get-default              # see current default target
sudo systemctl set-default multi-user.target
sudo systemctl isolate rescue.target   # switch target now
```

> **Historical note:** before systemd, Linux used SysV init with
> numbered runlevels (0-6) and startup scripts in
> `/etc/rc.d/rc[0-6].d/`, managed with `chkconfig`. You may still
> encounter this on very old systems or in older documentation — the
> systemd targets above are the direct conceptual replacement, and
> `runlevel` still works on many systems as a backward-compatible
> query even under systemd.

### Controlling shutdown/reboot properly

```bash
sudo shutdown -h now      # halt immediately
sudo shutdown -r now      # reboot immediately
sudo shutdown -h +10      # halt in 10 minutes, warns logged-in users
sudo reboot               # simpler reboot shortcut
```

Never just pull power on a live system unless it's genuinely
unresponsive — improper shutdown risks filesystem corruption,
especially on systems without journaling filesystems (modern
filesystems like ext4 are more resilient, but it's still bad
practice).

### GRUB basics

GRUB's config is generated, not hand-edited directly:

```bash
sudo nano /etc/default/grub        # edit settings
sudo update-grub                   # regenerate actual config (Debian/Ubuntu)
# or: sudo grub2-mkconfig -o /boot/grub2/grub.cfg  (RHEL-based)
```

### Advanced: compiling your own kernel

Most admins never need to do this, but understanding the process
demystifies what "the kernel" actually is and deepens everything
this module covers. The steps: download a kernel source tree,
configure it, compile it, compile its modules, install both, then
add a GRUB entry.

```bash
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev

cd /usr/src
sudo wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.tar.xz
sudo tar xf linux-6.6.tar.xz
cd linux-6.6

make menuconfig      # interactive config — start from your distro's
                      # existing config as a base rather than from scratch:
                      # cp /boot/config-$(uname -r) .config

make -j$(nproc)              # compile the kernel (parallelized across all cores)
make -j$(nproc) modules      # compile loadable modules
sudo make modules_install     # install modules to /lib/modules/<version>/
sudo make install             # install the kernel to /boot and update GRUB
```

**Before rebooting**, take a VM snapshot — a kernel missing a driver
your virtual hardware needs (commonly the storage controller or
`initrd`/`initramfs` support) will fail to boot, and a snapshot turns
that from a crisis into a two-minute revert.

```bash
uname -r                      # confirm which kernel you're actually running after reboot
lsmod                         # compare loaded modules between old and new kernel
du -sh /lib/modules/*         # compare module directory sizes
```

A genuinely useful practical motivation for doing this at all: a
kernel built with only the drivers your specific hardware needs boots
faster and uses less memory than a generic distro kernel carrying
support for hardware you'll never have.

## Hands-on lab

1. Check your current default systemd target.
2. Temporarily isolate `rescue.target`, observe what happens (most
   services stop), then return to `multi-user.target`.
3. Use `systemd-analyze` to see how long your last boot took, and
   `systemd-analyze blame` to see which service took longest to
   start.
4. Schedule a delayed shutdown with a warning message, then cancel it
   before it triggers (`sudo shutdown -c`).
5. From `rescue.target`, practice the recovery pattern: remount `/`
   read-write and run `fsck` against your root partition (on a
   healthy filesystem this just reports "clean," which is fine — the
   point is building comfort with the commands before you need them
   for real).
6. **Root password recovery, for real.** Have a friend/colleague (or
   do it yourself and genuinely forget) change your VM's root
   password to something you don't know. Recover it:
   - Reboot the VM and interrupt the GRUB countdown.
   - Edit the boot entry to append `single` (or on newer GRUB,
     `systemd.unit=rescue.target`) to the kernel line.
   - Boot into the resulting shell and run `passwd` to set a new root
     password.
   - Reboot normally and confirm the new password works.

   This is exactly the pattern from step 5 above, applied to the
   actual real-world reason rescue mode exists. If you can do this
   without hesitation, you understand this module.

## Common pitfalls

- Editing `/boot/grub/grub.cfg` directly instead of `/etc/default/grub` + `update-grub` — your changes get overwritten the next time GRUB regenerates its config.
- Isolating `rescue.target` on a remote VM you're SSH'd into and losing your connection — do this lab on a VM with local/console access, not a remote-only box.
- Force-powering off a VM to 'save time' instead of a proper shutdown — build the correct habit now.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Arch Wiki — systemd](https://wiki.archlinux.org/title/Systemd) — thorough and distro-agnostic in its explanations even if you're not on Arch
