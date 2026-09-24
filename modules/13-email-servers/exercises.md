# Module 13 — Extra Exercises

## Exercise 1 — Manual SMTP conversation

Use `telnet` to manually send an email via raw SMTP commands, without using the `mail` command or any client — write down each command you sent and the server's response.

## Exercise 2 — SPF record practice

Write a correct SPF TXT record for a fictional domain with two mail servers, using the proper SPF syntax.

## Exercise 3 — Relay restriction audit

Read through your Postfix `main.cf`'s relay-related settings and explain, in your own words, what would happen if `smtpd_relay_restrictions` were set to permit everything.

