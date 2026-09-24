# Module 13 — Email Servers

## Objective

Understand SMTP fundamentals and stand up a basic mail server, including the authentication mechanisms that keep it from becoming a spam relay.

## Core concepts

### SMTP basics

SMTP (Simple Mail Transfer Protocol) is how mail servers exchange
messages with each other. A full mail setup breaks down into five
distinct roles — worth knowing all five even though one piece of
software (like Postfix) often handles more than one role at once:

| Role | Job | Example software |
|---|---|---|
| **MUA** (Mail User Agent) | Lets a person read/compose mail | Thunderbird, Outlook, webmail |
| **MSA** (Mail Submission Agent) | Accepts outgoing mail from an MUA (port 587) | Postfix |
| **MTA** (Mail Transfer Agent) | Routes mail *between* servers (port 25), speaks SMTP | Postfix, Sendmail |
| **LDA** (Local Delivery Agent) | Places incoming mail into a local mailbox | Dovecot, procmail |
| **AA** (Access Agent) | Lets an MUA retrieve mail from the mailbox | Dovecot (IMAP/POP3) |

Postfix alone commonly plays MSA, MTA, and (with Dovecot for the
rest) hands off local delivery and retrieval — the roles are
conceptually separate even when one project's docs blur them
together.

### Installing Postfix

```bash
sudo apt install postfix
```

Key config: `/etc/postfix/main.cf`. Choose "Internet Site" during
setup for a server that sends/receives mail directly.

### A minimal main.cf for a lab

```
myhostname = mail.example.lab
mydomain = example.lab
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```

### Why open relays are dangerous

An open relay accepts and forwards mail from anyone to anyone,
without authentication. Spammers actively scan for these and abuse
them within hours of one appearing online — this is why default
Postfix configs are relay-restricted, and why you should never
loosen `smtpd_relay_restrictions` without understanding exactly what
you're opening up.

### Authentication: SPF, DKIM, DMARC (conceptual)

These exist to prove a message claiming to be from your domain
actually came from an authorized source:

- **SPF** — a DNS TXT record listing which servers are allowed to
  send mail for your domain
- **DKIM** — cryptographically signs outgoing mail so receivers can
  verify it wasn't altered and really came from you
- **DMARC** — tells receiving servers what to do with mail that fails
  SPF/DKIM (quarantine, reject, or just report)

Setting these up fully (DKIM key generation and signing, DMARC
policy) is a deeper topic than this module covers in depth — treat
this as the conceptual map, and the external resource below for
hands-on depth.

### Aliases and forwarding

**`/etc/aliases`** lets one address redirect to another (or several) —
commonly used so system mail (from cron, from root) actually reaches
a real person:

```bash
cat /etc/aliases
# root: blackcat, blackcat@example.lab

sudo newaliases    # rebuild the aliases database after editing — easy to forget
```

**`~/.forward`**, in a user's home directory, forwards that specific
user's mail elsewhere without needing root access to `/etc/aliases`:

```bash
cat ~/.forward
# jacob@example.lab
```

### Filtering and automating mail delivery: procmail

`procmail` (the LDA role from the table above) can filter, sort, or
even trigger actions based on incoming mail content, via
`~/.procmailrc` rules. A basic spam-sorting rule:

```
:0
* ^Subject:.*\[SPAM\]
INBOX.spam
```

A more elaborate example — forward any email from a specific sender
to an external notification address (e.g. an SMS gateway), while
still keeping a normal copy:

```
:0 c
* ^From:.*myboss@example\.com
| (formail -c -XFrom: -XSubject:; echo "To: $MY_PHONE_GATEWAY") | $SENDMAIL -oi -t
```

The `:0 c` means "process this rule with a *copy*, keep processing
further rules too" — useful for "notify me AND file it normally"
patterns.

`procmail` still works today but is effectively unmaintained;
**Sieve** (via Dovecot's `pigeonhole` plugin) is the actively
maintained modern equivalent, with clearer syntax and IMAP-server-side
execution instead of relying on the MTA to invoke a separate program.
Procmail's rule *logic* above (match a header, file into a folder,
or forward) maps directly onto Sieve's `if header :contains` / `fileinto`
/ `redirect` — worth knowing procmail's shape since so much existing
documentation and legacy config still uses it.

### Testing

```bash
sudo systemctl status postfix
telnet localhost 25          # manually speak SMTP to test locally
mail -s "test subject" user@example.lab   # send a quick test message
```

## Hands-on lab

1. Install Postfix configured for your lab domain.
2. Send a test email between two local users on the same server using
   the `mail` command, and confirm delivery by checking the
   recipient's mailbox (`/var/mail/<user>` or via Dovecot/IMAP if you
   set that up too).
3. Use `telnet localhost 25` to manually issue SMTP commands
   (`HELO`, `MAIL FROM`, `RCPT TO`, `DATA`) and send a message by
   hand — this demystifies what your mail client is actually doing.
4. Add an SPF TXT record to your lab DNS zone (from module 12) for
   your mail domain, even though it won't be validated by anyone
   outside your lab — the point is practicing the syntax.
5. Add an `/etc/aliases` entry redirecting `root`'s mail to your own
   test user, run `newaliases`, then trigger some system mail (a cron
   job failure is a reliable source) and confirm it actually arrives
   at the redirected address.
6. Install `procmail`, write a `~/.procmailrc` rule that files any
   message with a specific subject keyword into a separate mailbox
   folder, and test it by sending yourself a matching message.

## Common pitfalls

- Loosening relay restrictions to 'just get something working' and accidentally creating an open relay — always understand exactly what a config change permits before applying it, especially anything touching `smtpd_relay_restrictions`.
- Forgetting that mail delivered locally with basic Postfix+no MDA setup lands in `/var/mail/<user>`, not somewhere more obvious, and concluding delivery failed when it actually succeeded.
- Not opening the firewall for port 25 (or the ports Dovecot needs) if testing from a separate client machine, and mistaking a network issue for a mail configuration issue.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Postfix documentation](http://www.postfix.org/documentation.html)
