# Module 16 — Extra Exercises

## Exercise 1 — Lynis deep dive

Run Lynis, pick the five highest-priority suggestions in its report, and implement at least three, documenting what each one actually does and why it matters.

## Exercise 2 — Audit rule practice

Set up audit rules for at least two sensitive files/directories beyond the core lab's example, trigger each, and confirm proper logging.

## Exercise 3 — End-to-end hardening review

Go back through modules 02, 07, and 08 and write a checklist confirming every hardening step recommended there is actually applied on your lab VM.

## Exercise 4 — Lock down a real multi-service server, one port at a time

This is the exercise that ties nearly the whole course together, and
it's worth doing on a VM that's actually running the services from
modules 11-13 (web, DNS, mail) rather than a blank machine —
hardening something with nothing running on it isn't a real test.

1. Confirm your DNS (module 12), web (module 11), and mail (module
   13) servers are all currently reachable from a second VM, with the
   firewall either off or wide open.
2. Apply this module's default-deny `iptables` ruleset (flush,
   default DROP on `INPUT`, allow loopback and established/related
   connections, allow your own SSH access) — **do this over local
   console access, not SSH, the first time**, in case you get the SSH
   allow rule wrong.
3. From the second VM, confirm every service now fails: DNS lookups
   time out, the web server is unreachable, mail delivery fails.
4. Add rules back one service at a time, testing after each one:
   - UDP port 53 for basic DNS queries
   - TCP port 53 for zone transfers (module 12's master/secondary
     sync depends on this)
   - TCP port 80 (and 443 if you set up TLS) for the web server
   - TCP port 25 for mail
5. After each rule you add, re-test from the second VM and note in
   `NOTES.md` exactly which service started working and why that
   port/protocol combination was the right one to open.
6. End state: every legitimate service works, and `iptables -L -v`
   shows a short, deliberate, explainable list of allowed traffic —
   not a default-allow policy with some things blocked.

This exercise is a genuinely honest test of whether you understand
what each of your services actually needs on the network, rather than
just knowing the syntax for opening a firewall rule.

