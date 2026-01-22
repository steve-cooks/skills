---
name: views-structure
description: Experience, Dashboard, and Discover page structure
metadata:
  tags: views, pages, routing, structure
---

## Three Primary Views

Whop apps have three entry points configured in the Developer Dashboard.

### 1. Experience Page (`/experiences/[experienceId]`)

Customer-facing view for users who purchased access.

```typescript
// app/experiences/[experienceId]/page.tsx
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

export default async function ExperiencePage({
  params,
}: {
  params: Promise<{ experienceId: string }>;
}) {
  const headersList = await headers();
  const { experienceId } = await params;
  
  // Verify and check access
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    experienceId,
    { id: userId }
  );
  
  if (!has_access) {
    return <AccessDenied />;
  }
  
  return (
    <main className="p-6">
      <h1>Welcome to the Experience</h1>
      {/* Customer content */}
    </main>
  );
}
```

### 2. Dashboard Page (`/dashboard/[companyId]`)

Admin interface for company owners and moderators.

```typescript
// app/dashboard/[companyId]/page.tsx
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

export default async function DashboardPage({
  params,
}: {
  params: Promise<{ companyId: string }>;
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
  
  return (
    <main className="p-6">
      <h1>Admin Dashboard</h1>
      {/* Management interface */}
    </main>
  );
}
```

### 3. Discover Page (`/discover`)

Public marketing page shown in Whop App Store.

```typescript
// app/discover/page.tsx
export default function DiscoverPage() {
  // No auth required - public page
  return (
    <main className="p-6">
      <h1>Welcome to My App</h1>
      <p>Discover what this app can do for your business.</p>
      {/* Marketing content, features, pricing */}
    </main>
  );
}
```

## Directory Structure

```
app/
├── experiences/
│   └── [experienceId]/
│       └── page.tsx        # Customer view
├── dashboard/
│   └── [companyId]/
│       └── page.tsx        # Admin view
├── discover/
│   └── page.tsx            # Public marketing
├── api/
│   └── ...                 # API routes
├── layout.tsx              # Root layout
├── page.tsx                # Redirect to /discover
└── globals.css
```

## Root Page Redirect

```typescript
// app/page.tsx
import { redirect } from "next/navigation";

export default function HomePage() {
  redirect("/discover");
}
```

## Root Layout with Iframe Provider

```typescript
// app/layout.tsx
import { WhopIframeSdkProvider } from "@whop/react";
import "./globals.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <WhopIframeSdkProvider>
          {children}
        </WhopIframeSdkProvider>
      </body>
    </html>
  );
}
```

## Whop Dashboard Configuration

Configure paths in Developer Dashboard:
- **App path**: `/experiences/[experienceId]`
- **Dashboard path**: `/dashboard/[companyId]`
- **Discover path**: `/discover`

Paths must match exactly.

## Reference

- [App Views Guide](https://docs.whop.com/developer/guides/app-views)
