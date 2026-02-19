---
name: authos-platform-deployment
description: Deploy and configure the core AuthOS infrastructure, including environment variables, database backends (SQLite, PostgreSQL, MySQL), and Docker-based deployment. Use this when a user needs to set up a new AuthOS instance or modify the base application configuration.
---

# AuthOS Platform Deployment

This skill provides step-by-step instructions for deploying and configuring the AuthOS authentication platform.

## Prerequisites

- **Rust**: Version 1.89+ (for source builds)
- **Docker & Docker Compose**: For containerized deployment
- **OpenSSL**: For generating security keys

## 1. Key Generation (Mandatory)

Before deployment, you must generate asymmetric RSA keys for JWT signing and a symmetric key for database encryption.

```bash
# RSA keys for JWT
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# Base64 encode for environment variables
JWT_PRIVATE_KEY_BASE64=$(base64 -i private.pem | tr -d '\n')
JWT_PUBLIC_KEY_BASE64=$(base64 -i public.pem | tr -d '\n')

# Encryption key for BYOO credentials (32-byte hex)
ENCRYPTION_KEY=$(openssl rand -hex 32)
```

## 2. Choosing a Database Backend

AuthOS supports three backends via compile-time feature flags.

| Backend | Feature Flag | Best For |
|---------|--------------|----------|
| **SQLite** | `db_sqlite` | Single-server, simple deployment, <10k users. |
| **PostgreSQL**| `db_psql` | Multi-server, High Availability, >10k users. |
| **MySQL** | `db_mysql` | Existing MySQL infrastructure. |

## 3. Environment Configuration

Create a `.env` file in the `api/` directory.

### Core Variables
- `DATABASE_URL`: Connection string (e.g., `sqlite:./data/data.db`, `postgres://user:pass@host:5432/db`)
- `JWT_PRIVATE_KEY_BASE64`: The private key generated in step 1.
- `JWT_PUBLIC_KEY_BASE64`: The public key generated in step 1.
- `JWT_KID`: A unique identifier for the current key (e.g., `prod-2025-01`).
- `ENCRYPTION_KEY`: The 32-byte hex key generated in step 1.
- `BASE_URL`: Public URL of the API (e.g., `https://api.authos.dev`).
- `PLATFORM_OWNER_EMAIL`: Email of the super-admin to be auto-created.

### Platform Admin OAuth (Required for UI Login)
Configure at least one for access to the admin dashboard:
- `PLATFORM_GITHUB_CLIENT_ID` / `PLATFORM_GITHUB_CLIENT_SECRET`
- `PLATFORM_GOOGLE_CLIENT_ID` / `PLATFORM_GOOGLE_CLIENT_SECRET`
- `PLATFORM_MICROSOFT_CLIENT_ID` / `PLATFORM_MICROSOFT_CLIENT_SECRET`

## 4. Deployment Methods

### A. Docker Compose (Recommended)

Use the built-in profiles to start the app with your preferred database.

```bash
# SQLite (Default)
docker-compose up -d sso-sqlite

# PostgreSQL
docker-compose --profile postgres up -d

# MySQL
docker-compose --profile mysql up -d
```

### B. Building from Source

```bash
# For PostgreSQL build
cargo build --release --no-default-features --features db_psql

# Run the binary (matches the backend name)
./target/release/sso_psql
```

## 5. Reverse Proxy (Production)

Always run AuthOS behind a reverse proxy (Nginx, Caddy, or Traefik) for SSL termination.

**Nginx Example:**
```nginx
location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

## 6. Verification

Poll the health endpoints to ensure the system is operational.

```bash
# Basic health check
curl https://api.authos.dev/health

# Detailed readiness check (includes DB connectivity)
curl https://api.authos.dev/health/ready
```

## Common Edge Cases

- **Missing `ENCRYPTION_KEY`**: The API will fail to start if this is missing or the wrong length (must be 64 hex chars).
- **Incompatible Binary**: Ensure the `DATABASE_URL` protocol matches the binary type (e.g., `sso_psql` requires `postgres://`).
- **Migration Failures**: Migrations run automatically on startup. If they fail, check database permissions and connectivity.
