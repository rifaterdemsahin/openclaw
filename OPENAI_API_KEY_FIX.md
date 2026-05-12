# OpenAI API Key Fix Report

## Issue

**Error:** `No API key found for provider "openai"`

When sending a message through the OpenClaw Control UI, the agent failed because no OpenAI API key was configured.

**Log Location:**
```
Auth store: /data/agents/main/agent/auth-profiles.json
agentDir: /data/agents/main/agent
```

## Root Cause

OpenClaw requires an API key for the AI provider (OpenAI in this case) to process messages. The gateway was deployed without setting the `OPENAI_API_KEY` environment variable.

## Solution

### Step 1: Save API Key to Azure Key Vault

The API key was saved to Azure Key Vault for secure storage:

```powershell
$SecureString = ConvertTo-SecureString -String "<API_KEY>" -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName openshifthelper -Name "OpenAI-API-KEY" -SecretValue $SecureString
```

**Key Vault Details:**
| Property | Value |
|----------|-------|
| Vault Name | `openshifthelper` |
| Secret Name | `OpenAI-API-KEY` |
| Status | Saved |

### Step 2: Set Fly.io Secret

The API key was set as a Fly.io secret, which gets injected as an environment variable into the running container:

```bash
flyctl secrets set OPENAI_API_KEY=<API_KEY> --app openclaw-fly-lhr-20260512
```

**What Happens:**
- Fly.io performs a rolling deployment
- The secret is mounted into the container at `/run/secrets/OPENAI_API_KEY`
- OpenClaw reads it as an environment variable
- The gateway restarts with the new secret available

### Step 3: Verify

```bash
# Check app status
flyctl status --app openclaw-fly-lhr-20260512

# Test health endpoint
curl -I https://openclaw-fly-lhr-20260512.fly.dev/healthz
# Expected: HTTP/1.1 200 OK
```

## Result

| Before | After |
|--------|-------|
| `No API key found for provider "openai"` | OpenAI API key configured |
| Agent fails to reply | Agent can process messages |

## How to Add Other Providers

You can add API keys for other AI providers using the same method:

```bash
# Anthropic (Claude)
flyctl secrets set ANTHROPIC_API_KEY=sk-ant-... --app openclaw-fly-lhr-20260512

# Google (Gemini)
flyctl secrets set GEMINI_API_KEY=... --app openclaw-fly-lhr-20260512

# OpenRouter
flyctl secrets set OPENROUTER_API_KEY=sk-or-... --app openclaw-fly-lhr-20260512
```

## Security Notes

- **Never commit API keys to git** — always use Fly.io secrets or Azure Key Vault
- **Rotate keys periodically** — especially if accidentally exposed
- **Use least privilege** — only set the keys for providers you actually use
- **Fly secrets are encrypted** at rest and in transit

## References

- [OpenClaw Models Configuration](https://docs.openclaw.ai/concepts/models)
- [Fly.io Secrets Docs](https://fly.io/docs/apps/secrets/)
- [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)
