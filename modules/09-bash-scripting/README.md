# Module 09 — Bash Scripting

## Objective

Write real automation scripts: variables, conditionals, loops, and functions, moving beyond one-off commands into repeatable tooling.

## Core concepts

### Script basics

```bash
#!/usr/bin/env bash
# Always start with a shebang line, and make the script executable:
# chmod +x myscript.sh
```

### Variables

```bash
name="Jacob"
echo "Hello, $name"
readonly PI=3.14159       # constant
```

No spaces around `=` in bash — `name = "Jacob"` is a syntax error,
not an assignment.

### Conditionals

```bash
if [ -f "/etc/passwd" ]; then
    echo "File exists"
elif [ -d "/etc" ]; then
    echo "Directory exists instead"
else
    echo "Neither"
fi
```

Common test flags: `-f` (file exists), `-d` (directory exists), `-z`
(string is empty), `-eq`/`-ne` (numeric equal/not equal), `==` (string
equality, inside `[[ ]]`).

### Loops

```bash
for file in /var/log/*.log; do
    echo "Found: $file"
done

while read -r line; do
    echo "Line: $line"
done < input.txt
```

### Functions

```bash
backup_file() {
    local src="$1"
    local dest="$2"
    cp "$src" "$dest"
    echo "Backed up $src to $dest"
}

backup_file "/etc/nginx/nginx.conf" "/backups/nginx.conf.bak"
```

### Exit codes and error handling

Every command returns an exit code (0 = success, non-zero = failure).

```bash
set -e            # exit script immediately on any command failure
set -u            # error on undefined variables
set -o pipefail   # catch failures in piped commands too

command || echo "That failed"
command && echo "That succeeded"
```

### Always validate with ShellCheck

Bash has plenty of subtle footguns (word splitting, unquoted
variables). Run scripts through [ShellCheck](https://www.shellcheck.net/)
before considering them done — it catches an enormous class of real
bugs before they bite you in production.

## Hands-on lab

1. Write a script that takes a directory path as an argument and
   reports how many files, how many subdirectories, and total size.
2. Add error handling: if the argument is missing or isn't a valid
   directory, print a usage message and exit with a non-zero code.
3. Run your script through ShellCheck and fix every warning it
   raises.
4. Extend the script into a real backup tool: copy the directory to a
   timestamped destination, log the action to a file, and use `set -e`
   so any failure stops the script instead of continuing silently.

## Common pitfalls

- Unquoted variables in conditionals or loops (`if [ $var = foo ]` instead of `if [ '$var' = 'foo' ]`) — breaks badly on empty values or values with spaces. Quote your variables by default.
- Not checking a script's exit code / not using `set -e`, so a script silently continues after a step fails, potentially doing damage with bad assumptions later in the script.
- Skipping ShellCheck because 'the script works' — plenty of ShellCheck warnings describe bugs that only manifest on edge-case input, which is exactly the input that shows up in production eventually.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [ShellCheck](https://www.shellcheck.net/) (validator) and [Bash Guide](https://mywiki.wooledge.org/BashGuide) (reference)
