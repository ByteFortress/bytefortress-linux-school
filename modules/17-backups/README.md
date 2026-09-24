# Module 17 — Backups

## Objective

Understand backup strategy fundamentals and implement a real, tested backup and restore process — the module most admins skip until it's too late.

## Core concepts

### The 3-2-1 rule

A widely used baseline: keep **3** copies of your data, on **2**
different types of media, with **1** copy off-site. The exact numbers
matter less than the principle — a single backup on the same disk as
the original protects against almost nothing.

### Backup types

- **Full** — everything, every time. Simple, slow, storage-heavy.
- **Incremental** — only what changed since the last backup (full or
  incremental). Fast, storage-efficient, but restoring requires the
  full chain to be intact.
- **Differential** — everything changed since the last *full* backup.
  A middle ground: faster to restore than incremental (only need the
  last full + last differential), slower to create than incremental.

### Versioning individual config files with RCS

Before reaching for a full backup tool, sometimes you just want
version history on a single config file you're about to edit — RCS
(Revision Control System), a genuine predecessor to Git, still ships
on most Linux systems for exactly this:

```bash
cd /etc
ci hosts          # check in hosts as a new revision (prompts for a description)
co -l hosts        # check it back out, locked for editing
rcsdiff hosts       # show what's changed since the last checked-in revision
```

This creates an `RCS/hosts,v` file holding every revision's history.
Git has fully replaced this for real projects, but the "quick version
history on one file before I touch it" instinct is worth keeping —
some admins still `ci`/`co` a config file right before a risky edit,
purely as a fast undo button.

### rsync for backups

```bash
rsync -avz --delete \
    --exclude "*.o" \
    -b --backup-dir "$(date +%F)" \
    /etc /home /backups/mirror/
```

`--delete` removes files in the destination that no longer exist in
the source — matches the source exactly, which is usually what you
want for a mirror-style backup, but understand it before using it (it
can also delete backup history if misapplied). Combining `-b
--backup-dir` with `--delete` gets you the best of both: the
destination stays an exact mirror, but anything `--delete` *would*
have removed gets moved into a dated backup-dir first instead of
disappearing outright.

### tar for archival backups

```bash
tar -czvf backup-$(date +%F).tar.gz /home/user/important-data
tar -xzvf backup-2026-09-23.tar.gz -C /restore/location    # extract
```

### Point-in-time restores: the rdiff-backup pattern

Modern tools like `restic` (below) support this natively, but the
concept is worth understanding on its own: a proper backup tool lets
you restore not just "the latest backup" but "this file as it existed
N days ago" — genuinely useful when you need a file from before a bad
edit, not just before a disaster.

```bash
rdiff-backup /etc backup-host::/backups/etc      # incremental backup
rdiff-backup -r now backup-host::/backups/etc/hosts /etc/hosts       # restore latest
rdiff-backup -r 10D backup-host::/backups/etc/hosts /etc/hosts       # restore as of 10 days ago
```

`restic`'s `snapshots` + `restore --target` (below) gives you the
same point-in-time capability with better encryption and
deduplication — the underlying idea (keep enough history to restore
"as of when," not just "the latest") is the same.

### Modern backup tools worth knowing

- **restic** — encrypted, deduplicated, supports many backends
  (local, S3, SFTP), a strong modern default for real backup needs
- **Borg** — similar philosophy, another strong modern option

```bash
restic init --repo /backups/restic-repo
restic -r /backups/restic-repo backup /home/user
restic -r /backups/restic-repo snapshots
restic -r /backups/restic-repo restore latest --target /restore
```

### The step everyone skips: testing restores

**A backup you have never restored from is not a verified backup —
it's an assumption.** Corruption, permission issues, or a
misconfigured job can all produce a backup file that looks fine and
restores nothing usable. Schedule actual test restores, not just
backup jobs.

## Hands-on lab

1. Set up a daily cron job (from module 09) that backs up a test
   directory using `rsync` to a separate location.
2. Modify a file in the source, run the backup manually, and confirm
   the change propagated.
3. Set up `restic` against a local repository, back up the same test
   directory, and list the resulting snapshots.
4. **Delete the original test directory entirely**, then restore it
   from your restic backup, and verify every file is present and
   correct — this is the step that actually proves your backup
   works.
5. Pick a config file you're about to edit (e.g. one from an earlier
   module), check it into RCS with `ci`, make an edit, then use
   `rcsdiff` to see exactly what changed.

## Common pitfalls

- Setting up backup jobs and never testing a restore — this is the single most common real-world backup failure, discovered at the worst possible moment.
- Using `rsync --delete` without understanding it also removes destination files that no longer exist in the source, which can delete backup history if the source itself already lost the file.
- Backing up to the same physical disk as the original data — protects against nothing if that disk fails, satisfies neither the '2 media types' nor the '1 off-site' part of 3-2-1.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [restic documentation](https://restic.net/)
