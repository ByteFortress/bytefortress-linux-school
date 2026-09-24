# Module 12 — Extra Exercises

## Exercise 1 — Full lab zone

Build a complete zone for a fake domain with at least 5 hosts, a mail server (MX), a CNAME alias, and matching reverse (PTR) records.

## Exercise 2 — Serial number discipline

Make three separate edits to your zone file over time, each time correctly incrementing the serial and confirming with `dig` that the change took effect.

## Exercise 3 — Break and fix

Deliberately introduce a zone file syntax error, observe what `named-checkzone` reports and what happens if you restart BIND9 anyway (it should refuse to load the broken zone), then fix it.

