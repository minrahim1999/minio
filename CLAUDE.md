# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project runs a MinIO object storage server exposed via a Cloudflare Tunnel using Docker Compose. A one-shot `minio-init` service (using `mc`) automatically creates the required bucket on first start.

## Start / Stop Commands

```bash
# Start all services (detached)
docker compose up -d

# Stop all services
docker compose down

# Stop and remove volumes (WARNING: destroys all stored data)
docker compose down -v

# View logs
docker compose logs -f

# View logs for a specific service
docker compose logs -f minio
docker compose logs -f cloudflared
docker compose logs -f minio-init
```

## Getting a Cloudflare Tunnel Token

1. Go to the Cloudflare Zero Trust dashboard: https://one.dash.cloudflare.com/
2. Navigate to Networks > Tunnels.
3. Create a new tunnel (or select an existing one).
4. Choose "Docker" as the connector type to obtain the token.
5. Copy the token value and paste it into `.env` as `CLOUDFLARE_TUNNEL_TOKEN`.
6. In the tunnel's Public Hostname settings, point your domain to `http://minio:9000` (for the S3 API).

## MinIO Console

The MinIO web console is available locally at:

```
http://localhost:9001
```

Log in with the credentials defined in `.env` (`MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`).

## Key Environment Variables

| Variable                  | Description                                              |
|---------------------------|----------------------------------------------------------|
| `CLOUDFLARE_TUNNEL_TOKEN` | Token for the Cloudflare Tunnel (keep secret)            |
| `MINIO_ROOT_USER`         | MinIO admin username                                     |
| `MINIO_ROOT_PASSWORD`     | MinIO admin password                                     |
| `MINIO_BUCKET`            | Name of the bucket auto-created on first start           |
| `MINIO_REGION`            | AWS-compatible region string (e.g. `ap-southeast-1`)     |
| `MINIO_ENDPOINT`          | Public S3 endpoint via Cloudflare Tunnel                 |

## S3 Client Configuration

When connecting to MinIO from application code or AWS CLI, use path-style addressing:

- Endpoint: `https://s3.metasenvironment.com`
- Region: `ap-southeast-1`
- Force path style: `true`
- Access Key: value of `MINIO_ROOT_USER`
- Secret Key: value of `MINIO_ROOT_PASSWORD`

## Important Notes

- `.env` contains real credentials and must NOT be committed to version control. It is listed in `.gitignore`.
- `.env.example` is safe to commit and serves as a template for new environments.
- MinIO data is persisted in the named Docker volume `minio_data`.
- The `minio-init` container runs once to create the bucket and then exits cleanly (`restart: "no"`).
