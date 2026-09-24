# Module 17 — Extra Exercises

## Exercise 1 — Full restore drill

Delete a test directory entirely, restore it from backup, and diff the restored contents against a known-good copy to confirm byte-for-byte correctness.

## Exercise 2 — Incremental vs full timing

Compare how long a full rsync backup takes versus a second run where only a few files changed, to see the practical benefit of incremental-style behavior.

## Exercise 3 — Retention policy

Using restic's `forget` command with retention flags (e.g. keep last 7 daily, 4 weekly), set up and test a basic retention policy so backups don't grow unbounded.

