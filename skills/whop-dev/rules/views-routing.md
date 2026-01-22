---
name: views-routing
description: Dynamic route parameters in Whop apps
metadata:
  tags: routing, params, nextjs, dynamic-routes
---

## Accessing Route Parameters

Next.js 15+ passes `params` as a Promise.

**Incorrect (Next.js 14 pattern):**

```typescript
export default async function Page({ params }) {
  const { experienceId } = params; // Error in Next.js 15+
}
```

**Correct (Next.js 15+ pattern):**

```typescript
export default async function Page({
  params,
}: {
  params: Promise<{ experienceId: string }>;
}) {
  const { experienceId } = await params;
  // Now use experienceId
}
```

## Parallel Parameter + Auth Fetch

Fetch params and headers in parallel:

```typescript
export default async function ExperiencePage({
  params,
}: {
  params: Promise<{ experienceId: string }>;
}) {
  // Parallel fetch - both are independent
  const [{ experienceId }, headersList] = await Promise.all([
    params,
    headers(),
  ]);
  
  const { userId } = await whopSdk.verifyUserToken(headersList);
  
  // Continue with authenticated request...
}
```

## Search Params

For query parameters, use the same pattern:

```typescript
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ experienceId: string }>;
  searchParams: Promise<{ tab?: string }>;
}) {
  const [{ experienceId }, { tab }] = await Promise.all([
    params,
    searchParams,
  ]);
  
  const currentTab = tab ?? "default";
}
```

## API Route Parameters

For API routes with dynamic segments:

```typescript
// app/api/items/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  
  // Fetch item by id...
  return NextResponse.json({ id });
}
```

## Generating Static Params

For static generation of dynamic routes:

```typescript
export async function generateStaticParams() {
  // Return array of param objects
  return [
    { experienceId: "exp_xxxxx" },
    { experienceId: "exp_yyyyy" },
  ];
}
```

Note: Most Whop apps use dynamic rendering due to authentication requirements.
