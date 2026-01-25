---
name: app-scaffold-b2b
description: Scaffold a B2B Whop app with dashboard-only structure for creator/admin tools. Use when building automation tools, analytics dashboards, CRM, email senders, or any app only admins use.
metadata:
  tags: b2b, dashboard, admin, scaffold, template
---

# B2B App Scaffold (Dashboard Only)

For apps that only creators/admins use. No customer-facing `/experiences` route.

## Use Cases

- Analytics dashboards
- Automation tools
- Email/notification senders
- CRM and member management
- Offer/product creators
- Settings and configuration tools

## Project Setup

```bash
pnpm create next-app@latest \
    -e https://github.com/whopio/whop-nextjs-app-template \
    my-b2b-app

cd my-b2b-app
pnpm install
```

## Directory Structure

```
app/
├── dashboard/
│   └── [companyId]/
│       └── page.tsx          # Main admin interface
├── discover/
│   └── page.tsx              # Public marketing page
├── api/
│   └── webhooks/
│       └── route.ts          # Webhook handler (if needed)
├── layout.tsx
├── page.tsx                  # Redirects to /discover
└── globals.css
```

**Remove** the `/experiences` folder - B2B apps don't need it.

## Core Files

### lib/whop-sdk.ts

```typescript
import { Whop } from "@whop/sdk";

export const whopsdk = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID,
  apiKey: process.env.WHOP_API_KEY,
});
```

### app/layout.tsx

```typescript
import { WhopApp } from "@whop/react/components";
import "./globals.css";

const themeScript = `
(function() {
  try {
    var cookie = document.cookie.match(/whop-frosted-theme=appearance:(?<appearance>light|dark)/);
    var theme = cookie ? cookie.groups.appearance : (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    document.documentElement.classList.add(theme);
    document.documentElement.style.colorScheme = theme;
    if (theme === 'dark') {
      document.documentElement.style.backgroundColor = '#111';
    }
  } catch (e) {}
})();
`;

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <head>
        <script dangerouslySetInnerHTML={{ __html: themeScript }} />
      </head>
      <body>
        <WhopApp appearance="inherit">{children}</WhopApp>
      </body>
    </html>
  );
}
```

### app/page.tsx

```typescript
import { redirect } from "next/navigation";

export default function HomePage() {
  redirect("/discover");
}
```

### app/discover/page.tsx

```typescript
export default function DiscoverPage() {
  return (
    <div className="min-h-screen bg-gray-1 py-16 px-4">
      <div className="max-w-4xl mx-auto text-center space-y-8">
        <h1 className="text-9 font-bold text-gray-12">Your App Name</h1>
        <p className="text-5 text-gray-11 max-w-2xl mx-auto">
          Describe what your app does for creators.
        </p>
        
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mt-12">
          <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
            <h3 className="text-5 font-bold text-gray-12 mb-2">Feature 1</h3>
            <p className="text-4 text-gray-11">Description of feature.</p>
          </div>
          <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
            <h3 className="text-5 font-bold text-gray-12 mb-2">Feature 2</h3>
            <p className="text-4 text-gray-11">Description of feature.</p>
          </div>
          <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
            <h3 className="text-5 font-bold text-gray-12 mb-2">Feature 3</h3>
            <p className="text-4 text-gray-11">Description of feature.</p>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### app/dashboard/[companyId]/page.tsx

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

export default async function DashboardPage({
  params,
}: {
  params: Promise<{ companyId: string }>;
}) {
  const headersList = await headers();
  const { companyId } = await params;

  // Verify user and check admin access
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    companyId,
    { id: userId }
  );

  if (!has_access || access_level !== "admin") {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-1">
        <div className="text-center space-y-4">
          <h1 className="text-8 text-gray-12">Admin Access Required</h1>
          <p className="text-4 text-gray-11">
            Only admins can access this dashboard.
          </p>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-1 py-8 px-4">
      <div className="max-w-6xl mx-auto space-y-6">
        {/* Header */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <h1 className="text-8 font-bold text-gray-12 mb-2">Dashboard</h1>
          <p className="text-4 text-gray-11">Company: {companyId}</p>
        </div>

        {/* Stats Grid */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
            <p className="text-3 text-gray-11 mb-1">Metric 1</p>
            <p className="text-7 font-bold text-gray-12">0</p>
          </div>
          <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
            <p className="text-3 text-gray-11 mb-1">Metric 2</p>
            <p className="text-7 font-bold text-gray-12">0</p>
          </div>
          <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
            <p className="text-3 text-gray-11 mb-1">Metric 3</p>
            <p className="text-7 font-bold text-gray-12">0</p>
          </div>
        </div>

        {/* Main Content */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <h2 className="text-5 font-bold text-gray-12 mb-4">Main Content</h2>
          <p className="text-4 text-gray-11">
            Add your dashboard functionality here.
          </p>
        </div>
      </div>
    </div>
  );
}
```

## Whop Dashboard Configuration

In the Whop Developer Dashboard, configure paths:

| Setting | Value |
|---------|-------|
| App path | *Leave empty* |
| Dashboard path | `/dashboard/[companyId]` |
| Discover path | `/discover` |

## Environment Variables

```bash
# .env.local
WHOP_API_KEY=your_api_key
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx
```

## Next Steps

1. Add your business logic to the dashboard page
2. If you need a database, see [app-database.md](app-database.md)
3. Configure Whop Dashboard - see [app-whop-dashboard.md](app-whop-dashboard.md)
4. Deploy to Vercel - see [app-deployment.md](app-deployment.md)

## Companion Skill Handoff

This scaffold provides the **structure** (routes, auth, access checks). Hand off to companion skills for:

- **→ `frontend-design`** - Design dashboard UI, charts, data visualizations
- **→ `supabase-postgres-best-practices`** - If adding database, optimize schema and queries
- **→ `vercel-react-best-practices`** - Before deployment, optimize performance
- **→ `web-design-guidelines`** - Review UI/UX compliance before launch
