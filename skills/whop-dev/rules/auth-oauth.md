---
name: auth-oauth
description: OAuth authentication for external integrations
metadata:
  tags: oauth, authentication, external
---

Use OAuth when your app needs to authenticate users outside the Whop iframe.

## When to Use OAuth

- Building a standalone app (not in iframe)
- External service integrations
- Mobile apps using WebView
- CLI tools

## OAuth Flow

### 1. Redirect to Authorization

```typescript
// GET /api/auth/login
import { redirect } from "next/navigation";

export async function GET() {
  const authUrl = new URL("https://whop.com/oauth/authorize");
  
  authUrl.searchParams.set("client_id", process.env.WHOP_CLIENT_ID!);
  authUrl.searchParams.set("redirect_uri", process.env.WHOP_REDIRECT_URI!);
  authUrl.searchParams.set("response_type", "code");
  authUrl.searchParams.set("scope", "openid profile");
  
  return redirect(authUrl.toString());
}
```

### 2. Handle Callback

```typescript
// GET /api/auth/callback
import { NextRequest, NextResponse } from "next/server";
import { redirect } from "next/navigation";

export async function GET(request: NextRequest) {
  const code = request.nextUrl.searchParams.get("code");
  
  if (!code) {
    return NextResponse.json({ error: "No code provided" }, { status: 400 });
  }
  
  // Exchange code for tokens
  const tokenResponse = await fetch("https://api.whop.com/oauth/token", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      grant_type: "authorization_code",
      client_id: process.env.WHOP_CLIENT_ID,
      client_secret: process.env.WHOP_CLIENT_SECRET,
      code,
      redirect_uri: process.env.WHOP_REDIRECT_URI,
    }),
  });
  
  const tokens = await tokenResponse.json();
  
  // Store tokens securely (e.g., encrypted cookie, database)
  // tokens.access_token, tokens.refresh_token
  
  return redirect("/dashboard");
}
```

### 3. Use Access Token

```typescript
import { Whop } from "@whop/sdk";

const client = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID!,
  // Use OAuth access token instead of API key
  accessToken: userAccessToken,
});

// Now API calls are made on behalf of the user
const user = await client.users.me();
```

## Refresh Tokens

```typescript
async function refreshAccessToken(refreshToken: string) {
  const response = await fetch("https://api.whop.com/oauth/token", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      grant_type: "refresh_token",
      client_id: process.env.WHOP_CLIENT_ID,
      client_secret: process.env.WHOP_CLIENT_SECRET,
      refresh_token: refreshToken,
    }),
  });
  
  return response.json();
}
```

## Iframe vs OAuth

| Feature | Iframe (x-whop-user-token) | OAuth |
|---------|---------------------------|-------|
| Use case | Apps inside Whop | External apps |
| Setup | Automatic | Manual flow |
| Token refresh | Handled by Whop | Manual |
| User consent | Via app install | Each auth |

**Prefer iframe authentication** when building Whop apps. Use OAuth only for external integrations.

## Reference

- [OAuth Guide](https://docs.whop.com/developer/guides/oauth)
