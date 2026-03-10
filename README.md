# MinIO + Cloudflare Tunnel

Self-hosted S3-compatible object storage (MinIO) exposed publicly via a Cloudflare Tunnel — no open ports required.

## Architecture

```
Internet → Cloudflare Tunnel → cloudflared → minio:9000  (S3 API)
                                           → minio:9001  (Console)
```

- **minio** — S3-compatible object storage (API on port 9000, Console on port 9001)
- **cloudflared** — Cloudflare Tunnel connector, routes public domain to MinIO
- **minio-init** — One-shot service that creates the bucket on first start, then exits

## Prerequisites

- Docker + Docker Compose
- A domain managed by Cloudflare
- A Cloudflare Zero Trust account (free tier works)

## Setup

### 1. Configure environment

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
CLOUDFLARE_TUNNEL_TOKEN=your-token-here
MINIO_ROOT_USER=minadmin
MINIO_ROOT_PASSWORD=your-password
MINIO_BUCKET=chamberlink
MINIO_REGION=ap-southeast-1
MINIO_ENDPOINT=https://s3.yourdomain.com
```

### 2. Create a Cloudflare Tunnel

1. Go to [Cloudflare Zero Trust](https://one.dash.cloudflare.com) → **Networks → Tunnels**
2. Create a new tunnel → choose **Docker** connector
3. Copy the tunnel token into `.env` as `CLOUDFLARE_TUNNEL_TOKEN`
4. Under **Public Hostnames**, add:
   - Hostname: `s3.yourdomain.com` → Service: `http://minio:9000`
   - Hostname: `minio.yourdomain.com` → Service: `http://minio:9001`

> **Security:** Consider adding a [Cloudflare Access](https://one.dash.cloudflare.com) policy on the console hostname to restrict who can reach it.

### 3. Start

```bash
docker compose up -d
```

The `minio-init` service will automatically create the configured bucket on first run.

## Usage

| Resource | URL |
|---|---|
| S3 API (public) | `https://s3.yourdomain.com` |
| MinIO Console (public) | `https://minio.yourdomain.com` |
| MinIO Console (local) | `http://localhost:9001` |

### S3 Client Config

```env
AWS_ENDPOINT=https://s3.yourdomain.com
AWS_ACCESS_KEY_ID=<MINIO_ROOT_USER>
AWS_SECRET_ACCESS_KEY=<MINIO_ROOT_PASSWORD>
AWS_DEFAULT_REGION=ap-southeast-1
AWS_USE_PATH_STYLE_ENDPOINT=true
```

## Making a Bucket Publicly Readable

By default buckets are private. To allow anyone to read objects without credentials:

### Option A — MinIO Console

1. Open the console (`https://minio.yourdomain.com` or `http://localhost:9001`)
2. Go to **Buckets** → select your bucket
3. Click **Access Policy** → set to **Public**

### Option B — `mc` CLI (inside the container)

The `minio` container doesn't have the alias pre-configured, so set it first:

```bash
docker exec minio mc alias set local http://localhost:9000 <MINIO_ROOT_USER> <MINIO_ROOT_PASSWORD>
docker exec minio mc anonymous set download local/<your-bucket>
```

After this, objects are accessible directly:

```
https://s3.yourdomain.com/<your-bucket>/<object-key>
```

## Common Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f

# Destroy all data (irreversible)
docker compose down -v
```

## Data Persistence

MinIO data is stored in the Docker named volume `minio_data`. It survives `docker compose down` but is deleted by `docker compose down -v`.
