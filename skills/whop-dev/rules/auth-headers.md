---
name: auth-headers
description: Extracting Whop tokens from Next.js headers correctly
metadata:
  tags: authentication, headers, nextjs, app-router
---

Whop injects authentication tokens into request headers when your app runs in the iframe.

## Server Components

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

export default async function Page() {
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);
  // ...
}
```

## API Route Handlers

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  return NextResponse.json({ userId });
}
```

## Parallel Header + Body Fetch

For POST/PUT routes, fetch headers and body in parallel:

```typescript
export async function POST(request: NextRequest) {
  // Parallel fetch - headers() and request.json() are independent
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  // Now use body safely with verified userId
  return NextResponse.json({ success: true });
}
```

## Error Handling

Always handle token verification errors:

```typescript
export async function GET(request: NextRequest) {
  try {
    const headersList = await headers();
    const { userId } = await whopsdk.verifyUserToken(headersList);
    
    // Proceed with authenticated request
    return NextResponse.json({ data: "protected" });
  } catch (error) {
    // Token invalid, expired, or missing
    return NextResponse.json(
      { error: "Unauthorized" },
      { status: 401 }
    );
  }
}
```

## Using dontThrow Option

```typescript
const result = await whopsdk.verifyUserToken(headersList, { dontThrow: true });

if (!result) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}

const { userId } = result;
```

## Webhook Headers

For webhooks, convert headers to a plain object:

```typescript
export async function POST(request: NextRequest) {
  const requestBodyText = await request.text();
  const headers = Object.fromEntries(request.headers);
  
  const webhookData = whopsdk.webhooks.unwrap(requestBodyText, { headers });
  // ...
}
```

## Important

- `headers()` is async in Next.js 15+ App Router
- The token is automatically injected by Whop's iframe via `x-whop-user-token` header
- Never pass tokens via query parameters or request body
- Always use `await headers()` before `verifyUserToken()`

## Reference

- [Authentication Guide](https://docs.whop.com/developer/guides/authentication)
