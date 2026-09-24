# Module 06 — Controlling Processes

## Objective

Understand how to observe, control, and schedule processes — essential for both day-to-day admin work and troubleshooting.

## Core concepts

### Viewing processes

```bash
ps aux              # snapshot of all running processes
top                 # live, interactive view
htop                # nicer live view (install separately)
pstree              # process hierarchy as a tree
```

Every process carries a PID, PPID (parent PID), UID/GID, and an
EUID/EGID (effective UID/GID — usually the same as UID/GID, except
for setuid processes).

**setuid processes** run with their *owner's* privileges rather than
the invoking user's — this is how an unprivileged user can run
`passwd` and have it write to root-owned `/etc/shadow` through one
controlled path, rather than needing root themselves.

```bash
find /usr/bin /bin -perm -4000 -type f 2>/dev/null | xargs ls -l
# -rwsr-xr-x 1 root root ... /usr/bin/passwd   <- the 's' means setuid
```

Every setuid binary on a system is worth knowing about — each one is
a deliberate, narrow escalation of privilege, and an unexpected one
showing up is a real red flag worth investigating.

### Process states and signals

A process can be in one of several states, visible in `ps`'s `STAT`
column:

| State | Meaning |
|---|---|
| `R` | Running/runnable — ready to execute |
| `S` | Sleeping — waiting on something (I/O, a signal), using no CPU meanwhile |
| `Z` | Zombie — the process exited, but its parent hasn't yet collected (`wait()`ed on) its exit status; shows as `<defunct>` |
| `T` | Stopped — paused by `SIGSTOP`/`SIGTSTP` |

You control processes by sending signals — there are around 30
(`kill -l` lists them all):

| Signal | Number | Meaning |
|---|---|---|
| `SIGHUP` | 1 | Traditionally "hang up," often reused to tell a daemon to reload config |
| `SIGINT` | 2 | What Ctrl-C sends |
| `SIGQUIT` | 3 | Like SIGINT but also triggers a core dump |
| `SIGKILL` | 9 | Force-terminate immediately, cannot be caught, blocked, or ignored |
| `SIGTERM` | 15 | Politely ask a process to terminate (default for `kill`) |
| `SIGTSTP` | — | What Ctrl-Z sends (pause) |
| `SIGCONT` | — | Resume a stopped process |

```bash
kill <PID>            # sends SIGTERM
kill -9 <PID>          # sends SIGKILL, use as last resort
killall <name>          # kill by process name instead of PID
kill -STOP <PID>        # pause a process
kill -CONT <PID>        # resume it
```

Always prefer `SIGTERM` first — it gives the process a chance to
clean up (close files, finish a write) before exiting. `SIGKILL`
should be your last resort when a process is genuinely unresponsive.

A process can also explicitly handle a signal instead of accepting
the default action — in a bash script, `trap 'cleanup_function' TERM INT`
lets you run cleanup code when the script receives those signals
(this module's `exercises.md` has you build exactly this).

### Niceness — process priority

Every process has a "niceness" value from -20 (highest priority) to
+19 (lowest priority) — a *higher* nice value is *less* demanding of
the CPU (i.e., "nicer" to other processes). Regular users can only
increase niceness (make a process lower priority); only root can
decrease it.

```bash
nice -n 10 ./long_running_script.sh    # start a new process with lower priority
renice 10 -p <PID>                      # change priority of an already-running process
```

### Managing services with systemd

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx      # reload config without full restart, if supported
sudo systemctl enable nginx      # start automatically on boot
sudo systemctl status nginx      # current state + recent logs
journalctl -u nginx              # full logs for this service
```

### Deeper monitoring: strace, lsof, uptime

Beyond `ps`/`top`, three tools answer specific "what is this process
actually doing" questions:

```bash
strace -p <PID>         # show every system call and signal a running process receives — great for "why is this hanging"
lsof -p <PID>            # list every file a process has open
lsof -u <username>       # list every file a given user has open anywhere
uptime                   # system uptime, user count, and load average (1/5/15 min)
```

### Handling runaway processes

A runaway process is one consuming excessive CPU, memory, or disk —
the practical workflow for dealing with one:

1. **Locate it** — `top`/`htop` for CPU/memory hogs, `lsof` if it's
   holding files open, `strace` if you need to see what it's actually
   doing moment to moment.
2. **Try niceness first** if it's legitimate work just competing
   badly for resources — `renice` it down rather than killing it.
3. **Pause it** (`kill -STOP`) if you need a moment to investigate
   without fully losing its state, then `kill -CONT` to resume or
   `kill`/`kill -9` once you've decided.
4. **Kill it** — `SIGTERM` first, `SIGKILL` only if it doesn't
   respond.

### Preventing runaway processes with PAM resource limits

Reacting to a runaway process is one approach; the other is
preventing any single user's process from being able to become one
in the first place. PAM (introduced in module 02) enforces per-user
resource limits via `/etc/security/limits.conf`:

```
# <user/group>  <hard|soft>  <item>  <value>
blackcat            hard         cpu     2          # max 2 minutes of CPU time
blackcat            hard         nproc   50          # max 50 simultaneous processes
blackcat            hard         nofile  1024        # max 1024 open files
```

A `soft` limit is the default a user's session starts with (they can
raise it up to the `hard` limit themselves, e.g. with `ulimit`); a
`hard` limit is the absolute ceiling only root can raise. **Changes
require the affected user to log out and back in** — an existing
session doesn't pick up new limits.

```bash
ulimit -a          # show all current limits for your shell session
ulimit -u           # show just the process limit
```

This is genuinely the difference between "a bug in one user's script
degrades that user's own work" and "a bug in one user's script takes
the whole system down" — worth setting on any real multi-user system.

### Shared libraries and function interposition

Most programs are dynamically linked — they load shared libraries
(`.so` files) at runtime rather than having that code baked into the
executable. You can inspect this:

```bash
file /bin/ls                 # confirms dynamically vs statically linked
ldd /bin/ls                   # lists every shared library it depends on
```

`LD_PRELOAD` lets you load your own shared library *before* the
normal ones, so your versions of specific functions get called
instead of the real ones — a technique called function interposition.
This is genuinely useful for testing and debugging: mocking a
function that would otherwise make your tests slow, flaky, or
non-deterministic (a network call, the system clock, a random number
generator) without touching the program's source code at all.

**Example: mocking `time()` for deterministic testing.** Say you're
testing a program whose behavior depends on the current time, and you
want it to always believe it's a specific moment, without editing the
program itself:

```c
/* fake_time.c */
#include <time.h>

time_t time(time_t *t) {
    time_t fixed = 1700000000;   /* whatever fixed timestamp your test needs */
    if (t) *t = fixed;
    return fixed;
}
```

```bash
gcc -shared -fPIC -o fake_time.so fake_time.c
LD_PRELOAD=./fake_time.so ./your_program
```

`your_program` now sees a frozen clock, with zero changes to its own
code — the exact same mechanism a test framework's "mock" or
"stub" tooling uses under the hood, just done manually so you can see
how it actually works.

### Scheduling: cron vs systemd timers

**Cron** — the traditional scheduler:

```bash
crontab -e
# minute hour day month weekday command
0 2 * * * /home/user/backup.sh              # runs daily at 2:00 AM
30 23 * * fri /home/user/weekly-backup.sh    # runs 11:30 PM every Friday
0 */1 * * * /usr/bin/rdate -s time.nist.gov  # runs every hour on the hour
```

**systemd timers** — the modern alternative, integrates with the
journal for logging and can express dependencies more richly. Worth
knowing exists even if cron remains simpler for basic scheduling.

## Hands-on lab

1. Start a long-running process deliberately (e.g. `sleep 300 &`),
   find its PID with `ps aux | grep sleep`, and terminate it with a
   plain `kill`.
2. Start another one, and this time practice `kill -STOP` then
   `kill -CONT` to pause and resume it.
3. Pick a service on your system (e.g. `ssh` or `cron`), check its
   status, view its recent logs with `journalctl -u`, then restart it
   and confirm it's running again.
4. Write a cron job that appends the current date to a log file every
   minute, verify it's working, then remove it.
5. Write a small program (or use `yes > /dev/null &` a few times)
   that consumes significant CPU or spawns many processes. Set a
   `nproc` and a `cpu` hard limit for your user in
   `/etc/security/limits.conf`, log out and back in, then confirm the
   limit actually stops your test program from exceeding it.
6. Build the `fake_time.so` example from this module's README,
   confirm `LD_PRELOAD`-ing it into a small test program that calls
   `time()` actually returns your fixed timestamp instead of the real
   time.

## Common pitfalls

- Reaching for `kill -9` by default. It skips cleanup — a database or file-writing process killed this way can leave corrupted state. Try plain `kill` (`SIGTERM`) first.
- Forgetting `systemctl enable` after `systemctl start` — the service runs now but won't survive a reboot.
- Cron jobs that work when run manually but fail under cron, usually because cron runs with a minimal environment (different `PATH`, no shell profile loaded) — always use full paths in cron commands.
- Editing `/etc/security/limits.conf` and then testing in the same shell session you're already logged into — the new limits only apply to sessions started *after* the change. Log out and back in before concluding a limit "doesn't work."

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Linux Journey — Processes section](https://linuxjourney.com/)
