---
name: auth-verify-token
description: Always verify Whop user tokens server-side before any operation
metadata:
  tags: authentication, security, token, server
---

Every protected route MUST verify the user token using `whopsdk.verifyUserToken()`.

**Incorrect (trusting client-provided userId):**

```typescript
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const userId = searchParams.get("userId"); // DANGEROUS: Attacker-controlled!
  
  // Fetching data with untrusted userId
  const data = await db.secrets.findMany({ where: { user_id: userId } });
  return NextResponse.json(data);
}
```

**Correct (verifying token first):**

```typescript
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";

export async function GET(request: NextRequest) {
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  // userId is now trusted - comes from verified token
  const data = await db.secrets.findMany({ where: { user_id: userId } });
  return NextResponse.json(data);
}
```

The `verifyUserToken()` call will throw if the token is invalid, expired, or missing.

## Options

```typescript
// Default: throws on invalid token
const { userId } = await whopsdk.verifyUserToken(headersList);

// With dontThrow: returns null instead of throwing
const result = await whopsdk.verifyUserToken(headersList, { dontThrow: true });
if (!result) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
const { userId } = result;
```

## In Server Components

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

export default async function ExperiencePage({ 
  params 
}: { 
  params: Promise<{ experienceId: string }> 
}) {
  const headersList = await headers();
  const { experienceId } = await params;
  
  // Verify user first
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  // Then check access
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    experienceId, 
    { id: userId }
  );
  
  if (!has_access) {
    return <div>Access denied</div>;
  }
  
  // Safe to render protected content
  return <ProtectedContent userId={userId} />;
}
```

## In API Routes

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  try {
    const headersList = await headers();
    const { userId } = await whopsdk.verifyUserToken(headersList);
    
    // Proceed with authenticated request
    return NextResponse.json({ userId, data: "protected" });
  } catch (error) {
    // Token invalid, expired, or missing
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }
}
```

## POST Routes with Body

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

## Important

- `headers()` is async in Next.js 15+ App Router
- The token is automatically injected by Whop's iframe
- Never pass tokens via query parameters or request body
- Always use `await headers()` before `verifyUserToken()`
- The token comes from the `x-whop-user-token` header

## Reference

- [Authentication Guide](https://docs.whop.com/developer/guides/authentication)
