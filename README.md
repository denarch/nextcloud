# Filestash for Cloudflare R2

Google Drive-like web interface for your Cloudflare R2 storage. Connects to the **same** R2 bucket where Synology NAS syncs files, so you can browse, upload, share, and get links.

**Target URL:** `https://files.denarchvisuals.com`

## Quick links (Synology + Cloudflare Tunnel)

1. [Что делать сейчас — чеклист](docs/QUICK-START.md)
2. [Установка Filestash на Synology](docs/SYNOLOGY-SETUP.md)
3. [Настройка Cloudflare Tunnel](docs/CLOUDFLARE-TUNNEL.md)

## Architecture

```
Browser (files.denarchvisuals.com) → Filestash → R2 (S3 API)
                                             ↑
                                    NAS sync to R2
```

## Requirements

- Docker and Docker Compose
- VPS (DigitalOcean, Hetzner, etc.) or Synology NAS with Container Manager
- Cloudflare R2 bucket with API credentials

## Quick Start

### 1. Clone and configure

```bash
git clone https://github.com/denarch/nextcloud.git filestash-r2 && cd filestash-r2
cp .env.example .env
```

### 2. Generate APP_SECRET

```bash
# Linux/macOS
openssl rand -hex 32

# Windows PowerShell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }) -as [byte[]])
```

Add the output to `.env`:

```
APP_SECRET=your-generated-secret
```

### 3. Run Filestash

```bash
docker compose up -d
```

Filestash will be available at `http://localhost:8334`.

### 4. Connect Cloudflare R2

On first access, set admin password and configure R2:

1. Go to **Admin** (gear icon) → **Storage**
2. Enable **S3** backend
3. Add connection:
   - **Storage type:** S3 (or AWS S3)
   - **Endpoint:** `https://<ACCOUNT_ID>.r2.cloudflarestorage.com`
   - **Bucket:** your R2 bucket name
   - **Region:** `auto`
   - **Access Key ID:** from Cloudflare R2 → Manage API Tokens
   - **Secret Access Key:** from Cloudflare R2 → Manage API Tokens

#### Getting R2 credentials

1. [Cloudflare Dashboard](https://dash.cloudflare.com) → **R2 Object Storage**
2. **Manage** next to API Tokens → **Create API Token**
3. Permissions: **Object Read & Write**, apply to your bucket
4. Copy Access Key ID and Secret Access Key (shown once)
5. Account ID: R2 Overview page or bucket Settings → S3 API section

## Domain Setup (files.denarchvisuals.com)

### Option A: VPS with reverse proxy

1. Point DNS: `files` CNAME → your VPS IP or hostname
2. Use Nginx/Caddy as reverse proxy to `localhost:8334`
3. Enable SSL (Let's Encrypt or Cloudflare Origin Certificate)
4. Set Cloudflare SSL/TLS to **Full (strict)**

### Option B: Synology + Cloudflare Tunnel (recommended)

See [CLOUDFLARE-TUNNEL.md](docs/CLOUDFLARE-TUNNEL.md) for step-by-step instructions.

1. Install Cloudflare Tunnel (cloudflared) in Docker on Synology
2. Create tunnel and route `files.denarchvisuals.com` to `http://host.docker.internal:8334` or NAS IP
3. Cloudflare handles SSL; no port forwarding needed

### Example Nginx config

```nginx
server {
    listen 443 ssl http2;
    server_name files.denarchvisuals.com;

    ssl_certificate     /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:8334;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Sharing files

Filestash supports sharing with:

- Password protection
- Expiration date
- Access restrictions

Use the share icon next to files or folders in the web interface.

## Troubleshooting

- **Connection refused:** Ensure container is running (`docker compose ps`) and port 8334 is not blocked
- **R2 auth failed:** Check endpoint format `https://<ACCOUNT_ID>.r2.cloudflarestorage.com` and credentials
- **Empty bucket:** Verify bucket name and that NAS sync has populated files

## Upgrade

```bash
docker compose pull
docker compose up -d
```
