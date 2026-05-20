# Publishing AuthOS Skills

This repository is the canonical public source for AuthOS Agent Skills:

https://github.com/CkCreative/authos_skill

The skills are source-verified against the AuthOS codebase and organized as portable Agent Skills: one directory per skill, required `SKILL.md`, optional `scripts/`, `references/`, and `assets/`, plus `agents/openai.yaml` metadata where present.

Public AuthOS links to keep visible from every publishing surface:

- Main site: `https://authos.dev/`
- Documentation: `https://authos.dev/docs/`
- AI Agent Skills guide: `https://authos.dev/docs/ai-agent-skills/`
- AuthOS source repository: `https://github.com/drmhse/AuthOS`

## GitHub Repository Setup

Use these GitHub settings before promoting the repo:

- Description: `Source-verified Agent Skills for implementing and operating AuthOS.`
- Website: `https://authos.dev/docs/ai-agent-skills/`
- Topics: `agent-skills`, `codex-skills`, `claude-skills`, `cursor-skills`, `gemini-cli`, `copilot-skills`, `ai-agents`, `authos`, `authentication`, `sso`, `oauth`, `identity`

Also confirm:

- The default branch is public and readable without authentication.
- A license is present before submitting to public directories.
- `README.md` includes current install paths for Codex, Claude Code, Cursor, Gemini CLI, and GitHub Copilot.
- The AuthOS main site and docs site both link to this repository.

## Install Paths

Install all skills by cloning the repo and copying every `authos-*` directory into the target skills directory:

```bash
git clone https://github.com/CkCreative/authos_skill.git ~/authos_skill
```

| Agent | Project-level path | Personal path |
| --- | --- | --- |
| Codex | `.agents/skills/<skill-name>/SKILL.md` where supported | `${CODEX_HOME:-~/.codex}/skills/<skill-name>/SKILL.md` |
| Claude Code | `.claude/skills/<skill-name>/SKILL.md` | `~/.claude/skills/<skill-name>/SKILL.md` |
| Cursor | `.cursor/skills/<skill-name>/SKILL.md` | `~/.cursor/skills/<skill-name>/SKILL.md` |
| Gemini CLI | `.gemini/skills/<skill-name>/SKILL.md` or `.agents/skills/<skill-name>/SKILL.md` | `~/.gemini/skills/<skill-name>/SKILL.md` or `~/.agents/skills/<skill-name>/SKILL.md` |
| GitHub Copilot | `.github/skills/<skill-name>/SKILL.md`, `.claude/skills/<skill-name>/SKILL.md`, or `.agents/skills/<skill-name>/SKILL.md` | `~/.copilot/skills/<skill-name>/SKILL.md` or `~/.agents/skills/<skill-name>/SKILL.md` |

Copy commands:

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

For the most portable project-scoped placement, copy skills into `.agents/skills/` when the target tool supports it.

## AuthOS Site Links

Publish visibility through both AuthOS websites:

- Main site: `https://authos.dev/ai-agent-skills/`
- Docs site: `https://authos.dev/docs/ai-agent-skills/`

The docs page should be the deeper reference and should link to this repo, install paths, and skill coverage. The main site should be shorter and should send developers to the docs page or GitHub repo.

## Directory Submission Checklist

Submit the GitHub repo URL anywhere Agent Skills are indexed. Use this copy:

```text
AuthOS Skills are source-verified Agent Skills for implementing, integrating, and operating AuthOS. They cover platform deployment, tenant administration, web and backend integration, service APIs, device flow, webhooks, compliance, RBAC, and maintenance.
```

Suggested tags:

```text
AuthOS, authentication, SSO, OAuth, OIDC, SAML, RBAC, SCIM, webhooks, agent-skills, AI agents, Codex, Claude Code, Cursor, Gemini CLI, GitHub Copilot
```

Submit only after the GitHub repo, main site page, and docs page are live.
