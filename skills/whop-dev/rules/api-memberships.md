---
name: api-memberships
description: Working with memberships for CRM and user management
metadata:
  tags: memberships, crm, users, subscriptions
---

## Required Permissions

- `member:basic:read` - Read membership details
- `member:email:read` - Read member email addresses

## List Memberships

```typescript
import { whopsdk } from "@/lib/whop-sdk";

// List all memberships with automatic pagination
for await (const membership of whopsdk.memberships.list()) {
  console.log(membership.id, membership.user.email);
}
```

## Filter by Product

```typescript
const memberships = await whopsdk.memberships.list({
  product_id: "prod_xxxxx",
});

for await (const membership of memberships) {
  console.log(membership.user.username);
}
```

## Filter by Status

```typescript
// Active memberships only
const active = await whopsdk.memberships.list({
  status: "active",
});

// Possible statuses: active, trialing, past_due, canceled, expired
```

## Membership Object

```typescript
{
  id: "mem_xxxxxxxxxxxxxx",
  status: "active", // trialing, active, past_due, canceled, expired
  created_at: "2023-12-01T05:00:00.401Z",
  updated_at: "2023-12-01T05:00:00.401Z",
  manage_url: "https://whop.com/...",
  member: {
    id: "mber_xxxxxxxxxxxxx"
  },
  user: {
    id: "user_xxxxxxxxxxxxx",
    username: "johndoe42",
    name: "John Doe",
    email: "john.doe@example.com"
  },
  renewal_period_start: "2023-12-01T05:00:00.401Z",
  renewal_period_end: "2024-01-01T05:00:00.401Z",
  cancel_at_period_end: false,
  cancel_option: null,
  cancellation_reason: null,
  canceled_at: null,
  currency: "usd",
  company: {
    id: "biz_xxxxxxxxxxxxxx",
    title: "My Company"
  },
  plan: {
    id: "plan_xxxxxxxxxxxxx"
  },
  product: {
    id: "prod_xxxxxxxxxxxxx",
    title: "Premium Membership"
  },
  license_key: "XXXX-XXXX-XXXX-XXXX",
  metadata: {},
  payment_collection_paused: false
}
```

## Get Single Membership

```typescript
const membership = await whopsdk.memberships.retrieve("mem_xxxxx");
```

## Update Membership Metadata

```typescript
const updated = await whopsdk.memberships.update("mem_xxxxx", {
  metadata: {
    custom_field: "value",
    tier: "premium",
  },
});
```

## Cancel Membership

```typescript
await whopsdk.memberships.cancel("mem_xxxxx", {
  cancel_at_period_end: true, // Cancel at end of billing period
  // cancel_at_period_end: false, // Cancel immediately
});
```

## API Route: List Members

```typescript
// GET /api/admin/members
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";

export async function GET(request: NextRequest) {
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  const { searchParams } = new URL(request.url);
  const productId = searchParams.get("product_id");
  
  // Verify admin access
  const companyId = process.env.WHOP_COMPANY_ID!;
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    companyId,
    { id: userId }
  );
  
  if (!has_access || access_level !== "admin") {
    return NextResponse.json({ error: "Admin required" }, { status: 403 });
  }
  
  // Fetch memberships
  const members = [];
  for await (const membership of whopsdk.memberships.list({
    product_id: productId || undefined,
  })) {
    members.push({
      id: membership.id,
      user: membership.user,
      status: membership.status,
      product: membership.product,
      created_at: membership.created_at,
    });
  }
  
  return NextResponse.json({ members });
}
```

## Members vs Memberships

| Resource | Description |
|----------|-------------|
| **Member** | A user's connection to a company (one per company) |
| **Membership** | A user's access to a product (can have multiple) |

A user can have multiple memberships (to different products) but only one member record per company.

## Pagination

The SDK handles pagination automatically with async iteration:

```typescript
// Fetches pages as needed
for await (const membership of whopsdk.memberships.list()) {
  // Process each membership
}
```

For manual pagination:

```typescript
const page = await whopsdk.memberships.list({
  limit: 50,
  after: "cursor_from_previous_page",
});

console.log(page.data); // Array of memberships
console.log(page.page_info.has_next_page);
console.log(page.page_info.end_cursor);
```

## Webhook Events

Listen for membership changes:

- `membership.activated` - User gained access
- `membership.deactivated` - User lost access (canceled, expired, payment failed)

```typescript
if (webhookData.type === "membership.activated") {
  const membership = webhookData.data;
  // Grant access, send welcome email, etc.
}

if (webhookData.type === "membership.deactivated") {
  const membership = webhookData.data;
  // Revoke access, send win-back email, etc.
}
```

## Reference

- [List Memberships API](https://docs.whop.com/api-reference/memberships/list-memberships)
- [Retrieve Membership API](https://docs.whop.com/api-reference/memberships/retrieve-membership)
- [Membership Activated Hook](https://docs.whop.com/api-reference/memberships/membership-activated)
