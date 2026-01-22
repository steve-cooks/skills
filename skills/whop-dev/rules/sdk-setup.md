---
name: sdk-setup
description: Initialize Whop SDK correctly for server and client operations
metadata:
  tags: sdk, setup, configuration, server
---

## Installation

```bash
pnpm add @whop/sdk @whop/react
```

## Server-Side SDK Setup

Create a centralized SDK instance:

```typescript
// lib/whop-sdk.ts
import { Whop } from "@whop/sdk";

export const whopsdk = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID,
  apiKey: process.env.WHOP_API_KEY,
});
```

## With Webhook Support

```typescript
// lib/whop-sdk.ts
import { Whop } from "@whop/sdk";

export const whopsdk = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID,
  apiKey: process.env.WHOP_API_KEY,
  // Webhook key must be base64 encoded
  webhookKey: btoa(process.env.WHOP_WEBHOOK_SECRET || ""),
});
```

## Environment Variables

```bash
# Required (NEVER expose to client)
WHOP_API_KEY=your_api_key              # Server-side API key
WHOP_WEBHOOK_SECRET=whsec_xxxxx        # Webhook signing secret

# Required for apps
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx      # App identifier
```

## Common SDK Methods

### Authentication

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

// Verify user from request headers
const { userId } = await whopsdk.verifyUserToken(await headers());
// Throws on validation failure
// Pass { dontThrow: true } to return null instead
```

### Check Access

```typescript
// Check if user has access to a resource
const response = await whopsdk.users.checkAccess(
  "exp_xxxxx", // resource_id: exp_, biz_, or prod_ prefix
  { id: userId }
);

// Response: { has_access: true, access_level: "customer" }
```

### Payments

```typescript
// List payments
const payments = await whopsdk.payments.list({ 
  company_id: "biz_xxxxx" 
});

// Create checkout configuration
const checkout = await whopsdk.checkoutConfigurations.create({
  plan: { /* plan details */ },
});
```

### Memberships (CRM)

```typescript
// List all memberships with pagination
for await (const membership of whopsdk.memberships.list()) {
  console.log(membership.id, membership.user);
}

// Filter by product
const memberships = await whopsdk.memberships.list({
  product_id: "prod_xxxxx",
});
```

### Notifications

```typescript
await whopsdk.notifications.create({
  experience_id: "exp_xxxxx",
  title: "New message",
  content: "You have a notification",
});
```

### Transfers (Payouts)

```typescript
await whopsdk.transfers.create({
  amount: 10.0, // Dollars (not cents)
  currency: "usd",
  origin_id: "biz_xxxxx",
  destination_id: "user_xxxxx",
});
```

## Client-Side SDK (Iframe)

For client-side interactions, use `@whop/react`:

```typescript
"use client";

import { useIframeSdk } from "@whop/react";

export function MyComponent() {
  const iframeSdk = useIframeSdk();
  
  // Open checkout
  iframeSdk.openCheckout({ checkoutConfigurationId: "cc_xxx" });
}
```

## Type Imports

Import types from the SDK:

```typescript
import type { Payment, Membership, User } from "@whop/sdk/resources.js";
```

## MCP Server (AI Assistance)

Whop provides an MCP server for AI-assisted development:
- Cursor: `https://mcp.whop.com/mcp`
- Claude: `https://mcp.whop.com/sse`

## Security Rules

- `WHOP_API_KEY` is SECRET - never expose to client or logs
- Never log full API keys - truncate if needed: `key.slice(0, 8) + "..."`
- SDK is server-side only - use in API routes and Server Components
- Get API key from [Whop Developer Dashboard](https://whop.com/dashboard/developer)

## Reference

- [API Getting Started](https://docs.whop.com/developer/api/getting-started)
- [Developer Getting Started](https://docs.whop.com/developer/getting-started)
