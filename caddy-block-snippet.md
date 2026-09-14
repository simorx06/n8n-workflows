# Caddy Block Snippet — Paste into your Caddyfile

This snippet reverse-proxies to the n8n container. Replace `n8n.example.com` with your actual domain.

```caddy
n8n.example.com {
    reverse_proxy n8n:5678 {
        header_up Host {host}
        header_up X-Real-IP {remote_host}
        header_up X-Forwarded-For {remote_host}
        header_up X-Forwarded-Proto {scheme}
    }
}
```

> **Note:** If you already have a Caddyfile with other sites, just paste this block inside it. The `{$CADDY_DOMAIN}` placeholder in the repo's `Caddyfile` will be substituted at runtime from the `CADDY_DOMAIN` environment variable.