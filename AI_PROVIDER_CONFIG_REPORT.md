# AI Provider Configuration Report

## Issue

**Error:** `401 Incorrect API key provided`

The OpenClaw agent is failing because the API key configured as `OPENAI_API_KEY` is being rejected by OpenAI's API endpoint.

## Root Cause

The key provided (`sk-ZRnfH...UKZd`) is from **OpenCode.ai** (https://opencode.ai/workspace/.../keys), not from **OpenAI** (https://platform.openai.com).

These are **different platforms**:
- **OpenAI** → Provides GPT models (gpt-4, gpt-3.5-turbo, etc.)
- **OpenCode** → Appears to be a separate AI platform that may use Kimi (Moonshot AI) models

OpenClaw is trying to call `api.openai.com` with an OpenCode key, which fails with 401.

## Key Vault Contents

| Secret Name | Provider | Valid for OpenClaw? |
|------------|----------|---------------------|
| `OpenAI-API-KEY` | OpenCode.ai (NOT OpenAI) | ❌ Invalid for OpenAI endpoint |
| `OpenClawGatewayToken` | OpenClaw Gateway | ✅ Working |
| `Telegram-Bot-Token` | Telegram Bot API | ✅ Working |
| `vmpass` | Unknown | ❓ Not an AI key |

## What We Need

To fix the "[assistant turn failed before producing content]" error, we need a **valid API key** for one of these providers:

### Option 1: Real OpenAI API Key (Recommended)
Get from: https://platform.openai.com/account/api-keys

### Option 2: OpenCode/Kimi Custom Endpoint
If OpenCode provides an OpenAI-compatible API endpoint, we can configure OpenClaw to use it:
- Base URL: (provided by OpenCode)
- Model: kimi-2.6 or equivalent
- Key: the OpenCode key we already have

### Option 3: Other Supported Providers
OpenClaw also supports:
- **Anthropic** (Claude) → `ANTHROPIC_API_KEY`
- **Google** (Gemini) → `GEMINI_API_KEY`
- **OpenRouter** → `OPENROUTER_API_KEY`
- **Local models** (Ollama, LM Studio)

## Next Steps

Please provide **one** of the following:

1. **A real OpenAI API key** from https://platform.openai.com
2. **The OpenCode API base URL** if they offer an OpenAI-compatible endpoint
3. **API keys for another provider** (Anthropic, Google, OpenRouter)

Once you provide the correct key, I will:
- Save it to Azure Key Vault
- Set it as a Fly.io secret
- Configure the default model in OpenClaw
- Document the setup

## Temporary Workaround

Until a valid AI provider key is configured, the OpenClaw gateway will:
- ✅ Accept connections (Control UI, Telegram)
- ✅ Authenticate users
- ❌ Fail to generate AI responses

## References

- [OpenClaw Models Docs](https://docs.openclaw.ai/concepts/models)
- [OpenClaw Model Failover](https://docs.openclaw.ai/concepts/model-failover)
- [OpenAI API Keys](https://platform.openai.com/account/api-keys)
