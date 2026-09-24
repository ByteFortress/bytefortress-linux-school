# Module 12 — DNS

## Objective

Move from 'DNS is how names become IPs' (foundations module 03) to actually running your own DNS server and understanding zone files.

## Core concepts

### Recap: the resolution chain

Client → resolver → root servers → TLD servers → authoritative
name server for the domain. Running your own DNS server means being
that last authoritative piece for your own zone (or acting as an
internal resolver for your lab network).

The DNS namespace is a tree rooted at `.` (yes, an actual trailing
dot — usually hidden by clients, but it's the real root). Below that:

- **gTLDs (generic top-level domains)** — `.com`, `.edu`, `.net`,
  `.org`, `.gov`, plus newer ones like `.info`, `.dev`
- **ccTLDs (country-code TLDs)** — `.uk`, `.jp`, `.de`, `.pr` (yes,
  Puerto Rico has its own)

Every name server on the internet is configured with the addresses of
the 13 root server clusters (`a.root-servers.net` through
`m.root-servers.net`) — that's the seed of trust the whole delegation
chain builds from. **Authoritative** answers come from a server that
actually holds the zone data (or its designated secondary);
**non-authoritative** answers come from a resolver's cache — usually
correct, but not guaranteed fresh.

**Registering a real domain** requires at least two name servers
(a master and at least one secondary) per long-standing DNS
convention — never rely on a single name server for a production
domain.

### Client-side resolver configuration

Two files control how a Linux machine itself resolves names, distinct
from running your own DNS *server*:

```bash
cat /etc/resolv.conf
# search example.lab
# nameserver 192.168.1.10
# nameserver 8.8.8.8
```

```bash
cat /etc/nsswitch.conf
# hosts: files dns
```

`nsswitch.conf`'s `hosts:` line controls the *order* of lookup
sources the C library uses (`files` = check `/etc/hosts` first, then
`dns`) — this is the actual mechanism behind foundations module 03's
"`/etc/hosts` is checked before DNS" behavior.

### BIND9 — the traditional Linux DNS server

```bash
sudo apt install bind9
```

Key config files:
- `/etc/bind/named.conf.options` — global server options
- `/etc/bind/named.conf.local` — your zone definitions
- `/etc/bind/db.<yourzone>` — the actual zone file with records

**Master/secondary (slave) setup** — a production zone should have at
least one secondary server that syncs from the master via zone
transfer:

```
zone "example.lab" IN {
    type master;
    file "/etc/bind/db.example.lab";
};

// On the secondary server instead:
zone "example.lab" IN {
    type slave;
    file "/var/cache/bind/db.example.lab.bak";
    masters { 192.168.1.10; };   // the master's IP
};
```

Zone transfers happen over TCP port 53 (ordinary queries use UDP) —
the secondary compares serial numbers with the master and pulls a
fresh copy when the master's is newer, which is exactly why
forgetting to increment the serial after an edit breaks propagation.

### A basic zone file

```
$TTL 604800
@   IN  SOA  ns1.example.lab. admin.example.lab. (
        2024010101 ; serial (increment on every change)
        604800     ; refresh
        86400      ; retry
        2419200    ; expire
        604800 )   ; negative cache TTL
@       IN  NS   ns1.example.lab.
ns1     IN  A    192.168.1.10
www     IN  A    192.168.1.20
mail    IN  A    192.168.1.30
        IN  MX 10 mail.example.lab.
```

**The serial number matters** — BIND (and secondary/slave servers)
use it to detect zone changes. Forgetting to increment it after an
edit means changes may not propagate as expected.

### Record types recap + new ones

From foundations: `A`, `MX`, `TXT`, `NS`. New here:
- **PTR** — reverse lookup (IP → name), lives in a separate reverse
  zone
- **CNAME** — an alias pointing to another name (not an IP directly)
- **SOA** — Start of Authority, the zone's own metadata (serial,
  timers)

### Testing your server

```bash
sudo systemctl restart bind9
dig @localhost www.example.lab       # query your own server directly
named-checkzone example.lab /etc/bind/db.example.lab   # validate zone syntax before reloading
```

### Simple load balancing via round-robin DNS

Adding multiple `A` records for the same name is a genuinely simple
(if crude) load-balancing technique — most resolvers rotate which
address they hand back first on repeated queries:

```
www    IN  A  10.0.0.1
www    IN  A  10.0.0.2
www    IN  A  10.0.0.3
```

```bash
dig www.example.lab +short   # run this a few times — watch the order of returned IPs rotate
```

This is exactly what large real-world sites do (Google's own
infrastructure returns several rotating addresses for `www.google.com`)
— it doesn't handle failover on its own (a dead backend's address
still gets served), but it's a genuinely useful basic technique to
know exists before reaching for a dedicated load balancer.

### whois — who actually owns a domain

```bash
whois example.com    # registrar, registration/expiry dates, name servers, sometimes contact info
```

Different from `dig`/`host` (which query DNS records) — `whois`
queries domain *registration* data instead, useful for figuring out
who to contact about a domain or when it expires.

## Hands-on lab

1. Install BIND9 and configure a simple forward zone for a fake
   domain (e.g. `example.lab`).
2. Add A records for at least three hosts and an MX record.
3. Validate your zone file with `named-checkzone` before restarting
   the service.
4. Query your own server directly with `dig @localhost` and confirm
   the records resolve correctly.
5. Set up a reverse zone and PTR records for the same hosts, and
   verify reverse lookups work with `dig -x`.
6. Add three `A` records for the same hostname pointing at three
   different fake IPs, then run `dig` against it several times and
   observe the rotation.
7. Run `whois` against a real domain you use and identify its
   registrar and expiration date.

## Common pitfalls

- Forgetting to increment the serial number after editing a zone file — changes may not be picked up as expected, especially with secondary servers.
- A syntax error in a zone file silently preventing the zone from loading — always run `named-checkzone` before restarting, and check `journalctl -u bind9` if something's not resolving.
- Confusing a CNAME's target (another name) with an A record's target (an IP) — a CNAME pointing to an IP address directly is invalid.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [BIND9 documentation](https://bind9.readthedocs.io/)
