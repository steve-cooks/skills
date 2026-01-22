---
name: dev-sandbox
description: Testing Whop apps in sandbox mode
metadata:
  tags: development, sandbox, testing
---

Test payments and webhooks without real money using sandbox mode.

## Enable Sandbox Mode

```typescript
// lib/whop-sdk.ts
import { Whop } from "@whop/sdk";

export const whopsdk = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID,
  apiKey: process.env.WHOP_API_KEY,
  webhookKey: btoa(process.env.WHOP_WEBHOOK_SECRET || ""),
  // Enable sandbox for testing
  sandbox: process.env.NODE_ENV === "development",
});
```

## Test Cards

Use these test card numbers in sandbox:

| Card Number | Result |
|-------------|--------|
| `4242 4242 4242 4242` | Successful payment |
| `4000 0000 0000 0002` | Card declined |
| `4000 0000 0000 9995` | Insufficient funds |

Use any future expiry date and any 3-digit CVC.

## Sandbox Webhooks

Sandbox webhooks work identically to production. Test your webhook handlers with simulated events.

```typescript
import { waitUntil } from "@vercel/functions";
import type { Payment } from "@whop/sdk/resources.js";
import { whopsdk } from "@/lib/whop-sdk";

export async function POST(request: NextRequest) {
  const requestBodyText = await request.text();
  const headers = Object.fromEntries(request.headers);
  
  // Webhook verification works same in sandbox
  const webhookData = whopsdk.webhooks.unwrap(requestBodyText, { headers });
  
  console.log("Sandbox event:", webhookData.type);
  
  if (webhookData.type === "payment.succeeded") {
    waitUntil(handlePayment(webhookData.data));
  }
  
  return new Response("OK", { status: 200 });
}
```

## Environment Setup

```bash
# .env.development
WHOP_API_KEY=your_sandbox_api_key
WHOP_WEBHOOK_SECRET=whsec_sandbox_xxxxx
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx

# .env.production  
WHOP_API_KEY=your_production_api_key
WHOP_WEBHOOK_SECRET=whsec_production_xxxxx
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx
```

## Testing Locally

1. Use ngrok or Cloudflare Tunnel to expose localhost
2. Configure webhook URL in Whop dashboard
3. Trigger test events through the dashboard

```bash
# Start ngrok
ngrok http 3000

# Use the ngrok URL for webhook endpoint
# https://abc123.ngrok.io/api/webhooks
```

## Reference

- [Test in Sandbox Guide](https://docs.whop.com/developer/guides/sandbox)
