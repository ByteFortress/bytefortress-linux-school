# Module 06 — Extra Exercises

## Exercise 1 — Signal practice

Write a tiny script that traps `SIGTERM` and prints a message before exiting, to see the difference between a process that handles termination gracefully versus one that doesn't.

## Exercise 2 — Service log diving

Pick any running service, deliberately misconfigure it slightly (a wrong config value), restart it, and use `journalctl -u` to find the error that explains why it won't start properly.

## Exercise 3 — Cron vs systemd timer

Implement the exact same scheduled task (e.g. logging the date every 5 minutes) once with cron and once with a systemd timer + service unit. Compare the setup complexity and log visibility of each.

