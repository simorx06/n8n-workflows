# Caddy Block Snippet — Paste into `/etc/caddy/Caddyfile`

This snippet reverse-proxies to n8n running on `127.0.0.1:5678`. Replace `n8n.example.com` with your actual domain.

```caddy
n8n.example.com {
    reverse_proxy 127.0.0.1:5678 {
        header_up Host {host}
        header_up X-Real-IP {remote_host}
        header_up X-Forwarded-For {remote_host}
        header_up X-Forwarded-Proto {scheme}
    }
}
```

> **Note:** This assumes Caddy is installed **on the host** (not in a container). Paste this block into your existing `/etc/caddy/Caddyfile` and run `caddy reload` or `systemctl reload caddy`.