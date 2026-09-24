# Module 14 — Extra Exercises

## Exercise 1 — Restricted export

Configure an export that's readable by your whole lab subnet but writable only from one specific client IP, and verify the restriction actually works from a second client.

## Exercise 2 — UID mismatch investigation

Deliberately create the UID mismatch scenario from the core lab and write up, in your own words, why this matters for a security-conscious NFS deployment.

## Exercise 3 — NFS vs local performance

Time writing a moderately large file to a local disk versus to an NFS mount, and note the difference — useful context for when NFS is and isn't the right tool.

## Exercise 4 — soft vs hard mounts when the server disappears

This is the exercise that makes the `soft`/`hard` mount option
distinction concrete instead of theoretical.

1. Mount your NFS export on the client with `soft,timeo=5` (in
   `/etc/fstab` or via a manual `mount -o`).
2. On the server, bring down the network interface the export is
   served over (e.g. `sudo ip link set eth1 down`), simulating the
   server disappearing.
3. On the client, run `ls -l /mnt/shared` and `df` — time how long
   they take and note what they report. With `soft`, they should fail
   relatively quickly rather than hang.
4. Bring the server's interface back up, remount with `hard,intr`
   instead, then repeat steps 2-3. With `hard`, the same commands
   should hang indefinitely while the server is unreachable (that's
   the point of `hard` — never silently give up on a write) —
   confirm you can still interrupt the hung command with Ctrl-C
   thanks to the `intr` option.
5. Write up which option you'd actually choose for a share holding
   critical data that must never silently fail a write, versus one
   used for a nice-to-have cache directory where hanging the client
   is worse than a failed read.

