# Module 07 — Basic Networking

## Objective

Go beyond foundations' conceptual networking into actually configuring interfaces, routing, and firewalls on a live Linux system.

## Core concepts

> "Be liberal in what you accept, and conservative in what you send."
> — Jon Postel, one of the internet's founding engineers. This
> principle (Postel's Law) still guides how most internet protocols
> are actually implemented today, TCP/IP included.

### Local name resolution: /etc/hosts

Before any DNS server is even consulted, Linux checks `/etc/hosts`
for a matching entry:

```
127.0.0.1    localhost
192.168.1.10 lab-server lab-server.local
```

This is why you can give a machine a memorable local name without
running any DNS server at all — useful for a small lab network like
the one you'll build in module 09 of `bytefortress-foundations-school`,
and worth checking first whenever a hostname resolves to something
unexpected (an old, unrelated `/etc/hosts` entry is a classic gotcha).

### IP addressing: classes, CIDR, and private ranges

IPv4 addresses were originally divided into fixed "classes" based on
the first octet, before CIDR replaced this rigid scheme:

| Class | Range | Format |
|---|---|---|
| A | 1-126 | Network.Host.Host.Host |
| B | 128-191 | Network.Network.Host.Host |
| C | 192-223 | Network.Network.Network.Host |

This ran out of room fast — CIDR (Classless Inter-Domain Routing)
replaced it with flexible prefix notation, which is what you actually
use today: `192.168.1.0/24` means the first 24 bits are the network
portion. `/26` and other odd prefixes are genuinely fiddly to compute
by hand — a CIDR calculator (many free ones online) is a completely
normal tool to reach for, not a sign you don't understand
networking.

**Private IP ranges** (RFC 1918) — addresses that are never routed on
the public internet, reused by every home and office network:

| Range | CIDR |
|---|---|
| `10.0.0.0` - `10.255.255.255` | `10.0.0.0/8` |
| `172.16.0.0` - `172.31.255.255` | `172.16.0.0/12` |
| `192.168.0.0` - `192.168.255.255` | `192.168.0.0/16` |

NAT (Network Address Translation) is what lets many devices on one of
these private ranges share a single public IP — your home router does
this for every device on your Wi-Fi.

### Viewing and configuring interfaces

```bash
ip a                    # show all interfaces and their IPs (modern)
ip link set eth0 up      # bring an interface up
ip addr add 192.168.1.50/24 dev eth0   # assign an IP manually
```

Modern distros manage persistent network config via Netplan
(Ubuntu) or NetworkManager — manual `ip` commands are for
troubleshooting/temporary changes, not persistent configuration.

**You'll still see the older `ifconfig` command in a lot of existing
documentation and scripts** — it's deprecated in favor of `ip` but
worth recognizing:

```bash
ifconfig eth0 up
ifconfig eth0 138.23.169.9 netmask 255.255.255.0    # old-style manual IP assignment
ifconfig -a                                          # show all interfaces, including down ones
```

### Routing

```bash
ip route                # show the routing table
ip route add default via 192.168.1.1   # set a default gateway
```

The routing table decides, for any destination IP, which interface
and next hop to send traffic through. A missing or wrong default
route is one of the most common "no internet access" causes.

**Routing vs forwarding** — routing is the process of *building* the
routing table (deciding what paths exist); forwarding is actually
*using* that table to move a packet from one interface to another.
The older `netstat -rn` (superseded by `ip route`) makes this table
concrete:

```
Destination     Gateway        Genmask          Flags  Iface
192.168.1.0     0.0.0.0        255.255.255.0    U      eth0
0.0.0.0         192.168.1.1    0.0.0.0          UG     eth0
```

The last line — destination `0.0.0.0` — is the default route: "if
nothing more specific matches, send it to this gateway."

### Firewalls: iptables, nftables, and ufw

- **iptables** — the traditional Linux firewall tool, rule-based,
  still widely used and documented
- **nftables** — the modern replacement, cleaner syntax, gradually
  replacing iptables under the hood on many distros
- **ufw (Uncomplicated Firewall)** — a friendly frontend for iptables/
  nftables, the easiest starting point on Ubuntu

```bash
sudo ufw allow 22/tcp        # allow SSH
sudo ufw allow from 192.168.1.100 to any port 22   # allow SSH from one IP only
sudo ufw deny 23             # deny telnet
sudo ufw enable
sudo ufw status verbose
```

### Basic troubleshooting toolkit

```bash
ping <host>          # basic reachability
traceroute <host>    # path to a destination
ss -tuln             # show listening ports (modern replacement for netstat)
dig / nslookup       # DNS resolution testing (from foundations module 03)
```

**`tcpdump`** captures raw packets on an interface — the command-line
sibling of Wireshark (foundations module 03), useful when you're on a
server with no GUI:

```bash
sudo tcpdump -i eth0                    # capture everything on eth0
sudo tcpdump -i eth0 port 80             # capture only port 80 traffic
sudo tcpdump -i eth0 host 192.168.1.50   # capture only traffic to/from one host
```

### Lab project: build a two-VM router

A genuinely great way to make "routing vs forwarding" concrete is to
build a minimal router yourself out of a second VM, rather than only
reading about routing tables in the abstract.

**Topology:** VM 1 ("host") on one internal network, VM 2 ("router")
bridging that internal network and your normal lab network, VM 1's
only path to anything is through VM 2.

1. Give the router VM two interfaces: one on the same internal
   network as the host VM, one on your regular lab network.
2. On the host VM, set its default gateway to the router VM's
   internal-facing IP.
3. On the router VM, enable IP forwarding:
   ```bash
   sudo sysctl -w net.ipv4.ip_forward=1
   # persist it: add net.ipv4.ip_forward=1 to /etc/sysctl.conf
   ```
4. From the host VM, ping something on the lab network beyond the
   router — confirm it works only once forwarding is enabled (test
   with it disabled first, to see the failure).
5. Use `tcpdump` on the router VM's two interfaces to actually watch
   a packet arrive on one interface and leave on the other — this is
   forwarding, made visible.

## Hands-on lab

1. View your current interfaces, IP addresses, and routing table.
2. Temporarily assign an additional IP address to your interface with
   `ip addr add`, confirm it with `ip a`, then remove it.
3. Set up `ufw` to allow SSH only from your lab network's subnet, deny
   everything else, and verify with `ufw status verbose`.
4. Use `ss -tuln` to see what's currently listening on your system,
   and identify each service by its port.

## Common pitfalls

- Applying a restrictive firewall rule over an active SSH session without a rule allowing your own current connection — you can lock yourself out immediately. Always allow your own management access before tightening anything else.
- Confusing temporary `ip` command changes (lost on reboot) with persistent Netplan/NetworkManager configuration — know which one you're actually changing.
- Forgetting `ufw enable` — rules configured but the firewall not enabled do nothing.
- Adding a second network interface and finding `/etc/resolv.conf` mysteriously changed — a newly configured interface commonly pulls its own DNS settings via DHCP and overwrites what you had. Check `resolv.conf` after any interface change to a multi-VM lab setup like the router lab below.
- Starting both VMs in the router lab at the same time — bring the router VM up first, confirm its forwarding and interfaces are correct, then start the host VM. Starting both simultaneously makes it much harder to tell which VM a networking problem is actually coming from.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Arch Wiki networking pages](https://wiki.archlinux.org/title/Network_configuration) — distro-agnostic depth on every topic above
