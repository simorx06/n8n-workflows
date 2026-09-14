# n8n Workflows

Self-hosted [n8n](https://n8n.io) instance with **PostgreSQL** as the database and **Caddy** as the automated HTTPS reverse proxy.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/) & [Docker Compose](https://docs.docker.com/compose/install/) (v2.x+)
- A domain name pointing to your server (for Caddy / HTTPS)

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
| `N8N_DEFAULT_TIMEZONE` | Default timezone | `Europe/London` |
| `N8N_PAYLOAD_SIZE_MAX` | Max incoming payload in MB | `16` |
| `DB_TYPE` | Database type | `postgresdb` |
| `DB_POSTGRESDB_HOST` | PostgreSQL host (service name) | `postgres` |
| `DB_POSTGRESDB_PORT` | PostgreSQL port | `5432` |
| `DB_POSTGRESDB_DATABASE` | Database name | `n8n` |
| `DB_POSTGRESDB_USER` | Database user | `n8n` |
| `DB_POSTGRESDB_PASSWORD` | **Secret** – database password | `change-me` |
| `POSTGRES_USER` | PostgreSQL superuser | `n8n` |
| `POSTGRES_PASSWORD` | **Secret** – superuser password | `change-me` |
| `POSTGRES_DB` | PostgreSQL database name | `n8n` |
| `CADDY_DOMAIN` | Domain for Caddy to provision SSL | `n8n.example.com` |
| `CADDY_EMAIL` | Email for Let's Encrypt | `admin@example.com` |
| `GENERIC_TIMEZONE` | System timezone | `Europe/London` |

> **Important:** Generate a strong `N8N_ENCRYPTION_KEY`:
> ```bash
> openssl rand -hex 32
> ```

### 3. Start the stack

```bash
docker compose up -d
```

### 4. Access n8n

Open `https://<CADDY_DOMAIN>` in your browser. Create your admin account on first visit.

## File Structure

```
.
├── docker-compose.yml       # Service definitions (n8n, postgres, caddy)
├── .env.example             # Template – copy to .env and fill in
├── .env                     # Your actual secrets (git-ignored)
├── Caddyfile                # Caddy reverse-proxy configuration
├── caddy-block-snippet.md   # Standalone Caddy block for pasting
├── local_files/             # Directory mounted into n8n at /files
└── README.md                # This file
```

## Useful Commands

| Action | Command |
|---|---|
| Start services | `docker compose up -d` |
| Stop services | `docker compose down` |
| View logs (n8n) | `docker compose logs -f n8n` |
| View logs (Caddy) | `docker compose logs -f caddy` |
| Restart a service | `docker compose restart n8n` |
| Update n8n | `docker compose pull n8n && docker compose up -d` |
| Full cleanup | `docker compose down -v` (⚠️ deletes volumes) |

## Environment Variables Reference

### n8n Core

| Variable | Default | Description |
|---|---|---|
| `N8N_HOST` | — | Public hostname |
| `N8N_PORT` | `5678` | Internal listen port |
| `N8N_PROTOCOL` | `https` | Protocol for webhook URLs |
| `N8N_ENCRYPTION_KEY` | — | Encryption key for credentials (required) |
| `N8N_DEFAULT_TIMEZONE` | `America/New_York` | Default timezone |
| `N8N_PAYLOAD_SIZE_MAX` | `16` | Max payload size in MB |
| `N8N_SKIP_WEBHOOK_DEPLOYMENT` | `false` | Skip webhook deployment on start |
| `N8N_VERSION_NOTIFICATIONS_ENABLED` | `true` | Version update notifications |
| `N8N_DIAGNOSTICS_ENABLED` | `true` | Telemetry / diagnostics |
| `N8N_PERSONALIZATION_ENABLED` | `true` | Onboarding personalization |
| `WEBHOOK_URL` | — | Override webhook URL (e.g. `https://n8n.example.com/`) |

### Execution Data Pruning

| Variable | Default | Description |
|---|---|---|
| `EXECUTIONS_DATA_PRUNE` | `false` | Enable automatic pruning |
| `EXECUTIONS_DATA_MAX_AGE` | `168` | Max age in hours (7 days) |
| `EXECUTIONS_DATA_PRUNE_TIMEOUT` | `3600` | Prune timeout in seconds |

### Metrics (OpenTelemetry / Prometheus)

| Variable | Default | Description |
|---|---|---|
| `N8N_METRICS` | `false` | Enable metrics endpoint |
| `N8N_METRICS_INCLUDE_DEFAULT_METRICS` | `true` | Include default metrics |
| `N8N_METRICS_INCLUDE_MESSAGE_METRICS` | `false` | Include per-message metrics |
| `N8N_METRICS_INCLUDE_METADATA_LABELS` | `false` | Include metadata labels |
| `N8N_METRICS_INCLUDE_NODE_TYPE` | `false` | Include node type label |
| `N8N_METRICS_INCLUDE_WORKFLOW_ID` | `false` | Include workflow ID label |

### PostgreSQL

| Variable | Default | Description |
|---|---|---|
| `DB_TYPE` | `postgresdb` | Database type |
| `DB_POSTGRESDB_HOST` | `postgres` | PostgreSQL host |
| `DB_POSTGRESDB_PORT` | `5432` | PostgreSQL port |
| `DB_POSTGRESDB_DATABASE` | `n8n` | Database name |
| `DB_POSTGRESDB_USER` | `n8n` | Database user |
| `DB_POSTGRESDB_PASSWORD` | — | Database password |
| `POSTGRES_USER` | `n8n` | PostgreSQL superuser |
| `POSTGRES_PASSWORD` | — | PostgreSQL superuser password |
| `POSTGRES_DB` | `n8n` | PostgreSQL database name |

### Caddy

| Variable | Default | Description |
|---|---|---|
| `CADDY_DOMAIN` | — | Domain for automatic SSL |
| `CADDY_EMAIL` | — | Email for Let's Encrypt |

### General

| Variable | Default | Description |
|---|---|---|
| `GENERIC_TIMEZONE` | — | System timezone (e.g. `Europe/London`) |

## Caddy Block Snippet

If you already run Caddy separately, paste this block into your existing Caddyfile:

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

See [`caddy-block-snippet.md`](caddy-block-snippet.md) for the standalone version.

## Security Notes

- Keep your [`.env`](.env) file **out of version control** — it is already listed in `.gitignore` (create one if missing).
- Rotate `N8N_ENCRYPTION_KEY` and database passwords regularly.
- Restrict port `5432` (PostgreSQL) to internal Docker networks only — it is exposed for local tooling but should be firewalled in production.
- Use Caddy's automatic HTTPS in production; avoid exposing n8n directly on port `5678` without a reverse proxy.

## Updating

```bash
docker compose pull n8n
docker compose up -d
```

Caddy and PostgreSQL images can be updated the same way.