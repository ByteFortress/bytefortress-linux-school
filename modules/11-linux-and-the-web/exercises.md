# Module 11 — Extra Exercises

## Exercise 1 — Multi-site hosting

Configure two separate virtual hosts on the same Nginx instance, each serving different content under a different `server_name`, and verify both work independently.

## Exercise 2 — Reverse proxy with headers

Set up a reverse proxy to a backend app and confirm (via the backend's own logs) that the real client IP is being forwarded correctly using `proxy_set_header X-Real-IP`.

## Exercise 3 — Config test discipline

Deliberately introduce a syntax error into an Nginx config, run `nginx -t`, read the error message, and fix it before ever attempting a reload.

