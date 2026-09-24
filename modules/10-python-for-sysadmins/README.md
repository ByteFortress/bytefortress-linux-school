# Module 10 — Python for SysAdmins

## Objective

Learn when and how to reach for Python instead of bash for automation tasks that need more structure, error handling, or third-party integration.

## Core concepts

### Coming from Perl (a bridge, if that's your background)

This course originally taught this exact material in Perl —
sysadmin scripting's dominant language for a long stretch before
Python took over that role. If you (or an old reference you're
holding onto) still think in Perl, here's the direct mapping onto
Python's equivalents:

| Perl | Python | Notes |
|---|---|---|
| `$a = 1;` | `a = 1` | No sigil, no semicolon required |
| `@fruit = ("apples", "oranges");` | `fruit = ["apples", "oranges"]` | List instead of array |
| `push(@fruit, "banana");` | `fruit.append("banana")` | |
| `%scores = ("Alice" => 80, "Bob" => 90);` | `scores = {"Alice": 80, "Bob": 90}` | Dict instead of hash |
| `foreach $i (keys %scores) { ... }` | `for name in scores:` | |
| `for $i (sort keys %scores) { ... }` | `for name in sorted(scores):` | |
| `$a eq $b` | `a == b` | Python doesn't distinguish string vs numeric equality operators |
| `open(FH, "<file.txt")` | `with open("file.txt") as f:` | Python's `with` auto-closes the file |
| `sub myfunc { ... }` | `def myfunc():` | |
| `$ARGV[0]` | `sys.argv[1]` | Python's `argv[0]` is the script name itself, unlike Perl |

The underlying sysadmin instincts (parse text, manipulate files, glue
commands together, handle command-line arguments) transfer directly —
only the syntax changes. If you're maintaining old Perl scripts
inherited from a previous admin, this table is a reasonable starting
point for a rewrite.

### When Python beats bash

Bash is great for quick command chaining and simple scripts. Python
tends to win once you need: real data structures (dicts, lists of
objects), robust error handling, JSON/API interaction, or a script
that other people (including future you) will need to maintain.

### Running system commands from Python

```python
import subprocess

result = subprocess.run(
    ["ls", "-la", "/var/log"],
    capture_output=True, text=True, check=True
)
print(result.stdout)
```

`subprocess.run` with `check=True` raises an exception on a non-zero
exit code — don't silently swallow command failures the way an
unguarded bash script might.

### Working with files

```python
from pathlib import Path

log_dir = Path("/var/log")
for log_file in log_dir.glob("*.log"):
    print(log_file.name, log_file.stat().st_size)
```

`pathlib` is the modern, more readable way to handle paths compared
to string concatenation.

### Parsing structured data

```python
import json

with open("/etc/myapp/config.json") as f:
    config = json.load(f)

print(config["database"]["host"])
```

Most modern APIs and many config formats are JSON — Python's built-in
`json` module makes this trivial compared to parsing it in bash.

### A minimal real example: disk usage alert

```python
import shutil
import sys

def check_disk(path="/", threshold_pct=90):
    total, used, free = shutil.disk_usage(path)
    pct_used = used / total * 100
    if pct_used > threshold_pct:
        print(f"WARNING: {path} is {pct_used:.1f}% full")
        sys.exit(1)
    print(f"OK: {path} is {pct_used:.1f}% full")
    sys.exit(0)

if __name__ == "__main__":
    check_disk()
```

This pattern (a function, a clear exit code, a `__main__` guard) is
the shape of most real sysadmin automation scripts.

## Hands-on lab

1. Write the disk usage checker above, then extend it to check
   multiple mount points instead of just `/`.
2. Write a script that reads a list of hostnames from a text file and
   pings each one, reporting which are reachable.
3. Write a script that parses a log file with `json` (create a
   sample JSON-lines log file first) and counts how many entries have
   a given field value (e.g. `"level": "ERROR"`).
4. Add proper error handling (`try`/`except`) to at least one script
   above so it fails gracefully on a missing file instead of crashing
   with an ugly traceback.

## Common pitfalls

- Using `os.system()` instead of `subprocess.run()` — `os.system` is legacy, offers worse error handling, and is more prone to shell-injection issues if any part of the command includes user input.
- Forgetting `check=True` on `subprocess.run` and not noticing a command silently failed.
- Writing a script with no `if __name__ == '__main__':` guard, which causes surprises the moment the script is imported as a module rather than run directly.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/) (free online) — practical, sysadmin-flavored Python from the ground up
