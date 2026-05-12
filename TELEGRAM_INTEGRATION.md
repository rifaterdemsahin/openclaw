# Telegram Bot Integration Report

## Bot Details

| Property | Value |
|----------|-------|
| **Bot Name** | fly_open_claw_bot |
| **Bot URL** | https://t.me/fly_open_claw_bot |
| **Platform** | Telegram |
| **Status** | ✅ Connected & Running |

## Implementation Steps

### Step 1: Create Telegram Bot

1. Open Telegram and message [@BotFather](https://t.me/botfather)
2. Send `/newbot` command
3. Name the bot: `fly_open_claw_bot`
4. BotFather provides the HTTP API token
5. Save the token securely

### Step 2: Save Token Securely

**Azure Key Vault:**
```powershell
$SecureString = ConvertTo-SecureString -String "<TOKEN>" -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName openshifthelper -Name "Telegram-Bot-Token" -SecretValue $SecureString
```

**Vault Details:**
| Property | Value |
|----------|-------|
| Vault Name | `openshifthelper` |
| Secret Name | `Telegram-Bot-Token` |
| Status | ✅ Saved |

### Step 3: Configure Fly.io Secret

```bash
flyctl secrets set TELEGRAM_BOT_TOKEN=<TOKEN> --app openclaw-fly-lhr-20260512
```

This triggers a rolling deployment of the app with the new secret.

### Step 4: Enable Telegram Channel

The OpenClaw config (`/data/openclaw.json`) already includes:
```json
{
  "channels": {
    "telegram": {
      "enabled": true
    }
  }
}
```

When the gateway starts, it:
1. Detects the `TELEGRAM_BOT_TOKEN` environment variable
2. Automatically enables the Telegram channel
3. Connects to the Telegram Bot API
4. Starts listening for messages

## Verification

### Check Logs

```bash
flyctl logs --app openclaw-fly-lhr-20260512 --no-tail
```

Expected output:
```
Telegram configured, enabled automatically.
[telegram] [default] starting provider (@fly_open_claw_bot)
```

### Check Environment Variable

```bash
flyctl ssh console --app openclaw-fly-lhr-20260512 --command "sh -c 'env | grep TELEGRAM'"
```

Expected output:
```
TELEGRAM_BOT_TOKEN=8785937321:...<redacted>
```

## How to Use

1. **Open Telegram** and search for `@fly_open_claw_bot`
2. **Start a conversation** with the bot
3. **Send a message** — the OpenClaw AI assistant will reply

The bot acts as another channel into your OpenClaw gateway, just like the Control UI.

## Security Notes

- **Never share the bot token** — anyone with the token can control your bot
- **Store in Key Vault** — the real token is in Azure Key Vault, not in code
- **Fly secrets are encrypted** — the token is encrypted at rest and in transit
- **Rotate if leaked** — if the token is accidentally exposed, revoke it via @BotFather and generate a new one

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Bot not responding | Gateway not running | Check `flyctl status --app openclaw-fly-lhr-20260512` |
| Bot not responding | Token not set | `flyctl secrets set TELEGRAM_BOT_TOKEN=... --app ...` |
| Bot not responding | Channel disabled | Ensure `channels.telegram.enabled: true` in `/data/openclaw.json` |
| Bot not responding | Token revoked | Generate new token from @BotFather and update secret |
| 401 Unauthorized | Wrong token | Verify token matches what @BotFather provided |

## Adding More Channels

You can add other messaging channels using the same pattern:

### Discord
```bash
flyctl secrets set DISCORD_BOT_TOKEN=<token> --app openclaw-fly-lhr-20260512
```

### Slack
```bash
flyctl secrets set SLACK_BOT_TOKEN=xoxb-... --app openclaw-fly-lhr-20260512
flyctl secrets set SLACK_APP_TOKEN=xapp-... --app openclaw-fly-lhr-20260512
```

### WhatsApp
Requires QR code scan during setup. See [OpenClaw WhatsApp docs](https://docs.openclaw.ai/channels/whatsapp).

## References

- [OpenClaw Channels Docs](https://docs.openclaw.ai/channels)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [OpenClaw Telegram Channel](https://docs.openclaw.ai/channels/telegram)
- [Fly.io Secrets](https://fly.io/docs/apps/secrets/)
