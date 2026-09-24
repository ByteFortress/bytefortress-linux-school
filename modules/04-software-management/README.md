# Module 04 — Software Management

## Objective

Understand how packages, dependencies, and repositories work, and how to safely install software both from a repo and from source.

## Core concepts

### Package managers, high vs low level

- **Low-level** (`dpkg` on Debian/Ubuntu, `rpm` on Red Hat) — installs
  a single package file, does NOT resolve dependencies for you
- **High-level** (`apt`, `dnf`) — resolves and installs dependencies
  automatically, manages repositories

```bash
sudo apt update              # refresh package index from repos
sudo apt install nginx       # install a package + dependencies
sudo apt remove nginx        # remove package, keep config
sudo apt purge nginx         # remove package AND config
sudo apt autoremove          # clean up orphaned dependencies
apt list --installed         # see what's installed
```

**On Red Hat-family systems**, the same operations look like this —
worth knowing both since you'll encounter both families in the wild:

```bash
rpm -qa                              # list all installed packages (low-level, no dependency resolution)
rpm -q emacs                         # query whether/which version is installed
rpm -i emacs-21.4-19.el5.i386.rpm    # install a single .rpm file directly — fails loudly on unmet dependencies
rpm -e emacs                         # erase/uninstall

sudo dnf list installed              # high-level equivalent of `apt list --installed`
sudo dnf install emacs               # resolves dependencies automatically
sudo dnf remove emacs
```

The relationship is the same shape on both families: the low-level
tool (`rpm`/`dpkg`) handles a single package file and makes you solve
dependencies yourself; the high-level tool (`dnf`/`apt`) wraps it and
solves dependencies for you by consulting a repository.

### Repositories

A repository is a server hosting packages your package manager
knows about. Debian/Ubuntu repo sources live in
`/etc/apt/sources.list` and `/etc/apt/sources.list.d/`. Adding a
third-party repository means trusting that source's signing key and
maintainers — do this deliberately, not casually.

### Building from source (when there's no package)

```bash
./configure     # checks dependencies, generates a Makefile
make            # compiles
sudo make install   # installs, usually to /usr/local
```

Building from source means YOU are now responsible for tracking
updates and security patches for that software — package-managed
software gets this for free via your normal update process. Prefer
packages when they exist.

### Beyond `make install`: managing your own built-from-source software

Once you're building several things from source, dumping them all
into `/usr/local` creates the exact problem package managers exist to
solve: no record of which files belong to which piece of software,
and no clean way to remove one without hunting down its files by
hand.

A common pattern (predating and conceptually identical to tools like
GNU Stow, or how Homebrew organizes its Cellar) is to give each
package its own directory, then symlink the pieces you actually want
on your `PATH` into a shared location:

```
/opt/pkgs/emacs/            # emacs's own complete install, self-contained
/opt/pkgs/emacs/bin/emacs
/opt/pkgs/gcc/              # gcc's own complete install
/opt/bin/emacs -> /opt/pkgs/emacs/bin/emacs    # symlink into the shared bin
/opt/bin/gcc    -> /opt/pkgs/gcc/bin/gcc
```

```bash
tar xzf emacs-30.1.tar.gz
cd emacs-30.1
./configure --prefix=/opt/pkgs/emacs    # install INTO its own directory, not /usr/local
make && sudo make install
sudo ln -s /opt/pkgs/emacs/bin/emacs /opt/bin/emacs   # "install" = create the symlinks
sudo rm /opt/bin/emacs                                 # "uninstall" = remove them, source dir untouched
```

This gives you: a clean uninstall (delete the symlinks, or the whole
package directory), a clear record of what came from where, and the
option to have multiple versions of the same tool installed side by
side, switching which one is symlinked as "active." The tradeoff is
you've now built a small package manager of your own — see this
module's `exercises.md` for a hands-on version of exactly this.

**A related permissions trick worth knowing** if multiple admins
share write access to a directory like `/opt/pkgs`: setting the setgid
bit on a directory makes every new file created inside it inherit the
directory's group, instead of the creating user's default group —
useful so files a team creates stay group-writable by the whole team
automatically:

```bash
ls -ld /opt/pkgs        # drwxrwxr-x ... root installers
touch /opt/pkgs/a       # new file: group is your own default group
chmod g+s /opt/pkgs     # set setgid on the directory
ls -ld /opt/pkgs        # drwxrwsr-x ... root installers  (note the 's')
touch /opt/pkgs/b       # new file: group is now "installers", inherited from the dir
```

### Verifying what you installed

```bash
dpkg -L <package>        # list files a package installed
dpkg -S /path/to/file    # find which package owns a file
apt show <package>       # package metadata, version, description
```

## Hands-on lab

1. Update your package index and list how many packages are
   currently installed on your VM.
2. Install a package you don't already have (e.g. `tree` or `htop`),
   confirm it works, then check which files it installed with
   `dpkg -L`.
3. Purge that package and confirm its config is gone too.
4. Find a small, simple piece of open-source software with no
   package available for your distro, and build it from source
   following its own build instructions.

## Common pitfalls

- Running `apt install` without `apt update` first, and getting confusing errors about a package version that doesn't exist — always update the index first if it's been a while.
- Adding third-party repositories without verifying their signing key, which undermines the trust model package managers are built on.
- Building from source and then forgetting you did — six months later you have no idea why `apt upgrade` isn't updating that piece of software.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- Your distro's own package manager documentation (Debian/Ubuntu: `man apt`; Fedora/RHEL: `man dnf`)
