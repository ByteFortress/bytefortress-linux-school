# Module 04 — Extra Exercises

## Exercise 1 — Dependency detective

Install a package with several dependencies (e.g. a database server), then use `apt show` and `apt-cache depends` to map out what it actually pulled in.

## Exercise 2 — Clean removal practice

Install and then fully purge three different packages, confirming each time that config files are actually gone (`dpkg -L` before purging to know what to check).

## Exercise 3 — Source build with tracking

Build one small tool from source, then write yourself a note (in `NOTES.md`) explaining how you'd remember to check for updates to it manually.

## Exercise 4 — Build Your Own Package Manager (capstone-style project)

This is a real project from the original course this repo is built
from — a genuinely great way to internalize the directory-per-package
+ symlink pattern from this module's README by implementing it
yourself, in bash (this pairs well with module 09's bash scripting
content — do this exercise after or alongside that module).

**Goal:** implement a small package management system as four bash
scripts, all sourcing shared logic from one master script.

- **`pkgtools.sh`** — the master script, sourced by the others.
  `./pkgtools.sh init` should create an `installers` group and set up
  the shared directory structure (`$IMPORT_HOME/bin`,
  `$IMPORT_HOME/etc`, `$IMPORT_HOME/pkgs`, etc. — default
  `$IMPORT_HOME` to `/opt/import` if unset).
- **`pkginstall`** — `./pkginstall emacs` should prompt for a version
  and a source URL, download/build the package into
  `$IMPORT_HOME/pkgs/emacs`, then create symlinks for every relevant
  file into the shared `$IMPORT_HOME/bin`, `/etc`, `/lib`, `/man`,
  etc. (the man directory is the trickiest part — get the other
  symlinks working first).
- **`pkgremove`** — the reverse: remove a package's symlinks and
  optionally its source directory.
- **`pkginfo`** — track installed packages in a simple flat file
  (one line per package: name, version, installing user, source URL,
  install date, and a `*` flag marking a removed package rather than
  deleting its history entry). Support at least `-a` (all packages,
  including removed), `-i` (currently installed only), and `-u`
  flags.

**Constraints that make this a real exercise, not just a script:**

- Every script should respect `$IMPORT_HOME` if set, defaulting
  sensibly if not.
- Every script should print a usage message when run with `-h` or no
  arguments.
- `pkginstall`/`pkgremove` should update the same tracking file
  `pkginfo` reads from — think about the data format up front so all
  three scripts agree on it.

Run every script through ShellCheck (module 09) before considering it
done. This is a substantial exercise — expect it to take real time,
and that's the point.

