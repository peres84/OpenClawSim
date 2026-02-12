# OpenClaw Setup & Troubleshooting Guide

This guide documents the steps to resolve common authentication and connection issues when setting up OpenClaw Gateway and Control UI.

## Prerequisites

- Node.js 22 or newer
- OpenClaw installed (`curl -fsSL https://openclaw.ai/install.sh | bash`)

## Initial Setup

1. **Run the onboarding wizard:**
   ```bash
   openclaw onboard --install-daemon
   ```

2. **Configure gateway mode:**
   ```bash
   openclaw config set gateway.mode local
   ```

3. **Start the gateway:**
   ```bash
   openclaw gateway --port 18789
   ```
   
   Or run in background:
   ```bash
   nohup openclaw gateway --port 18789 > /tmp/openclaw.log 2>&1 &
   ```

## Common Issues & Solutions

### Issue 1: "disconnected (1008): unauthorized: gateway token missing"

**Symptoms:**
- Control UI shows: "disconnected (1008): unauthorized: gateway token missing (open the dashboard URL and paste the token in Control UI settings)"
- Cannot connect to the gateway

**Root Cause:**
The browser's Control UI doesn't have the authentication token stored in localStorage.

**Solution:**

1. **Get your gateway token:**
   ```bash
   cat ~/.openclaw/openclaw.json | grep -A3 '"auth"'
   ```
   
   Look for the `token` value under `gateway.auth.token`.

2. **Add token to browser localStorage:**
   
   Open Browser DevTools (F12) → Console tab, then run:
   ```javascript
   let settings = JSON.parse(localStorage.getItem('openclaw.control.settings.v1'));
   settings.token = 'YOUR_TOKEN_HERE';  // Replace with actual token
   localStorage.setItem('openclaw.control.settings.v1', JSON.stringify(settings));
   location.reload();
   ```

   **Or manually:**
   - F12 → Application tab → Local Storage → your URL
   - Find `openclaw.control.settings.v1`
   - Double-click the value and edit the `"token":""` field to include your token
   - Refresh the page

### Issue 2: "disconnected (1008): pairing required"

**Symptoms:**
- After fixing token issue, you see: "disconnected (1008): pairing required"
- Gateway logs show: "closed before connect ... code=1008 reason=pairing required"

**Root Cause:**
When connecting from a remote location (like GitHub Codespaces forwarded ports, LAN, or Tailnet), OpenClaw requires one-time device pairing approval for security. Local connections (127.0.0.1) are auto-approved, but proxied/forwarded connections need manual approval.

**Solution:**

1. **List pending device pairing requests:**
   ```bash
   openclaw devices list
   ```
   
   You'll see output like:
   ```
   Pending (1)
   ┌────────────────────────────┬────────┬────────┐
   │ Request                    │ Device │ Role   │
   ├────────────────────────────┼────────┼────────┤
   │ 21c97109-f017-45d7-b348-   │ b0e397 │ operat │
   │ 5c42f4192678               │ ...    │ or     │
   └────────────────────────────┴────────┴────────┘
   ```

2. **Approve the device:**
   ```bash
   openclaw devices approve <REQUEST_ID>
   ```
   
   Example:
   ```bash
   openclaw devices approve 21c97109-f017-45d7-b348-5c42f4192678
   ```

3. **Refresh the Control UI browser page**

   The device is now paired and won't require re-approval unless you:
   - Clear browser data
   - Switch browsers or browser profiles
   - Revoke the device manually

### Issue 3: "Proxy headers detected from untrusted address"

**Symptoms:**
- Gateway logs show: "Proxy headers detected from untrusted address. Connection will not be treated as local."
- Connections are treated as remote even when coming through localhost

**Root Cause:**
Gateway receives proxy headers (like X-Forwarded-For) but doesn't trust the proxy source.

**Solution:**

Add trusted proxies to your configuration:

```bash
# Edit ~/.openclaw/openclaw.json
```

Add `trustedProxies` to the gateway section:
```json
{
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "YOUR_TOKEN"
    },
    "trustedProxies": ["127.0.0.1", "::1"]
  }
}
```

Then restart the gateway:
```bash
pkill -f openclaw
openclaw gateway --port 18789
```

## Configuration File Location

Your OpenClaw configuration is stored at:
```
~/.openclaw/openclaw.json
```

## Gateway Token Location

The gateway authentication token can be found in the config file:
```bash
cat ~/.openclaw/openclaw.json
```

Look for:
```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "YOUR_TOKEN_HERE"
    }
  }
}
```

## Useful Commands

### Check Gateway Status
```bash
openclaw gateway status
```

### View Gateway Logs
```bash
tail -f /tmp/openclaw/openclaw-2026-02-12.log
```

Or if running in background:
```bash
tail -f /tmp/openclaw.log
```

### List All Devices (Pending & Paired)
```bash
openclaw devices list
```

### Revoke a Device
```bash
openclaw devices revoke --device <DEVICE_ID> --role <ROLE>
```

### Open Dashboard
```bash
openclaw dashboard
```

### Run Security Audit
```bash
openclaw security audit --deep
openclaw security audit --fix
```

## Access URLs

- **Dashboard:** `http://127.0.0.1:18789/`
- **Control UI:** `http://127.0.0.1:18789/` (same as dashboard)
- **Gateway WebSocket:** `ws://127.0.0.1:18789`

## Security Notes

1. **Keep your gateway token secret** - it provides full access to your gateway
2. **Review pending devices** before approving them
3. **Use pairing for remote connections** - don't disable it unless you understand the risks
4. **Run security audits regularly:**
   ```bash
   openclaw security audit --deep
   ```

## Additional Resources

- Official Docs: https://docs.openclaw.ai/
- Control UI Guide: https://docs.openclaw.ai/web/control-ui
- Pairing Documentation: https://docs.openclaw.ai/channels/pairing
- Troubleshooting: https://docs.openclaw.ai/gateway/troubleshooting

## Summary of Steps

1. ✅ Install OpenClaw
2. ✅ Run onboarding wizard
3. ✅ Set gateway mode to local
4. ✅ Start gateway on port 18789
5. ✅ Get gateway token from config file
6. ✅ Add token to browser localStorage
7. ✅ Approve device pairing request
8. ✅ Refresh browser - Control UI should connect!

---

**Installation completed:** February 12, 2026
**OpenClaw Version:** 2026.2.9
**Gateway Token:** Check `~/.openclaw/openclaw.json`
