---
name: security-checklist
description: Security best practices for Whop app development
metadata:
  tags: security, authentication, authorization, validation
---

## Authentication Security

### Always Verify Tokens

```typescript
// CORRECT: Verify token before ANY operation
const { userId } = await whopSdk.verifyUserToken(headersList);

// WRONG: Trusting client-provided data
const userId = request.headers.get("x-user-id"); // NEVER DO THIS
```

### Handle Auth Errors Securely

```typescript
try {
  const { userId } = await whopSdk.verifyUserToken(headersList);
} catch (error) {
  // Don't leak error details
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```

## Authorization Security

### Always Check Access

```typescript
// After auth, verify user has access to the resource
const { hasAccess, accessLevel } = await whopSdk.access.checkIfUserHasAccessToExperience({
  userId,
  experienceId,
});

if (!hasAccess) {
  return NextResponse.json({ error: "Forbidden" }, { status: 403 });
}

// For admin-only routes
if (accessLevel !== "admin") {
  return NextResponse.json({ error: "Admin access required" }, { status: 403 });
}
```

### Resource Ownership

If your app uses a database, always verify ownership:

```typescript
// Always verify ownership when accessing user data
// Example with any database:
const item = await db.items.findOne({
  where: {
    id: itemId,
    user_id: userId, // CRITICAL: ownership check
  },
});

if (!item) {
  return NextResponse.json({ error: "Not found" }, { status: 404 });
}
```

## Input Validation

### Validate All Input

```typescript
export async function POST(request: NextRequest) {
  const body = await request.json();
  
  // Type validation
  if (typeof body.text !== "string") {
    return NextResponse.json({ error: "Invalid input" }, { status: 400 });
  }
  
  // Length limits
  if (body.text.length > 5000) {
    return NextResponse.json({ error: "Text too long" }, { status: 400 });
  }
  
  // Sanitize HTML/XSS
  const sanitizedText = sanitizeUserInput(body.text);
}
```

### Validate IDs

```typescript
// Validate Whop ID formats
function isValidWhopId(id: string, prefix: string): boolean {
  return id.startsWith(`${prefix}_`) && id.length > prefix.length + 1;
}

if (!isValidWhopId(experienceId, "exp")) {
  return NextResponse.json({ error: "Invalid experience ID" }, { status: 400 });
}
```

## Error Handling

### Never Leak Internal Details

```typescript
// WRONG: Leaks internal information
catch (error) {
  return NextResponse.json({ 
    error: error.message,  // May contain sensitive info
    stack: error.stack     // NEVER expose stack traces
  }, { status: 500 });
}

// CORRECT: Generic error message
catch (error) {
  console.error("API error:", error); // Log internally only
  return NextResponse.json(
    { error: "Internal server error" },
    { status: 500 }
  );
}
```

## Data Protection

### Explicit Field Selection

If using a database, only select/return needed fields:

```typescript
// WRONG: May expose sensitive fields
const users = await db.users.findMany(); // Returns everything

// CORRECT: Select only needed columns
const users = await db.users.findMany({
  select: { id: true, username: true, avatar_url: true },
  // No email, passwords, or internal fields
});
```

### Sanitize Response Data

```typescript
// Strip sensitive fields before returning to client
function sanitizeForClient<T extends Record<string, unknown>>(data: T) {
  const { internal_notes, admin_only_field, password_hash, ...safe } = data;
  return safe;
}

return NextResponse.json({ data: sanitizeForClient(result) });
```

## Environment Security

### Never Expose Secrets

```typescript
// WRONG
console.log("API Key:", process.env.WHOP_API_KEY);

// CORRECT: Truncate if logging needed
console.log("Using API key:", process.env.WHOP_API_KEY?.slice(0, 8) + "...");
```

### Validate Environment

```typescript
// lib/whop-sdk.ts
if (!process.env.WHOP_API_KEY) {
  throw new Error("WHOP_API_KEY is required");
}

if (!process.env.NEXT_PUBLIC_WHOP_APP_ID) {
  throw new Error("NEXT_PUBLIC_WHOP_APP_ID is required");
}
```

## Rate Limiting

### Implement Per-User Limits

```typescript
const rateLimiter = new Map<string, { count: number; resetAt: number }>();

function checkRateLimit(userId: string, limit = 10, windowMs = 60000): boolean {
  const now = Date.now();
  const userLimit = rateLimiter.get(userId);
  
  if (!userLimit || userLimit.resetAt < now) {
    rateLimiter.set(userId, { count: 1, resetAt: now + windowMs });
    return true;
  }
  
  if (userLimit.count >= limit) {
    return false;
  }
  
  userLimit.count++;
  return true;
}

// In API route
if (!checkRateLimit(userId)) {
  return NextResponse.json({ error: "Too many requests" }, { status: 429 });
}
```

## Security Order

Always follow this order in API routes:

```
1. Authentication (verify token)     → 401 if failed
2. Input validation (cheap checks)   → 400 if failed  
3. Rate limiting                     → 429 if exceeded
4. Authorization (access check)      → 403 if denied
5. Business logic                    → Only after all pass
```
