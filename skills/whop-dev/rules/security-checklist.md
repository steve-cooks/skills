---
name: security-checklist
description: Security best practices for Whop app development
impact: CRITICAL
impactDescription: Prevents data breaches, auth bypass, and injection attacks
metadata:
  tags: security, authentication, authorization, validation
---

## Security Best Practices

Every Whop app MUST follow these security practices.

## Authentication Security

### MUST: Always Verify Tokens

```typescript
// CORRECT: Verify token before ANY operation
const { userId } = await whopSdk.verifyUserToken(headersList);

// WRONG: Trusting client-provided data - NEVER DO THIS
const userId = request.headers.get("x-user-id");
```

### MUST: Handle Auth Errors Securely

```typescript
try {
  const { userId } = await whopSdk.verifyUserToken(headersList);
} catch (error) {
  // Don't leak error details
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```

## Authorization Security

### MUST: Always Check Access

```typescript
// After auth, verify user has access to the resource
const { has_access, access_level } = await whopSdk.users.checkAccess(
  experienceId,
  { id: userId }
);

if (!has_access) {
  return NextResponse.json({ error: "Forbidden" }, { status: 403 });
}

// For admin-only routes
if (access_level !== "admin") {
  return NextResponse.json({ error: "Admin access required" }, { status: 403 });
}
```

### MUST: Verify Resource Ownership

If your app uses a database, ALWAYS verify ownership:

```typescript
// ALWAYS include ownership check in queries
const item = await db.items.findOne({
  where: {
    id: itemId,
    user_id: userId, // CRITICAL: prevents accessing other users' data
  },
});

if (!item) {
  return NextResponse.json({ error: "Not found" }, { status: 404 });
}
```

## Input Validation

### MUST: Validate All Input

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

### MUST: Validate Whop IDs

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

### NEVER: Leak Internal Details

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

### MUST: Use Explicit Field Selection

```typescript
// WRONG: May expose sensitive fields
const users = await db.users.findMany(); // Returns everything

// CORRECT: Select only needed columns
const users = await db.users.findMany({
  select: { id: true, username: true, avatar_url: true },
  // Excludes email, passwords, internal fields
});
```

### MUST: Sanitize Response Data

```typescript
// Strip sensitive fields before returning to client
function sanitizeForClient<T extends Record<string, unknown>>(data: T) {
  const { internal_notes, admin_only_field, password_hash, ...safe } = data;
  return safe;
}

return NextResponse.json({ data: sanitizeForClient(result) });
```

## Environment Security

### NEVER: Expose Secrets

```typescript
// WRONG
console.log("API Key:", process.env.WHOP_API_KEY);

// CORRECT: Truncate if logging needed
console.log("Using API key:", process.env.WHOP_API_KEY?.slice(0, 8) + "...");
```

### MUST: Validate Environment

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

### SHOULD: Implement Per-User Limits

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

ALWAYS follow this order in API routes:

```
1. Authentication (verify token)     → 401 if failed
2. Input validation (cheap checks)   → 400 if failed  
3. Rate limiting                     → 429 if exceeded
4. Authorization (access check)      → 403 if denied
5. Business logic                    → Only after all pass
```

## Quick Checklist

- [ ] All routes verify user token
- [ ] All routes check access level
- [ ] All database queries include ownership filter
- [ ] All input is validated and sanitized
- [ ] No secrets in logs or responses
- [ ] Generic error messages to clients
- [ ] Rate limiting on public endpoints

## Reference

- [Security Best Practices](https://docs.whop.com/developer/guides/security)
