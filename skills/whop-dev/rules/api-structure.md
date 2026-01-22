---
name: api-structure
description: Route handler patterns for Whop API routes
metadata:
  tags: api, routes, handlers, patterns
---

## Standard API Route Pattern

Every protected API route follows this structure:

```typescript
// app/api/resource/route.ts
import { whopSdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  try {
    // 1. Authenticate
    const headersList = await headers();
    const { userId } = await whopSdk.verifyUserToken(headersList);
    
    // 2. Fetch data (with ownership check)
    const data = await fetchUserData(userId);
    
    // 3. Return response (sanitized)
    return NextResponse.json({ data });
  } catch (error) {
    // Log internally but don't expose details to client
    console.error("API error:", error instanceof Error ? error.message : "Unknown");
    
    // Return generic error - never leak internal details
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    // 1. Validate Content-Type
    const contentType = request.headers.get("content-type");
    if (!contentType?.includes("application/json")) {
      return NextResponse.json(
        { error: "Content-Type must be application/json" },
        { status: 415 }
      );
    }
    
    // 2. Parallel fetch of independent data
    const [headersList, body] = await Promise.all([
      headers(),
      request.json(),
    ]);
    
    // 3. Authenticate
    const { userId } = await whopSdk.verifyUserToken(headersList);
    
    // 4. Validate input (early exit, before expensive operations)
    if (!body.requiredField || typeof body.requiredField !== "string") {
      return NextResponse.json(
        { error: "Missing or invalid required field" },
        { status: 400 }
      );
    }
    
    // 5. Perform operation
    const result = await createResource(userId, body);
    
    // 6. Return response
    return NextResponse.json({ data: result }, { status: 201 });
  } catch (error) {
    console.error("API error:", error instanceof Error ? error.message : "Unknown");
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

## Dynamic Route Handler

```typescript
// app/api/resource/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const [headersList, { id }] = await Promise.all([
    headers(),
    params,
  ]);
  
  const { userId } = await whopSdk.verifyUserToken(headersList);
  
  // Fetch resource by id...
  return NextResponse.json({ data });
}
```

## Response Patterns

### Success

```typescript
return NextResponse.json({ data: result });
return NextResponse.json({ data: result }, { status: 201 }); // Created
```

### Errors

```typescript
return NextResponse.json({ error: "Not found" }, { status: 404 });
return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
return NextResponse.json({ error: "Forbidden" }, { status: 403 });
return NextResponse.json({ error: "Bad request" }, { status: 400 });
```

## Input Validation Order

1. **Authentication** - Verify token first
2. **Input validation** - Validate before expensive operations
3. **Authorization** - Check access after auth
4. **Rate limiting** - Before expensive operations
5. **Business logic** - Only after all checks pass

```typescript
export async function POST(request: NextRequest) {
  const [headersList, body] = await Promise.all([headers(), request.json()]);
  
  // 1. Auth
  const { userId } = await whopSdk.verifyUserToken(headersList);
  
  // 2. Validate (cheap, sync)
  if (!body.text || body.text.length > 5000) {
    return NextResponse.json({ error: "Invalid input" }, { status: 400 });
  }
  
  // 3. Authorize (check access)
  const { hasAccess } = await whopSdk.access.checkIfUserHasAccessToExperience({
    userId,
    experienceId: body.experienceId,
  });
  
  if (!hasAccess) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }
  
  // 4. Now perform the operation
  const result = await performOperation(body);
  
  return NextResponse.json({ data: result });
}
```
