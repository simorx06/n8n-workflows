# n8n Workflows

Self-hosted [n8n](https://n8n.io) instance behind a **host-level Caddy** reverse proxy with automatic HTTPS.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/) & [Docker Compose](https://docs.docker.com/compose/install/) (v2.x+)
- [Caddy](https://caddyserver.com/docs/install) installed **on the host**
- A domain name pointing to your server

## Quick Start

### 1. Clone & enter the repo

```bash
git clone <your-repo-url> n8n-workflows
cd n8n-workflows
```

### 2. Configure environment

```bash
cp .env.example .env
```

Edit [`.env`](.env) and fill in every variable:

| Variable | Description | Example |
|---|---|---|
| `N8N_HOST` | Public domain n8n runs on | `n8n.example.com` |
| `N8N_PORT` | Internal port (keep `5678`) | `5678` |
| `N8N_PROTOCOL` | Protocol for webhook URLs | `https` |
| `N8N_ENCRYPTION_KEY` | **Secret** – encrypts credentials | `your-32-char-random-key` |
| `N8N_PROXY_HOPS` | Reverse-proxy hops (set to `1`) | `1` |
| `WEBHOOK_URL` | Override webhook URL | `https://n8n.example.com/` |
| `EXECUTIONS_DATA_PRUNE` | Enable auto-pruning | `true` |
| `EXECUTIONS_DATA_MAX_AGE` | Max age in hours | `168` |
| `EXECUTIONS_DATA_PRUNE_TIMEOUT` | Prune timeout in seconds | `3600` |
| `N8N_DIAGNOSTICS_ENABLED` | Telemetry / diagnostics | `false` |
| `N8N_PERSONALIZATION_ENABLED` | Onboarding personalization | `false` |
| `N8N_VERSION_NOTIFICATIONS_ENABLED` | Version update notifications | `true` |
| `GENERIC_TIMEZONE` | System timezone | `Europe/London` |

> **Important:** Generate a strong `N8N_ENCRYPTION_KEY`:
> ```bash
> openssl rand -hex 32
> ```

### 3. Start n8n

```bash
docker compose up -d
```

n8n is now listening on `127.0.0.1:5678` — only accessible locally.

### 4. Configure Caddy on the host

Paste this block into `/etc/caddy/Caddyfile` (replace `n8n.example.com` with your domain):

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

Then reload Caddy:

```bash
sudo systemctl reload caddy
```

### 5. Access n8n

Open `https://<your-domain>` in your browser. Create your admin account on first visit.

## File Structure

```
.
├── docker-compose.yml       # Single n8n service (binds to 127.0.0.1)
├── .env.example             # Template – copy to .env and fill in
├── .env                     # Your actual secrets (git-ignored)
├── caddy-block-snippet.md   # Caddy block to paste into /etc/caddy/Caddyfile
├── local_files/             # Directory mounted into n8n at /files
└── README.md                # This file
```

## Useful Commands

| Action | Command |
|---|---|
| Start n8n | `docker compose up -d` |
| Stop n8n | `docker compose down` |
| View logs | `docker compose logs -f n8n` |
| Restart n8n | `docker compose restart n8n` |
| Update n8n | `docker compose pull n8n && docker compose up -d` |
| Reload Caddy | `sudo systemctl reload caddy` |
| Full cleanup | `docker compose down -v` (⚠️ deletes volume) |

## Environment Variables Reference

| Variable | Default | Description |
|---|---|---|
| `N8N_HOST` | — | Public hostname |
| `N8N_PORT` | `5678` | Internal listen port |
| `N8N_PROTOCOL` | `https` | Protocol for webhook URLs |
| `N8N_ENCRYPTION_KEY` | — | Encryption key for credentials (required) |
| `N8N_PROXY_HOPS` | `0` | Number of reverse-proxy hops (set to `1` when behind Caddy) |
| `WEBHOOK_URL` | — | Override webhook URL |
| `EXECUTIONS_DATA_PRUNE` | `false` | Enable automatic pruning |
| `EXECUTIONS_DATA_MAX_AGE` | `168` | Max age in hours (7 days) |
| `EXECUTIONS_DATA_PRUNE_TIMEOUT` | `3600` | Prune timeout in seconds |
| `N8N_DIAGNOSTICS_ENABLED` | `true` | Telemetry / diagnostics |
| `N8N_PERSONALIZATION_ENABLED` | `true` | Onboarding personalization |
| `N8N_VERSION_NOTIFICATIONS_ENABLED` | `true` | Version update notifications |
| `GENERIC_TIMEZONE` | — | System timezone (e.g. `Europe/London`) |

## Caddy Block Snippet

Paste this into `/etc/caddy/Caddyfile` on the host:

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

See [`caddy-block-snippet.md`](caddy-block-snippet.md) for the standalone version.

## Security Notes

- Keep your [`.env`](.env) file **out of version control** — it is already listed in [`.gitignore`](.gitignore).
- Rotate `N8N_ENCRYPTION_KEY` regularly.
- n8n binds to `127.0.0.1` only — it is **not** exposed to the network. Only Caddy (on the host) can reach it.
- Use Caddy's automatic HTTPS in production.

## Updating

```bash
docker compose pull n8n
docker compose up -d