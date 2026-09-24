# Module 05 — Extra Exercises

## Exercise 1 — Safe fstab practice

Add a deliberately incorrect fstab entry (pointing to a nonexistent device), run `mount -a`, observe the error without rebooting, then fix it.

## Exercise 2 — Disk usage mystery

Fill a directory with several large files, delete one while a process still has it open (e.g. `tail -f` it in another terminal), and observe the `df` vs `du` discrepancy this creates.

## Exercise 3 — Basic LVM setup

On a spare virtual disk, set up a PV, VG, and LV, format it, mount it, then practice extending the LV's size.

