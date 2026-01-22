---
name: auth-access-check
description: Check user access levels before allowing operations
metadata:
  tags: authentication, authorization, access, permissions
---

After verifying the user token, check their access level to the resource.

## Access Levels

| Level | Meaning | Use Case |
|-------|---------|----------|
| `no_access` | User has no access | Deny all operations |
| `customer` | Paying member | Experience page access |
| `admin` | Company admin/moderator | Dashboard access, management |

## Check Access Method

```typescript
import { whopsdk } from "@/lib/whop-sdk";

const response = await whopsdk.users.checkAccess(
  "resource_id",  // First argument: exp_xxx, biz_xxx, or prod_xxx
  { id: userId }  // Second argument: user object with id
);

// Response:
// { has_access: true, access_level: "customer" }
```

## Resource ID Types

| Prefix | Resource | Example |
|--------|----------|---------|
| `exp_` | Experience | `exp_xxxxxxxxxxxxx` |
| `biz_` | Company | `biz_xxxxxxxxxxxxx` |
| `prod_` | Product | `prod_xxxxxxxxxxxxx` |

## Checking Experience Access

```typescript
const { has_access, access_level } = await whopsdk.users.checkAccess(
  experienceId, // exp_xxxxx
  { id: userId }
);

if (!has_access) {
  return NextResponse.json({ error: "Access denied" }, { status: 403 });
}

// access_level is "customer" | "admin"
```

## Checking Company Access (Admin Routes)

```typescript
const { has_access, access_level } = await whopsdk.users.checkAccess(
  companyId, // biz_xxxxx
  { id: userId }
);

// For admin-only routes, verify access level
if (!has_access || access_level !== "admin") {
  return NextResponse.json({ error: "Admin access required" }, { status: 403 });
}
```

## Checking Product Access

```typescript
const { has_access, access_level } = await whopsdk.users.checkAccess(
  productId, // prod_xxxxx
  { id: userId }
);

if (!has_access) {
  return NextResponse.json({ error: "Purchase required" }, { status: 403 });
}
```

## By Username

You can also check access by username instead of user ID:

```typescript
const response = await whopsdk.users.checkAccess(
  "exp_xxxxx",
  { id: "johndoe42" } // Username works too
);
```

## Parallel Access Check Pattern

For better performance, run access checks in parallel when safe:

```typescript
const headersList = await headers();
const { userId } = await whopsdk.verifyUserToken(headersList);

// These are independent - run in parallel
const [accessCheck, userData] = await Promise.all([
  whopsdk.users.checkAccess(experienceId, { id: userId }),
  fetchUserData(userId),
]);

if (!accessCheck.has_access) {
  return NextResponse.json({ error: "Forbidden" }, { status: 403 });
}
```

## Common Pattern: Experience Page

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
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    experienceId,
    { id: userId }
  );
  
  if (!has_access) {
    return <AccessDenied />;
  }
  
  // access_level available for conditional rendering
  const isAdmin = access_level === "admin";
  
  return <AppContent userId={userId} isAdmin={isAdmin} />;
}
```

## Common Pattern: Dashboard Page

```typescript
export default async function DashboardPage({ 
  params 
}: { 
  params: Promise<{ companyId: string }> 
}) {
  const headersList = await headers();
  const { companyId } = await params;
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    companyId,
    { id: userId }
  );
  
  // Dashboard requires admin access
  if (!has_access || access_level !== "admin") {
    return <AdminAccessRequired />;
  }
  
  return <AdminDashboard companyId={companyId} />;
}
```

## Reference

- [Authentication Guide](https://docs.whop.com/developer/guides/authentication)
- [Check Access API](https://docs.whop.com/api-reference/users/check-access)
