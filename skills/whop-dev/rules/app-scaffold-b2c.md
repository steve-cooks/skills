---
name: app-scaffold-b2c
description: Scaffold a B2C Whop app with experience pages for community members. Use when building games, forums, Q&A apps, content viewers, or any app community members interact with.
metadata:
  tags: b2c, experience, community, scaffold, template
---

# B2C App Scaffold (Experience + Dashboard)

For apps that community members/customers use. Has `/experiences` route and optionally `/dashboard` for admin settings.

## Use Cases

- Q&A and discussion apps
- Games and interactive content
- Community forums
- Content viewers
- Marketplace apps
- Chat and messaging tools

## Project Setup

```bash
pnpm create next-app@latest \
    -e https://github.com/whopio/whop-nextjs-app-template \
    my-b2c-app

cd my-b2c-app
pnpm install
```

## Directory Structure

```
app/
├── experiences/
│   └── [experienceId]/
│       └── page.tsx          # Customer-facing view
├── dashboard/
│   └── [companyId]/
│       └── page.tsx          # Admin settings (optional)
├── discover/
│   └── page.tsx              # Public marketing page
├── api/
│   └── ...                   # API routes
├── layout.tsx
├── page.tsx
└── globals.css
```

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
          Describe what your app does for community members.
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

### app/experiences/[experienceId]/page.tsx

```typescript
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";

export default async function ExperiencePage({
  params,
}: {
  params: Promise<{ experienceId: string }>;
}) {
  const headersList = await headers();
  const { experienceId } = await params;

  // Verify user and check access
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    experienceId,
    { id: userId }
  );

  if (!has_access) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-1">
        <div className="text-center space-y-4">
          <h1 className="text-8 text-gray-12">Access Denied</h1>
          <p className="text-4 text-gray-11">
            You don't have access to this experience.
          </p>
        </div>
      </div>
    );
  }

  // Check if user is admin (for showing admin link)
  const isAdmin = access_level === "admin";

  return (
    <div className="min-h-screen bg-gray-1 py-8 px-4">
      <div className="max-w-4xl mx-auto space-y-6">
        {/* Header */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <div className="flex justify-between items-start">
            <div>
              <h1 className="text-8 font-bold text-gray-12 mb-2">
                Welcome to the App
              </h1>
              <p className="text-4 text-gray-11">
                Your main app content goes here.
              </p>
            </div>
            
            {isAdmin && (
              <a
                href={`/dashboard/${experienceId.split("_")[0]}`}
                className="px-4 py-2 bg-gray-3 text-gray-12 rounded-lg hover:bg-gray-4 transition-colors text-4"
              >
                Admin Settings
              </a>
            )}
          </div>
        </div>

        {/* Main Content */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <h2 className="text-5 font-bold text-gray-12 mb-4">Content</h2>
          <p className="text-4 text-gray-11">
            Add your app functionality here. This is what community members see.
          </p>
        </div>

        {/* Example: User Actions */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <h2 className="text-5 font-bold text-gray-12 mb-4">Actions</h2>
          <div className="flex gap-4">
            <button className="px-4 py-2 bg-accent-9 text-white font-medium rounded-lg hover:bg-accent-10 transition-colors">
              Primary Action
            </button>
            <button className="px-4 py-2 bg-gray-3 text-gray-12 rounded-lg hover:bg-gray-4 transition-colors">
              Secondary Action
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### app/dashboard/[companyId]/page.tsx (Optional Admin Settings)

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
            Only admins can access settings.
          </p>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-1 py-8 px-4">
      <div className="max-w-4xl mx-auto space-y-6">
        {/* Header */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <h1 className="text-8 font-bold text-gray-12 mb-2">Admin Settings</h1>
          <p className="text-4 text-gray-11">
            Configure your app settings here.
          </p>
        </div>

        {/* Settings Sections */}
        <div className="bg-gray-2 border border-gray-6 rounded-lg p-6">
          <h2 className="text-5 font-bold text-gray-12 mb-4">General Settings</h2>
          <p className="text-4 text-gray-11">
            Add admin configuration options here.
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
| App path | `/experiences/[experienceId]` |
| Dashboard path | `/dashboard/[companyId]` (optional) |
| Discover path | `/discover` |

## Environment Variables

```bash
# .env.local
WHOP_API_KEY=your_api_key
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx
```

## Next Steps

1. Add your app functionality to the experience page
2. If you need a database, see [app-database.md](app-database.md)
3. Configure Whop Dashboard - see [app-whop-dashboard.md](app-whop-dashboard.md)
4. Deploy to Vercel - see [app-deployment.md](app-deployment.md)

## Companion Skill Handoff

This scaffold provides the **structure** (routes, auth, access checks). Hand off to companion skills for:

- **→ `frontend-design`** - Design the actual UI components, layouts, animations
- **→ `supabase-postgres-best-practices`** - If adding database, optimize schema and queries
- **→ `vercel-react-best-practices`** - Before deployment, optimize performance
- **→ `web-design-guidelines`** - Review UI/UX compliance before launch
