---
name: authos-platform-maintenance
description: Monitor and maintain the health, security, and performance of an AuthOS instance. Includes health checks, log analysis, master key rotation strategies, and managing platform-level security policies like MFA force-resets. Use this when a user needs to ensure the platform is running optimally or respond to security incidents.
---

# AuthOS Platform Maintenance

This skill provides guidelines and procedures for maintaining a healthy and secure AuthOS environment.

## 1. Health Monitoring

AuthOS provides specialized endpoints for monitoring service status.

### Health Endpoints
- **Liveness**: `GET /health/live` (Returns 200 OK if the process is running).
- **Readiness**: `GET /health/ready` (Returns 200 OK if the database is connected).
- **Public Health**: `GET /health` (Returns basic version and status info).

### Performance Metrics
If enabled, Prometheus metrics are available at `/metrics`. 
**Key Metrics to Monitor**:
- `sso_http_request_duration_seconds`: Request latency.
- `sso_db_pool_connections_total`: Database pool exhaustion indicator.
- `sso_active_users_total`: Use this for capacity planning.
- `sso_login_failures_total`: Watch for spikes indicating brute-force attacks.

## 2. Security Maintenance

### MFA Management (Platform Level)
If a user is locked out or loses their device, platform owners can manage their MFA status globally.
- **Check MFA Status**: `GET /api/platform/users/:user_id/mfa/status`
- **Force Disable MFA**: `DELETE /api/platform/users/:user_id/mfa`
- **Suspicious Activity**: `GET /api/platform/mfa/suspicious` (Lists impossible travel or brute force alerts).

### Key Rotation Strategy (Critical)
AuthOS uses two types of keys that require different rotation strategies.

#### JWT Keys (RS256)
- **Frequency**: Every 3-6 months.
- **Method**: 
  1. Generate new RSA keys.
  2. Update `JWT_PRIVATE_KEY_BASE64` and `JWT_PUBLIC_KEY_BASE64`.
  3. Change `JWT_KID`.
- **Note**: Old tokens signed with the old `KID` will immediately become invalid.

#### System Encryption Key (`ENCRYPTION_KEY`)
- **Warning**: Do NOT rotate this key unless you have a migration plan. Rotating this key will break all existing BYOO (OAuth) credentials and SMTP settings stored in the database.
- **Rotation Procedure**: Requires a custom double-decryption/re-encryption script (not provided natively).

## 3. Log Analysis

### Application Logs
The API logs to stdout by default. Monitor for these high-priority tags:
- `ERROR`: Indicates system failures (e.g., DB connection loss).
- `WARN`: Indicates non-fatal issues (e.g., validation errors, rate limits hit).
- `audit`: Tracks critical security events.

### Platform Audit Logs
Query the internal platform audit log for administrative activity.
- **Endpoint**: `GET /api/platform/audit-log`
- **Actions to watch**: `delete_organization`, `promote_platform_owner`, `force_disable_user_mfa`.

## 4. Database Maintenance

### SQLite Optimization
If using SQLite, the system automatically performs checkpoints every 10 seconds.
- **Manual VACUUM**: If the database file grows large after deletions, run `VACUUM;` directly on the SQLite file during a maintenance window.

### Backup Schedule
- **Database**: Daily backups (logical or physical).
- **Encryption Key**: Store the `ENCRYPTION_KEY` in a secure secrets manager (AWS Secrets Manager, HashiCorp Vault). **Losing this key is equivalent to total data loss for enterprise tenants.**

## Troubleshooting Procedures
1. **API returning 503**: Check `/health/ready`. If it fails, check the `DATABASE_URL` and database service status.
2. **Login Failures**: Check `JWT_PUBLIC_KEY_BASE64` and `JWT_PRIVATE_KEY_BASE64` base64 encoding integrity.
3. **Mails not sending**: Check the `SMTP_` environment variables or individual organization SMTP overrides.
