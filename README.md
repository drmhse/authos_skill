# AuthOS Skills

Source-verified Agent Skills for working with AuthOS. These skills were rewritten from the actual AuthOS source code, not from project docs, and target the current public package names and API surfaces.

Canonical repository: [github.com/drmhse/authos_skill](https://github.com/drmhse/authos_skill)

Public AuthOS links:

- Main site: [authos.dev](https://authos.dev/)
- Documentation: [authos.dev/docs](https://authos.dev/docs/)
- AI Agent Skills guide: [authos.dev/docs/ai-agent-skills](https://authos.dev/docs/ai-agent-skills/)
- AuthOS source repository: [github.com/drmhse/AuthOS](https://github.com/drmhse/AuthOS)

AuthOS package names used by these skills:

- `@drmhse/sso-sdk`
- `@drmhse/authos-node`
- `@drmhse/authos-react`
- `@drmhse/authos-vue`

## Skills

### Platform

- [Platform Deployment](./authos-platform-deployment/SKILL.md): Deploy the Rust API with SQLite, PostgreSQL, or MySQL; configure JWT keys, billing, SMTP, GeoIP, Docker, and health checks.
- [Platform Maintenance](./authos-platform-maintenance/SKILL.md): Operate health, metrics, jobs, token refresh, webhook delivery failures, MFA metrics, and key rotation.
- [Tenancy Governance](./authos-tenancy-governance/SKILL.md): Manage platform-owner tenant lifecycle, tiers, feature overrides, platform users, impersonation, analytics, and audit logs.

### Tenant Administration

- [Identity Configuration](./authos-identity-configuration/SKILL.md): Configure BYOO OAuth, upstream enterprise providers, HRD, custom domains, branding, SMTP, SAML IdP, passkeys, MFA, and provider-token reauth.
- [Service Management](./authos-service-management/SKILL.md): Create and configure organization services, client credentials, redirect URIs, device activation URIs, plans, API keys, SAML, and checkout.
- [RBAC Control](./authos-rbac-control/SKILL.md): Manage organization members, custom roles, capability strings, invitations, service access, and SCIM tokens.
- [Compliance Automation](./authos-compliance-automation/SKILL.md): Use privacy export/anonymization, audit logs, SIEM config, tenant risk events, and MFA evidence surfaces.

### Application Integration

- [Web Integration](./authos-web-integration/SKILL.md): Integrate browser, React, Vue, and TypeScript apps with OAuth callbacks, password login, magic links, passkeys, MFA, and token refresh.
- [Backend Integration](./authos-backend-integration/SKILL.md): Verify AuthOS JWTs with JWKS and the Node server adapter; protect Express or other backend APIs.
- [Service API Integration](./authos-service-api-integration/SKILL.md): Use `X-Api-Key` service-to-service routes for users, subscriptions, service analytics, service metadata, and provider-token requests.
- [Device Flow](./authos-device-flow/SKILL.md): Implement device authorization for CLIs, TVs, headless apps, and platform admin CLI login.
- [Webhook Integration](./authos-webhook-integration/SKILL.md): Manage tenant webhooks, verify live delivery signatures, and inspect delivery history.

## Format

Each skill follows the Agent Skills directory structure with a required `SKILL.md`. Skills also include `agents/openai.yaml` metadata for product-facing discovery.

## Installation

The portable Agent Skills shape is:

```text
skill-name/
├── SKILL.md
├── scripts/
├── references/
└── assets/
```

Only `SKILL.md` is required. The optional directories are included when a skill needs executable helpers, extra reference material, or static assets.

Clone the canonical repo first:

```bash
git clone https://github.com/drmhse/authos_skill.git ~/authos_skill
```

Install all AuthOS skills by copying the `authos-*` skill directories into the target agent's skills directory:

```bash
# Codex
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R ~/authos_skill/authos-* "${CODEX_HOME:-$HOME/.codex}/skills/"

# Claude Code
mkdir -p ~/.claude/skills
cp -R ~/authos_skill/authos-* ~/.claude/skills/

# Cursor
mkdir -p ~/.cursor/skills
cp -R ~/authos_skill/authos-* ~/.cursor/skills/

# Gemini CLI
mkdir -p ~/.gemini/skills
cp -R ~/authos_skill/authos-* ~/.gemini/skills/

# GitHub Copilot
mkdir -p ~/.copilot/skills
cp -R ~/authos_skill/authos-* ~/.copilot/skills/
```

For project-scoped installs, copy the same `authos-*` directories into the project-level location your agent supports:

| Agent | Project-level path | Personal path |
| --- | --- | --- |
| Codex | `.agents/skills/<skill-name>/SKILL.md` where supported | `${CODEX_HOME:-~/.codex}/skills/<skill-name>/SKILL.md` |
| Claude Code | `.claude/skills/<skill-name>/SKILL.md` | `~/.claude/skills/<skill-name>/SKILL.md` |
| Cursor | `.cursor/skills/<skill-name>/SKILL.md` | `~/.cursor/skills/<skill-name>/SKILL.md` |
| Gemini CLI | `.gemini/skills/<skill-name>/SKILL.md` or `.agents/skills/<skill-name>/SKILL.md` | `~/.gemini/skills/<skill-name>/SKILL.md` or `~/.agents/skills/<skill-name>/SKILL.md` |
| GitHub Copilot | `.github/skills/<skill-name>/SKILL.md`, `.claude/skills/<skill-name>/SKILL.md`, or `.agents/skills/<skill-name>/SKILL.md` | `~/.copilot/skills/<skill-name>/SKILL.md` or `~/.agents/skills/<skill-name>/SKILL.md` |

After copying skills, restart the agent session or run that agent's skill reload command if it has one.

## Visibility

Before publishing or announcing this repository:

- Set the GitHub repo description to `Source-verified Agent Skills for implementing and operating AuthOS.`
- Set the repo homepage to `https://authos.dev/docs/ai-agent-skills/`.
- Add GitHub topics: `agent-skills`, `codex-skills`, `claude-skills`, `cursor-skills`, `gemini-cli`, `copilot-skills`, `ai-agents`, `authos`, `authentication`, `sso`, `oauth`, `identity`.
- Link this repo from both AuthOS public websites: the main AuthOS site and the AuthOS docs site.
- Publish the skills to a shared or public agentregistry instance as Git-backed skill references only when you have its registry URL and token; the local `localhost:12121` daemon is not public distribution.
- Add a license before broad public submission if one is not already present.

See [PUBLISHING.md](./PUBLISHING.md) for the full release and directory-submission checklist.
