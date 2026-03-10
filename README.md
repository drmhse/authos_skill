# AuthOS Skills

A comprehensive collection of AI agent skills for managing, maintaining, and integrating the [AuthOS](https://authos.dev) authentication platform.

These skills are designed according to the [Agent Skills Specification](https://agentskills.io/) and are optimized for use by AI agents like Antigravity, ChatGPT, and Claude.

## 🚀 Overview

AuthOS is a powerful, multi-tenant authentication platform. This repository provides structured skills that allow AI agents to perform complex administrative and integration tasks on behalf of Platform Owners and Organization Admins.

## 🛠️ Included Skills

### Platform Management
- **[Platform Deployment](./authos-platform-deployment/SKILL.md)**: Infrastructure setup, RSA key generation, and multi-backend (Postgres/MySQL/SQLite) configuration.
- **[Tenancy Governance](./authos-tenancy-governance/SKILL.md)**: Organization approval workflows, tier management, and lifecycle operations.
- **[Platform Maintenance](./authos-platform-maintenance/SKILL.md)**: Health monitoring, security audits, and key rotation strategies.

### Organization Administration
- **[Identity Configuration](./authos-identity-configuration/SKILL.md)**: Bring Your Own OAuth (BYOO) for Google/GitHub/Microsoft and Enterprise SSO (SAML).
- **[Service Management](./authos-service-management/SKILL.md)**: Application architecture, client secrets, and multi-tier pricing plans.
- **[RBAC Control](./authos-rbac-control/SKILL.md)**: Custom roles, team invitations, and automated SCIM provisioning.

### Integration & Development
- **[Web Integration](./authos-web-integration/SKILL.md)**: Frontend integration via the "Invisible SDK" with auto-token rotation.
- **[Backend Integration](./authos-backend-integration/SKILL.md)**: Secure backend APIs by validating JWTs using JWKS and Node.js middleware.
- **[Device Flow](./authos-device-flow/SKILL.md)**: Input-constrained authentication (RFC 8628) for CLIs and IoT devices.
- **[Webhook Integration](./authos-webhook-integration/SKILL.md)**: Real-time event sync with HMAC signature verification.
- **[Compliance Automation](./authos-compliance-automation/SKILL.md)**: GDPR/SOC2 readiness, data exports, and SIEM streaming.

## 📦 Installation

To add these skills to your agent's environment using the `skills` CLI:

```bash
npx skills add drmhse/authos_skill
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
