# Module 07 — Extra Exercises

## Exercise 1 — Firewall lockout recovery practice

In a disposable VM (not one you need), deliberately misconfigure ufw to lock out your own SSH access, then recover using local console access instead of SSH. Understanding this failure mode before it happens for real is valuable.

## Exercise 2 — Route tracing

Use `traceroute` to a few different real destinations and explain, in your own words, what each hop represents.

## Exercise 3 — Port inventory

Run `ss -tuln` on your VM, identify every listening port, and for each one note which service owns it and whether it should realistically be exposed beyond localhost.

