# Module 11 — Linux and the Web

## Objective

Stand up and configure a real web server, including virtual hosts, TLS, a database-backed page, and a caching proxy, moving from 'concept' (foundations module 04) to 'I run this.'

## Core concepts

### Apache vs Nginx, briefly

- **Apache** — mature, highly configurable via `.htaccess` per-directory
  overrides, module-based
- **Nginx** — event-driven architecture, often faster for static
  content and as a reverse proxy, configuration is centralized rather
  than per-directory

Either is a reasonable choice; this module uses Nginx for examples,
but here's Apache's equivalent setup since you'll encounter it just
as often in the wild:

```bash
sudo apt install apache2          # Debian/Ubuntu (package is httpd on Red Hat-family)
sudo systemctl enable --now apache2
```

Apache's files, by convention, end up more spread out than Nginx's:

| Path | Purpose |
|---|---|
| `/etc/apache2/apache2.conf` (or `/etc/httpd/conf/httpd.conf`) | Main config |
| `/etc/apache2/sites-available/`, `sites-enabled/` | Per-site configs, same enable-via-symlink pattern as Nginx |
| `/var/www/html/` | Default document root |
| `/var/log/apache2/access.log`, `error.log` | Logs |

```bash
sudo apachectl configtest    # Apache's equivalent of `nginx -t`
sudo systemctl reload apache2
```

### Anatomy of a URL

Worth being able to name every part of a URL precisely, since you'll
be troubleshooting based on exactly these pieces constantly:

```
https://www.example.com:443/blog/index.html
└─┬──┘   └──────┬──────┘└┬┘└──────┬───────┘
 scheme       hostname   port    path
```

Beyond `http`/`https`, you'll see other URL schemes: `ftp://` (file
transfer), `mailto:` (opens an email client), `file://` (a local
file path). Each implies a completely different protocol handling the
request, not just a different server.

### Installing and basic config

```bash
sudo apt install nginx
sudo systemctl enable --now nginx
```

Config lives in `/etc/nginx/`, with site-specific configs typically
in `/etc/nginx/sites-available/` and enabled via a symlink in
`/etc/nginx/sites-enabled/`.

### A basic virtual host (server block)

```nginx
server {
    listen 80;
    server_name example.lab;
    root /var/www/example;
    index index.html;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/
sudo nginx -t          # test config syntax before reloading
sudo systemctl reload nginx
```

Always `nginx -t` before reloading — a syntax error in a reload can
take down a server that was working fine before your edit.

### TLS with Let's Encrypt

For a real public domain, [Certbot](https://certbot.eff.org/) automates
getting and renewing free TLS certificates from Let's Encrypt:

```bash
sudo certbot --nginx -d example.com
```

For a lab environment with no public domain, you'd instead generate a
self-signed certificate for practice — browsers will warn about it,
which is expected and fine for a lab.

### A real misconfiguration worth seeing once: FollowSymLinks

Apache's `FollowSymLinks` option (enabled by default in many configs)
means the web server will follow a symbolic link inside the document
root out to wherever it points — including outside the document root
entirely. This is worth demonstrating on your own lab server once, so
the risk isn't just theoretical:

```bash
# Inside your document root:
sudo ln -s /etc/passwd /var/www/html/leaky
```

```bash
curl http://localhost/leaky    # if FollowSymLinks is on, this serves /etc/passwd's contents
```

Now disable it (in your Apache `<Directory>` block, or by removing
`FollowSymLinks` from the `Options` line) and reload:

```bash
sudo apachectl configtest && sudo systemctl reload apache2
curl http://localhost/leaky    # should now fail
```

This is exactly the shape of a huge number of real web server
misconfigurations: a convenience feature (symlinks are genuinely
useful for legitimate reasons) left enabled without considering what
it exposes. Delete the test symlink when you're done.

### The full LAMP stack: adding a database

Nginx/Apache serving static pages is only part of the picture — LAMP
(**L**inux, **A**pache, **M**ySQL, **P**HP) is the classic stack for
anything database-backed.

```bash
sudo apt install mariadb-server php php-mysql libapache2-mod-php
sudo systemctl enable --now mariadb
sudo mysql_secure_installation    # set a root password, remove test defaults — don't skip this on anything beyond a throwaway lab
```

Create a small database to confirm everything's connected end to end:

```sql
-- via: sudo mysql
CREATE DATABASE labdb;
USE labdb;
CREATE TABLE grades (name VARCHAR(20), grade INT);
INSERT INTO grades VALUES ('Alice', 80), ('Bob', 90), ('Claire', 92);
```

```php
<?php
// /var/www/html/grades.php
$conn = new mysqli("localhost", "root", "", "labdb");
if ($conn->connect_error) die("Connection failed: " . $conn->connect_error);

$result = $conn->query("SELECT * FROM grades ORDER BY grade DESC");
while ($row = $result->fetch_assoc()) {
    printf("%s: %d\n", $row["name"], $row["grade"]);
}
$conn->close();
?>
```

Request `grades.php` from a browser or `curl` and confirm the data
comes back — this end-to-end chain (web server → PHP → database →
back through PHP → back through the web server) is worth actually
seeing work once, since it's the backbone of an enormous fraction of
real-world web applications.

### Reverse proxying

Nginx is very commonly used as a reverse proxy in front of an
application server (e.g. a Python/Node app running on
`localhost:8000`):

```nginx
location / {
    proxy_pass http://localhost:8000;
    proxy_set_header Host $host;
}
```

### Caching proxies: Squid

Nginx as a reverse proxy sits in *front of your own* server. A
**caching proxy** like Squid solves a different problem: reducing
bandwidth and latency by caching *other* sites' content on behalf of
your own network's clients (a "forward" proxy), or accelerating your
own site's static content (a "reverse"/accelerator use case,
conceptually similar to a CDN).

```bash
sudo apt install squid
sudo squid -z              # create swap/cache directories — run once, ever
sudo systemctl enable --now squid
```

Squid listens on port 3128 by default. You can watch it work exactly
the way you'd watch a web server, since it speaks HTTP too:

```bash
telnet localhost 3128
GET http://example.com/ HTTP/1.0
Host: example.com

```

The response includes an `X-Cache` header (`HIT` or `MISS`) showing
whether Squid served the request from its own cache or had to fetch
it fresh — genuinely satisfying to watch flip to `HIT` on a repeat
request.

**A more convincing proof than reading a header once**: measure it.

```bash
http_proxy=localhost:3128 time wget http://example.com/somefile.tar.gz -O /dev/null
# run the exact same command again immediately
http_proxy=localhost:3128 time wget http://example.com/somefile.tar.gz -O /dev/null
```

The second download should complete dramatically faster — that
speed difference *is* the cache working, not just a header claiming
it did. Cross-check it against Squid's own log:

```bash
grep somefile.tar.gz /var/log/squid/access.log
# look for TCP_MISS on the first request, TCP_HIT on the second
```

**Transparent caching** (clients need zero configuration) is done by
redirecting outbound port 80 traffic into Squid via `iptables`/`ufw`
rules rather than requiring every client to set a proxy setting —
tying directly back to this module's own firewall content and module
07's networking fundamentals.

```bash
sudo squid -k reconfigure   # reload config without dropping the cache
sudo squid -k rotate        # rotate Squid's logs
```

## Hands-on lab

1. Install Nginx and confirm the default page loads.
2. Create a virtual host serving a simple custom HTML page under a
   different `server_name`.
3. Test your config with `nginx -t` before every reload — make this a
   habit now.
4. Generate a self-signed TLS certificate and configure your virtual
   host to serve over HTTPS, accepting the browser warning as
   expected for a lab cert.
5. Set up a second, trivial "app" (even a simple Python
   `http.server` on port 8000) and configure Nginx to reverse-proxy
   to it.
6. Install Squid, run `squid -z` once, then use `telnet` to manually
   request the same URL twice through it — compare the `X-Cache`
   header between the first request (`MISS`) and the second (`HIT`),
   then confirm it with a timed `wget` download and the access log.
7. On Apache (install it alongside or instead of Nginx for this
   step), demonstrate the `FollowSymLinks` risk: symlink `/etc/passwd`
   into your document root, confirm it's servable, then disable
   `FollowSymLinks` and confirm it's blocked. Remove the test symlink
   afterward.
8. Install MariaDB and PHP, create a small database and table, and
   write a PHP page that queries it and displays the results —
   confirm the full request → PHP → database → response chain works.

## Common pitfalls

- Reloading Nginx after an edit without running `nginx -t` first — a bad config can take down every site the server hosts, not just the one you were editing.
- Forgetting to actually enable a new site (the symlink into `sites-enabled/`) and wondering why the config seems to have no effect.
- Confusing `reload` (graceful, applies new config without dropping connections) with `restart` (drops everything briefly) — prefer `reload` for config changes.
- Leaving `FollowSymLinks` enabled on a production Apache document root without thinking about it — it's a default in many configs, not a deliberate choice most admins make consciously.
- Skipping `mysql_secure_installation` because "it's just a lab" — the habit of skipping it is the actual risk, since the same shortcut on a real deployment leaves default credentials and test databases in place.

## Extra practice

Additional exercises live in `exercises.md` — do Exercise 1 here:
[`exercises.md`](exercises.md).

## Further reading

- [Nginx docs](https://nginx.org/en/docs/) and [Let's Encrypt](https://letsencrypt.org/) for real-domain TLS
