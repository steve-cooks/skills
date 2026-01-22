---
name: payments-webhooks
description: Handling Whop payment webhooks securely
impact: HIGH
impactDescription: Required for processing payments - missed webhooks = lost revenue
metadata:
  tags: payments, webhooks, events, security
---

## Payment Webhooks

Webhooks notify your app when payments complete.

## SDK Setup with Webhook Key

```typescript
// lib/whop-sdk.ts
import { Whop } from "@whop/sdk";

export const whopsdk = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID,
  apiKey: process.env.WHOP_API_KEY,
  // Webhook key MUST be base64 encoded
  webhookKey: btoa(process.env.WHOP_WEBHOOK_SECRET || ""),
});
```

## Webhook Endpoint

Create `app/api/webhooks/route.ts`:

```typescript
import { waitUntil } from "@vercel/functions";
import type { Payment } from "@whop/sdk/resources.js";
import type { NextRequest } from "next/server";
import { whopsdk } from "@/lib/whop-sdk";

export async function POST(request: NextRequest): Promise<Response> {
  try {
    // Get raw body and headers
    const requestBodyText = await request.text();
    const headers = Object.fromEntries(request.headers);
    
    // MUST validate signature before processing
    const webhookData = whopsdk.webhooks.unwrap(requestBodyText, { headers });
    
    // Handle specific events
    switch (webhookData.type) {
      case "payment.succeeded":
        waitUntil(handlePaymentSucceeded(webhookData.data));
        break;
      case "payment.failed":
        waitUntil(handlePaymentFailed(webhookData.data));
        break;
      case "membership.activated":
        waitUntil(handleMembershipActivated(webhookData.data));
        break;
      case "membership.deactivated":
        waitUntil(handleMembershipDeactivated(webhookData.data));
        break;
    }
    
    // MUST return 2xx quickly to avoid retries
    return new Response("OK", { status: 200 });
  } catch (error) {
    console.error("Webhook error:", error);
    return new Response("Invalid webhook", { status: 400 });
  }
}

async function handlePaymentSucceeded(payment: Payment) {
  console.log("[PAYMENT SUCCEEDED]", payment.id);
  
  const metadata = payment.checkout_configuration?.metadata;
  
  // Process the payment...
  await db.purchases.create({
    data: {
      payment_id: payment.id,
      user_id: metadata?.userId,
      amount: payment.amount,
      status: "completed",
    },
  });
}
```

## MUST: Use waitUntil

The `waitUntil` function from `@vercel/functions` allows you to:
- Return a response quickly (avoiding webhook timeouts)
- Continue processing in the background

```typescript
import { waitUntil } from "@vercel/functions";

// Return immediately, process async
waitUntil(handlePaymentSucceeded(webhookData.data));
return new Response("OK", { status: 200 });
```

## Webhook Event Types

| Event | When | Permission Required |
|-------|------|---------------------|
| `payment.succeeded` | Payment completed | `webhook_receive:payment` |
| `payment.failed` | Payment failed | `webhook_receive:payment` |
| `payment.pending` | Payment awaiting confirmation | `webhook_receive:payment` |
| `membership.activated` | User gained access | `webhook_receive:membership` |
| `membership.deactivated` | User lost access | `webhook_receive:membership` |
| `entry.created` | Waitlist entry created | `webhook_receive:entry` |
| `refund.created` | Refund issued | `webhook_receive:refund` |
| `dispute.created` | Chargeback opened | `webhook_receive:dispute` |

## Configure Webhooks

### Company Webhooks
For events on your own company:
1. Go to [Developer Dashboard](https://whop.com/dashboard/developer)
2. Click "Create Webhook"
3. Enter URL and select events

### App Webhooks
For events on all installed companies:
1. Go to your app in Developer Dashboard
2. Select "Webhooks" tab
3. Create webhook and select events
4. Request `webhook_receive:*` permissions

## MUST: Handle Idempotency

Webhooks may be delivered multiple times. ALWAYS handle duplicates:

```typescript
async function handlePaymentSucceeded(payment: Payment) {
  // Check if already processed
  const existing = await db.purchases.findFirst({
    where: { payment_id: payment.id },
  });
  
  if (existing) {
    console.log("Payment already processed:", payment.id);
    return; // Idempotent - safe to ignore duplicate
  }
  
  // Process the payment...
}
```

## Security Checklist

- [ ] ALWAYS use `webhooks.unwrap()` to verify signatures
- [ ] NEVER trust unverified webhook data
- [ ] Store `payment_id` to prevent duplicate processing
- [ ] Return 2xx quickly to avoid retries
- [ ] Use `waitUntil` for long-running operations

## Reference

- [Webhooks Guide](https://docs.whop.com/developer/guides/webhooks)
- [Payment Succeeded Hook](https://docs.whop.com/api-reference/payments/payment-succeeded)
- [Membership Activated Hook](https://docs.whop.com/api-reference/memberships/membership-activated)
