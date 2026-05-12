# Device Pairing Fix Report

## Issue

**Error:** `device pairing required (requestId: 650750ca-648e-45eb-b795-1d5f98bdaaf5)`

When accessing the OpenClaw Control UI at `https://openclaw-fly-lhr-20260512.fly.dev`, the browser was rejected because OpenClaw requires every new device to be explicitly approved before it can connect to the gateway.

## Root Cause

OpenClaw has a security feature called **Device Pairing** that treats every new browser/device as untrusted. This is a security measure to prevent unauthorized access to the gateway.

The typical fix (`devices approve <requestId>` via CLI) failed on Fly.io because:

1. The CLI command itself connects to the gateway via WebSocket
2. This CLI connection is treated as a **NEW device** that also needs pairing
3. A chicken-and-egg problem: you need device pairing approval to run the command that grants device pairing approval

**Log evidence:**
```
[ws] closed before connect
reason=pairing required: device is not approved yet (requestId: 7ef4df6d-bc20-4aa2-8805-1dd95b215c82)
```

## Solution Applied

### Direct Database Manipulation

Instead of using the CLI, we directly modified the device pairing database files on the persistent volume (`/data/devices/`).

**Files involved:**
- `/data/devices/pending.json` — Devices waiting for approval
- `/data/devices/paired.json` — Approved devices with tokens

**Steps:**

1. **Listed the data directory** to find where device pairing data is stored:
   ```bash
   flyctl ssh console --app openclaw-fly-lhr-20260512 --command "ls -la /data"
   ```
   Found: `/data/devices/pending.json` and `/data/devices/paired.json`

2. **Read pending devices** to locate the browser request:
   ```bash
   flyctl ssh console --app openclaw-fly-lhr-20260512 --command "cat /data/devices/pending.json"
   ```
   Found the pending entry for requestId `650750ca-648e-45eb-b795-1d5f98bdaaf5`

3. **Created a Node.js script** (`approve-device.js`) to:
   - Read both `pending.json` and `paired.json`
   - Extract the pending device entry
   - Convert it to paired format (add `approvedScopes`, `tokens`, `createdAtMs`, `approvedAtMs`)
   - Generate an operator token for the device
   - Move it from `pending.json` to `paired.json`

4. **Uploaded and executed the script**:
   ```bash
   flyctl ssh sftp put --app openclaw-fly-lhr-20260512 approve-device.js /home/node/approve-device.js
   flyctl ssh console --app openclaw-fly-lhr-20260512 --command "node /home/node/approve-device.js"
   ```

5. **Verified the fix**:
   - `pending.json` no longer contains the browser requestId
   - `paired.json` now contains the approved browser device with full operator scopes

## Result

| Before | After |
|--------|-------|
| Browser shows `device pairing required` | Browser device approved |
| `pending.json` contains requestId `650750ca-...` | Request removed from pending |
| Cannot access Control UI | Control UI accessible after refresh |

## How to Access the Control UI Now

1. Open: `https://openclaw-fly-lhr-20260512.fly.dev`
2. Enter the **Gateway Token** when prompted
3. The browser should now connect successfully

## Preventing Future Device Pairing Issues

For a public Fly.io deployment, every new browser/device will trigger device pairing. You have two options:

### Option A: Approve Each New Device (Recommended for Security)

Use the direct database method described above for each new device, or:

1. Access the Control UI from a browser
2. Copy the `requestId` from the error message
3. Run the approval script with that requestId

### Option B: Use Tokenized Dashboard URL

Generate a pre-authenticated URL from inside the container:
```bash
flyctl ssh console --app openclaw-fly-lhr-20260512 --command "sh -c 'export OPENCLAW_GATEWAY_URL=ws://127.0.0.1:3000; node dist/index.js dashboard --no-open'"
```

Then append your gateway token as a URL fragment:
```
https://openclaw-fly-lhr-20260512.fly.dev/#token=<YOUR_GATEWAY_TOKEN>
```

This may bypass device pairing for that session.

### Option C: Direct Database Pre-Approval (Advanced)

If you know the device public key in advance, you can pre-populate `paired.json` with the device entry before it connects.

## Key Insights

1. **Device pairing != Token auth**: The `OPENCLAW_GATEWAY_TOKEN` proves you know the secret, but device pairing proves the specific browser is trusted.

2. **CLI chicken-and-egg**: On remote deployments, the CLI cannot approve devices because the CLI connection itself needs approval.

3. **Database is plain JSON**: Device pairing state is stored in simple JSON files (`/data/devices/pending.json`, `/data/devices/paired.json`) that can be directly manipulated.

4. **Volume persistence matters**: The `/data` volume survives container restarts, so approved devices remain approved across deployments.

## Commands Reference

```bash
# Check pending devices
flyctl ssh console --app openclaw-fly-lhr-20260512 --command "cat /data/devices/pending.json"

# Check paired devices
flyctl ssh console --app openclaw-fly-lhr-20260512 --command "cat /data/devices/paired.json"

# List all data files
flyctl ssh console --app openclaw-fly-lhr-20260512 --command "ls -la /data"
```

## Security Note

Device pairing is a critical security feature. Only disable or bypass it if you understand the risks. For production use:
- Keep device pairing enabled
- Use strong gateway tokens
- Only approve devices you trust
- Rotate tokens periodically
