# Module 08 — Extra Exercises

## Exercise 1 — Safe hardening workflow

Practice the 'keep a working session open while testing changes' pattern explicitly: disable password auth, verify key-based login works in a new terminal, THEN close your original session.

## Exercise 2 — rsync efficiency test

Sync a directory with a few large files via `rsync`, time it, make a small change to one file, sync again, and compare the time — quantify the difference.

## Exercise 3 — SSH config file

Set up an `~/.ssh/config` entry so you can connect with just `ssh mylab` instead of typing the full user/host/port/key every time.

