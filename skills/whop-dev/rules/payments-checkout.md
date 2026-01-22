---
name: payments-checkout
description: Creating checkout configurations for accepting payments
metadata:
  tags: payments, checkout, monetization, pricing
---

## Required Permissions

- `checkout_configuration:create`
- `plan:create`
- `access_pass:create`

## Creating a Checkout Configuration

Checkout configurations define what users pay for.

### With Inline Plan

```typescript
// POST /api/purchase
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const body = await request.json();
  
  const checkout = await whopsdk.checkoutConfigurations.create({
    plan: {
      // Create plan inline
      plan_type: "one_time",
      initial_price: body.priceDollars, // Price in dollars (e.g., 9.99)
      currency: "usd",
    },
    metadata: {
      userId,
      itemId: body.itemId,
    },
  });
  
  return NextResponse.json({ checkoutConfigurationId: checkout.id });
}
```

### With Existing Plan ID

```typescript
const checkout = await whopsdk.checkoutConfigurations.create({
  plan_id: "plan_xxxxx", // Use existing plan
  metadata: {
    userId,
    itemId: body.itemId,
  },
});
```

## Client-Side: Open Checkout

```typescript
"use client";

import { useIframeSdk } from "@whop/react";
import { useState } from "react";
import { Button } from "@whop/react";

export function PurchaseButton({ priceDollars, itemId }: Props) {
  const iframeSdk = useIframeSdk();
  const [loading, setLoading] = useState(false);
  
  const handlePurchase = async () => {
    setLoading(true);
    try {
      // Create checkout configuration
      const res = await fetch("/api/purchase", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ priceDollars, itemId }),
      });
      
      if (!res.ok) throw new Error("Failed to create checkout");
      
      const { checkoutConfigurationId } = await res.json();
      
      // Open Whop's checkout modal
      iframeSdk.openCheckout({ checkoutConfigurationId });
    } catch (error) {
      console.error("Purchase error:", error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <Button onClick={handlePurchase} disabled={loading}>
      {loading ? "Processing..." : `Pay $${priceDollars}`}
    </Button>
  );
}
```

## Plan Types

| Type | Use Case |
|------|----------|
| `one_time` | Single purchase |
| `recurring` | Subscriptions |
| `free` | Free access |

## Recurring Subscription

```typescript
const checkout = await whopsdk.checkoutConfigurations.create({
  plan: {
    plan_type: "recurring",
    initial_price: 9.99, // First payment
    renewal_price: 9.99, // Subsequent payments
    currency: "usd",
    billing_period: "monthly", // monthly, yearly
    trial_period_days: 7, // Optional trial
  },
});
```

## Additional Options

```typescript
const checkout = await whopsdk.checkoutConfigurations.create({
  plan: {
    plan_type: "one_time",
    initial_price: 29.99,
    currency: "usd",
  },
  // Optional settings
  affiliate_code: "PARTNER10", // Apply affiliate code
  redirect_url: "https://myapp.com/success", // Redirect after purchase
  purchase_url: "https://myapp.com/buy", // Custom purchase page
  metadata: {
    // Store custom data for webhooks
    orderId: "order_123",
    userId: "user_xxx",
  },
  // Control payment methods
  payment_method_configuration: {
    enabled: ["card", "paypal"],
    disabled: ["crypto"],
    include_platform_defaults: true,
  },
});
```

## Supported Currencies

USD, EUR, GBP, CAD, AUD, and 80+ more currencies including crypto (ETH, BTC).

## Metadata

Store custom data with checkout for webhook processing:

```typescript
metadata: {
  userId: "user_xxxxx",
  itemType: "question",
  itemId: "123",
  tier: "premium",
}
```

Access this metadata in your webhook handler.

## Reference

- [Accept Payments Guide](https://docs.whop.com/developer/guides/accept-payments)
- [Create Checkout Configuration API](https://docs.whop.com/api-reference/checkout-configurations/create-checkout-configuration)
