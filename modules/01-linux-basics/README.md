# Module 01 — Linux Basics

## Objective

Get oriented in the Linux/Unix world: where it came from, how distributions differ, and the core commands and editors you'll use constantly from here on.

## Core concepts

### Unix, then Linux

Unix was developed at Bell Labs in the late 1960s/70s. Linux (started
by Linus Torvalds in 1991) is a Unix-like kernel, not Unix itself, but
follows the same philosophy: small, composable tools, everything is a
file, and text streams as the universal interface between programs.

### Distributions ("distros")

A distro is the Linux kernel bundled with a package manager, default
software, and configuration conventions. Major families:

- **Debian-based** (Debian, Ubuntu, Mint) — `apt` package manager
- **Red Hat-based** (RHEL, Fedora, Rocky/Alma) — `dnf`/`yum`
- **Arch-based** (Arch, Manjaro) — `pacman`, rolling release
- **SUSE-based** (openSUSE) — `zypper`

For this curriculum, commands assume Ubuntu/Debian (`apt`) unless
noted.

### Getting help without leaving the terminal

- `man <command>` — the manual page, your primary reference
- `<command> --help` — quick usage summary
- `apropos <keyword>` — search man page descriptions by keyword
- `tldr <command>` — community-maintained simplified examples (not
  installed by default, worth adding)

### Core commands to have muscle memory for

`ls`, `cd`, `pwd`, `cp`, `mv`, `rm`, `mkdir`, `touch`, `cat`, `less`,
`grep`, `find`, `chmod`, `chown`, `ps`, `df`, `du`, `history`.

### Text editors

You will edit config files constantly as an admin. Know at least one
terminal editor well:

- **Vi/Vim** — modal editor, ubiquitous (every Linux system has some
  form of vi), steep initial learning curve, extremely fast once
  learned
- **Nano** — simple, beginner-friendly, shows keybindings on screen
- **Emacs** — powerful, extensible, different philosophy from vi

Learn Vim basics even if Nano is your daily driver — you'll encounter
systems where it's the only editor available.

## Hands-on lab

1. Identify your own distro: `cat /etc/os-release`.
2. Use `man` to read the manual for `ls`, then find the flag that
   shows hidden files and the flag that shows file sizes in
   human-readable format.
3. Open a file in Vim, enter insert mode, type a line, save, and
   quit (`:wq`) — this single workflow trips up nearly everyone the
   first time, get it down now.
4. Do the same file edit in Nano for comparison.

## Common pitfalls

- Getting stuck in Vim on the first try. Everyone does. `Esc` then `:q!` exits without saving, `:wq` saves and exits.
- Assuming all distros use the same package manager and command flags — always check `/etc/os-release` on an unfamiliar system first.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Linux Journey](https://linuxjourney.com/) — free, well-structured, covers this module and several after it
