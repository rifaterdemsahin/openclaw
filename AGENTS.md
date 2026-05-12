# Agent Instructions for OpenClaw

## Project Context
This repository contains the Fly.io deployment configuration and documentation for the OpenClaw personal AI assistant.

## Critical Rules

### NEVER Commit Secrets or Tokens
- **Do NOT** include passwords, API keys, tokens, or any sensitive credentials in any committed file.
- **Do NOT** include them in markdown documentation, code comments, config files, or scripts.
- Always use placeholders like `<TOKEN>` or `<API_KEY>` in committed files.
- Store real secrets in Azure Key Vault or Fly.io secrets.
- If a secret accidentally ends up in a file, rotate it immediately and rewrite git history if it was committed.
- When updating documentation that references secrets, replace the real value with a reference to where it is stored (e.g., "See Azure Key Vault `openshifthelper` secret `OpenClawGatewayToken'`").

## Technology Stack
- Fly.io (deployment platform)
- Docker / container images
- OpenClaw Gateway (Node.js-based AI assistant)

## Files & Conventions
- `fly.toml` — Fly.io app configuration
- `DEPLOYMENT.md` — Deployment and troubleshooting guide
- `.dockerignore` — Docker build exclusions
