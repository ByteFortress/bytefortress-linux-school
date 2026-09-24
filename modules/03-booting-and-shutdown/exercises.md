# Module 03 — Extra Exercises

## Exercise 1 — Boot time analysis

Run `systemd-analyze blame` and identify the three slowest-starting services on your VM. Research what each one does.

## Exercise 2 — Target isolation practice

From console access (not SSH), isolate `rescue.target`, confirm you're in a minimal environment, then return to normal operation.

## Exercise 3 — Scheduled shutdown

Schedule a shutdown 5 minutes out with a custom warning message, verify logged-in users see the warning, then cancel it.

## Exercise 4 — Build a minimal kernel

Snapshot your VM first. Using `make menuconfig` starting from your
current running config (`cp /boot/config-$(uname -r) .config`), go
through the device driver sections and disable support for hardware
you know your VM doesn't have (most physical hardware drivers won't
apply to a VM at all — leave `initrd`/RAM disk support and your
virtual disk controller's driver enabled, or your new kernel won't
boot). Compile it, install it, and boot from it. Compare `du -sh` on
the old vs. new `/lib/modules/` directories, and write up what you
disabled and why in `NOTES.md`.

