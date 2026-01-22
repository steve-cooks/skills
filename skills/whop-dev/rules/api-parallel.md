---
name: api-parallel
description: Parallel fetching patterns in API routes
metadata:
  tags: api, performance, parallel, async
---

Eliminate waterfalls by running independent operations in parallel.

## Parallel Headers + Body

**Incorrect (sequential):**

```typescript
export async function POST(request: NextRequest) {
  const headersList = await headers();        // Wait...
  const body = await request.json();          // Wait...
  const { userId } = await whopSdk.verifyUserToken(headersList);
}
```

**Correct (parallel):**

```typescript
export async function POST(request: NextRequest) {
  // Headers and body are independent - fetch in parallel
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  const { userId } = await whopSdk.verifyUserToken(headersList);
}
```

## Parallel Data Fetching

**Incorrect (sequential):**

```typescript
const user = await fetchUser(userId);
const settings = await fetchSettings(companyId);
const balance = await fetchBalance(userId, companyId);
// 3 round trips!
```

**Correct (parallel):**

```typescript
const [user, settings, balance] = await Promise.all([
  fetchUser(userId),
  fetchSettings(companyId),
  fetchBalance(userId, companyId),
]);
// 1 round trip!
```

## Safe Parallelization After Auth

Auth must complete before data operations, but subsequent fetches can parallelize:

```typescript
export async function GET(request: NextRequest) {
  const headersList = await headers();
  
  // Auth first - MUST complete
  const { userId } = await whopSdk.verifyUserToken(headersList);
  
  // After auth, parallelize independent operations
  const [accessCheck, userData, companyData] = await Promise.all([
    whopSdk.access.checkIfUserHasAccessToExperience({ userId, experienceId }),
    fetchUserProfile(userId),
    fetchCompanyInfo(companyId),
  ]);
  
  // Check authorization result
  if (!accessCheck.hasAccess) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }
  
  return NextResponse.json({ userData, companyData });
}
```

## Parallel Params + Headers

```typescript
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  // Both are independent
  const [headersList, { id }] = await Promise.all([
    headers(),
    params,
  ]);
  
  const { userId } = await whopSdk.verifyUserToken(headersList);
  // Continue...
}
```

## When NOT to Parallelize

Don't parallelize when one operation depends on another:

```typescript
// WRONG - data fetch before auth verified!
const [authResult, sensitiveData] = await Promise.all([
  whopSdk.verifyUserToken(headersList),
  supabase.from("secrets").select("*"), // Fetching before auth!
]);
```

**Rule:** Auth/validation must complete before data operations.
