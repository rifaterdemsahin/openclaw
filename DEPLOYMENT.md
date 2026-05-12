# OpenClaw Fly.io Deployment Guide

This document explains how the official [OpenClaw](https://github.com/openclaw/openclaw) Docker image was deployed to [Fly.io](https://fly.io) from the local workspace.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [What We Deployed](#what-we-deployed)
3. [Phase 1: Codebase Inspection](#phase-1-codebase-inspection)
4. [Phase 2: Configuration Generation](#phase-2-configuration-generation)
5. [Phase 3: Secret & Environment Management](#phase-3-secret--environment-management)
6. [Phase 4: Build & Local Testing](#phase-4-build--local-testing)
7. [Phase 5: Fly.io Deployment](#phase-5-flyio-deployment)
8. [Phase 6: Validation & Monitoring](#phase-6-validation--monitoring)
9. [Deployment Result](#deployment-result)
10. [Post-Deployment Operations](#post-deployment-operations)
11. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, verify the following tools are installed and authenticated:

| Tool | Command | Status |
|------|---------|--------|
| Fly CLI | `flyctl version` | v0.4.35 |
| Fly Auth | `flyctl auth whoami` | rifaterdemsahin@gmail.com |
| Docker | `docker --version` | v29.4.1 |
| Git | `git --version` | v2.54.0 |

---

## What We Deployed

- **Project**: [OpenClaw](https://openclaw.ai) — Personal AI Assistant
- **Image**: `ghcr.io/openclaw/openclaw:latest` (official pre-built image)
- **Platform**: Fly.io (London region `lhr`)
- **Method**: Remote image deployment (no local Docker build required)

---

## Phase 1: Codebase Inspection

1. **Checked the local repository** at `C:\projects\openclaw`.
2. **Found it empty** — only a `README.md` existed.
3. **Searched Docker Hub & GitHub** for existing OpenClaw images.
4. **Identified the official image** at `ghcr.io/openclaw/openclaw:latest` (371k+ stars, actively maintained).
5. **Reviewed the official `fly.toml`** and `Dockerfile` from the upstream repo to understand port and process requirements.

Key findings:
- Internal port: **3000**
- Command: `node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan`
- Health endpoints: `/healthz`, `/readyz`
- Requires persistent volume at `/data`

---

## Phase 2: Configuration Generation

### Created `fly.toml`

```toml
app = "openclaw-fly-lhr-20260512"
primary_region = "lhr"

[build]
image = "ghcr.io/openclaw/openclaw:latest"

[env]
NODE_ENV = "production"
OPENCLAW_PREFER_PNPM = "1"
OPENCLAW_STATE_DIR = "/data"
NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
internal_port = 3000
force_https = true
auto_stop_machines = false
auto_start_machines = true
min_machines_running = 1
processes = ["app"]

[[vm]]
size = "shared-cpu-2x"
memory = "2048mb"

[mounts]
source = "openclaw_data"
destination = "/data"
```

### Created `.dockerignore`

```
.git
__pycache__
.env
.env.local
node_modules
.DS_Store
```

---

## Phase 3: Secret & Environment Management

1. **Generated a secure gateway token** using PowerShell:
   ```powershell
   $token = -join ((1..32) | ForEach-Object { "{0:x2}" -f (Get-Random -Maximum 256) })
   ```

2. **Set the secret via Fly CLI** (use the actual token from your Key Vault):
   ```bash
   flyctl secrets set OPENCLAW_GATEWAY_TOKEN=<TOKEN_FROM_KEY_VAULT> --app openclaw-fly-lhr-20260512
   ```

3. **Saved the token to Azure Key Vault** (`openshifthelper`):
   ```powershell
   $SecureString = ConvertTo-SecureString -String "<TOKEN>" -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName openshifthelper -Name "OpenClawGatewayToken" -SecretValue $SecureString
   ```

4. **Confirmed secrets are staged** for the first deployment.

> **Note:** The gateway refuses to start if authentication is missing when binding to a non-loopback address. Setting `OPENCLAW_GATEWAY_TOKEN` satisfies this requirement.

---

## Phase 4: Build & Local Testing

- **Skipped local Docker build** because Fly.io pulls the pre-built image directly from `ghcr.io`.
- The official image is already optimized (multi-stage build, `node:24-bookworm-slim` runtime, runs as non-root `node` user).

If you prefer to build locally for testing:
```bash
docker build -t openclaw:local -f Dockerfile .
docker run -p 3000:3000 -e OPENCLAW_GATEWAY_TOKEN=<token> openclaw:local
```

---

## Phase 5: Fly.io Deployment

### Step 1: Create the app
```bash
flyctl apps create openclaw-fly-lhr-20260512 --org personal -y
```

### Step 2: Create persistent volume
```bash
flyctl volumes create openclaw_data --region lhr --size 1 --app openclaw-fly-lhr-20260512 -y
```

### Step 3: Deploy
```bash
flyctl deploy --app openclaw-fly-lhr-20260512
```

**Deployment output highlights:**
- Image found remotely: `img_2wokpyo7y87k43g1`
- Dedicated IPv6 and shared IPv4 provisioned
- Machine `e829425b6e3038` created successfully
- DNS verified for `openclaw-fly-lhr-20260512.fly.dev`

---

## Phase 6: Validation & Monitoring

### Status Check
```bash
flyctl status --app openclaw-fly-lhr-20260512
```
Result: Machine `started` in region `lhr`.

### Health Endpoint Test
```bash
curl -I https://openclaw-fly-lhr-20260512.fly.dev/healthz
```
Result: `HTTP/1.1 200 OK`

### Root URL Test
```bash
curl -s -o /dev/null -w "%{http_code}" https://openclaw-fly-lhr-20260512.fly.dev/
```
Result: `200`

### Log Inspection
```bash
flyctl logs --app openclaw-fly-lhr-20260512 --no-tail
```
Key log lines observed:
- `loading configuration...`
- `starting HTTP server...`
- `http server listening (6 plugins; 5.8s)`
- `ready`

---

## Deployment Result

| Property | Value |
|----------|-------|
| **App Name** | `openclaw-fly-lhr-20260512` |
| **Public URL** | `https://openclaw-fly-lhr-20260512.fly.dev` |
| **Health Check** | `https://openclaw-fly-lhr-20260512.fly.dev/healthz` |
| **Region** | `lhr` (London) |
| **Image** | `ghcr.io/openclaw/openclaw:latest` |
| **VM** | `shared-cpu-2x` / 2048 MB RAM |
| **Volume** | `openclaw_data` (1 GB, encrypted) |
| **Gateway Token** | *See Azure Key Vault `openshifthelper` → `OpenClawGatewayToken`* |

---

## Post-Deployment Operations

### Open the Control UI
Visit `https://openclaw-fly-lhr-20260512.fly.dev` and paste the **Gateway Token** into Settings.

> **If you see "origin not allowed"**: The Fly.io public URL must be added to `gateway.controlUi.allowedOrigins`. This was fixed by updating `/data/openclaw.json` on the running machine:
> ```bash
> # Upload and run a small Node.js script to update the config
> flyctl ssh sftp put --app openclaw-fly-lhr-20260512 update-origins.js /tmp/update-origins.js
> flyctl ssh console --app openclaw-fly-lhr-20260512 --command "node /tmp/update-origins.js"
> flyctl apps restart openclaw-fly-lhr-20260512
> ```
> The allowed origins now include:
> - `http://localhost:3000`
> - `http://127.0.0.1:3000`
> - `https://openclaw-fly-lhr-20260512.fly.dev`
>
> **If you see "device pairing required"**: OpenClaw treats every new browser as an untrusted device. Approve it via SSH:
> ```bash
> flyctl ssh console --app openclaw-fly-lhr-20260512 --command "node dist/index.js devices approve <requestId>"
> ```
> Then refresh the browser. See [Troubleshooting](#troubleshooting) for more details.

### Secret Management (Azure Key Vault)

The Gateway Token has been saved to the following Key Vaults:

| Vault | Subscription | Status |
|-------|-------------|--------|
| `openshifthelper` | Azure DevTest subscription 1 | ✅ Saved as `OpenClawGatewayToken` |
| `dp-kv-deliverypilot` | deliverypilot-rg | ⏳ Requires manual save (MFA tenant) |

#### Save to `dp-kv-deliverypilot` manually
Because this subscription requires MFA against tenant `de4adc2e-4d6f-4aab-b150-67c369a12924`, you must run the command in your own terminal where you are already authenticated:

**Azure CLI:**
```bash
az account set --subscription b85b029d-9f7c-4c5a-8939-819480780c5d
az keyvault secret set --vault-name dp-kv-deliverypilot --name OpenClawGatewayToken --value <TOKEN>
```

**PowerShell:**
```powershell
Set-AzContext -Subscription b85b029d-9f7c-4c5a-8939-819480780c5d
$SecureString = ConvertTo-SecureString -String "<TOKEN>" -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName dp-kv-deliverypilot -Name "OpenClawGatewayToken" -SecretValue $SecureString
```

### Add an AI Model Provider
Set an API key (e.g., OpenAI) via secrets:
```bash
flyctl secrets set OPENAI_API_KEY=sk-... --app openclaw-fly-lhr-20260512
```

Or configure via the dashboard/CLI.

### Scale Up
```bash
flyctl scale count 2 --app openclaw-fly-lhr-20260512
flyctl scale vm shared-cpu-2x --memory 4096 --app openclaw-fly-lhr-20260512
```

### Auto-Scaling
```bash
flyctl autoscale set min=1 max=3 --app openclaw-fly-lhr-20260512
```

### View Logs
```bash
flyctl logs --app openclaw-fly-lhr-20260512
```

### Rollback
```bash
flyctl releases --app openclaw-fly-lhr-20260512
flyctl rollback <version> --app openclaw-fly-lhr-20260512
```

---

## Troubleshooting

| Symptom | Solution |
|---------|----------|
| `WARNING: app is not listening on expected address` | This can appear during the 8-second gateway bootstrap. The app becomes reachable shortly after. Verify with `curl /healthz`. |
| Gateway returns 401/403 | Ensure `OPENCLAW_GATEWAY_TOKEN` is set via `flyctl secrets list --app <app>`. |
| OOM during build | Not applicable for remote-image deploys. If building locally, ensure 2 GB+ RAM. |
| Volume permission errors | The image runs as `node` (uid 1000). Fly's init ensures `/data` is mounted with uid 1000. |
| Slow cold start | Increase VM memory or use a larger CPU class. |
| **Device pairing required** (`requestId: ...`) | When opening the Control UI from a new browser/device, OpenClaw requires explicit device approval for security. <br><br> **Fix via CLI:** <br> 1. List pending devices: <br> `flyctl ssh console --app openclaw-fly-lhr-20260512 --command "node dist/index.js devices list"` <br> 2. Approve the device: <br> `flyctl ssh console --app openclaw-fly-lhr-20260512 --command "node dist/index.js devices approve <requestId>"` <br> 3. Refresh the browser. <br><br> **Alternative:** Disable device pairing (not recommended for public deployments): <br> `flyctl ssh console --app openclaw-fly-lhr-20260512 --command "node dist/index.js config set gateway.devicePairing.enabled false"` |
| **Origin not allowed** | The Fly.io public URL is not in `gateway.controlUi.allowedOrigins`. See [Open the Control UI](#open-the-control-ui) section for the fix. |

---

## References

- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw Docker Docs](https://docs.openclaw.ai/install/docker)
- [OpenClaw Fly.toml (upstream)](https://github.com/openclaw/openclaw/blob/main/fly.toml)
- [Fly.io Docs](https://fly.io/docs/)
