# Module 10 — Extra Exercises

## Exercise 1 — Service watchdog

Write a Python script that checks whether a given process name is running (hint: `subprocess.run(['pgrep', name])`) and, if not, attempts to start it via `systemctl`.

## Exercise 2 — Config validator

Write a script that reads a JSON config file and validates that required keys are present, printing a clear error for each missing key rather than crashing on the first one.

## Exercise 3 — Bash vs Python rewrite

Take one bash script you wrote in module 09 and rewrite it in Python. Write a short note on which version you find more readable and why.

## Exercise 4 — Perl-to-Python translation

Using the mapping table in this module's README, translate the
following small Perl script into Python by hand:

```perl
#!/usr/bin/perl
%scores = ("Alice" => 80, "Bob" => 90, "Claire" => 92, "David" => 60);
for $name (sort { $scores{$b} <=> $scores{$a} } keys %scores) {
    printf("%10s: %d\n", $name, $scores{$name});
}
```

The output should print each name and score, sorted from highest
score to lowest. This exercise is a good test of whether the mapping
table's concepts actually transfer, not just the syntax of individual
lines.

