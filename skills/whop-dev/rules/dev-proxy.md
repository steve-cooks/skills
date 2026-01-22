---
name: dev-proxy
description: Using the Whop development proxy for local testing
metadata:
  tags: development, proxy, local, testing
---

## Why Use the Proxy

Whop apps run inside an iframe. The development proxy:
- Simulates the Whop iframe environment
- Injects authentication tokens into requests
- Handles theme/appearance syncing

## Installation

```bash
pnpm add -D @whop-apps/dev-proxy
```

## Package.json Scripts

```json
{
  "scripts": {
    "dev": "whop-proxy --command 'next dev --turbopack'",
    "build": "next build",
    "start": "next start"
  }
}
```

## Running Development

**Correct:**

```bash
pnpm dev
# or
npm run dev
```

**Incorrect:**

```bash
next dev          # No auth tokens!
pnpm start        # Production mode, no proxy
```

## Accessing Your App

1. Run `pnpm dev`
2. Go to your app in Whop Developer Dashboard
3. Click the settings icon (top-right)
4. Select "localhost" 
5. App loads through Whop's iframe

Never access `localhost:3000` directly - authentication won't work.

## Common Issues

### "Whop user token not found"

**Cause:** Accessing app directly, not through Whop iframe
**Fix:** Access via Whop dashboard with "localhost" selected

### Proxy not starting

**Cause:** Port conflict or missing dependency
**Fix:**
```bash
pnpm add -D @whop-apps/dev-proxy
# Kill any process on port 3000
lsof -i :3000
kill -9 <PID>
```

### Theme not syncing

**Cause:** Accessing directly instead of through iframe
**Fix:** Always use Whop's iframe for testing

## Environment Setup

```bash
# .env.local
WHOP_API_KEY=your_api_key
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx
```

## Production vs Development

| Environment | Command | Auth Source |
|-------------|---------|-------------|
| Development | `pnpm dev` | Proxy injects tokens |
| Production | Vercel deploy | Whop iframe injects tokens |

Never hardcode auth tokens or bypass authentication in production.

## Reference

See [Whop Development Proxy Guide](https://docs.whop.com/developer/guides/dev-proxy)
