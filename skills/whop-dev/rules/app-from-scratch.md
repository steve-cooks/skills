---
name: app-from-scratch
description: Build a complete Whop app from an empty repository. Use when starting a new Whop app, scaffolding a project, or when user says "build me a Whop app" or describes an app idea. Works for both beginners (guided discovery) and experienced developers (quick scaffold).
metadata:
  tags: scaffold, new-app, getting-started, template
---

# Build a Whop App From Scratch

Guide users from an empty repo to a deployed Whop app. Works for beginners and experts.

## For Beginners: Discovery Questions

Ask these plain-language questions to understand the app. Do NOT ask technical questions like "what database do you need?" - infer that yourself.

### Essential Questions (Ask First)

1. **"What does your app do? Describe it like you're explaining to a friend."**
   - Get the core functionality in their words
   - Example answers: "lets people ask me questions and pay for answers", "shows my community's stats", "lets members play a game"

2. **"Who will use this app?"**
   - **"Just you/your team"** → B2B (Dashboard only)
   - **"My community members/customers"** → B2C (Experience page)
   - **"Both - members use it, but I configure it"** → B2C with Dashboard

### Follow-up Questions (If Needed)

3. **"What happens when someone uses your app? Walk me through it."**
   - Reveals the user flow and what data gets created
   - Example: "They type a question, pay $5, I answer it, they see my answer"

4. **"Does your app need to remember anything between visits?"**
   - Yes (questions, votes, purchases, profiles) → Needs database
   - No (calculator, embed, config tool) → No database needed

5. **"Do you want real-time updates? Like seeing new messages instantly?"**
   - Yes → Full-stack with Supabase realtime
   - No → Simpler architecture works

## For Experts: Quick Decision

Skip discovery if the user already knows what they want. Jump straight to scaffolding.

**Fast path triggers:**
- "Build me a B2B analytics dashboard"
- "I need a B2C app with Supabase"
- "Scaffold a simple widget app"

## Architecture Decision Matrix

Based on user's description, determine architecture:

| User Says... | Architecture | Why |
|--------------|--------------|-----|
| "Display info", "embed something", "calculator", "config panel" | **Simple** (no DB) | No data to persist |
| "Upload files", "share files with members" | **Whop-native** | Use `client.files.upload()` |
| "Post updates", "discussion board" | **Whop-native** | Use Whop Forums API |
| "Chat", "messaging" | **Whop-native** | Use Whop Chat API |
| "Questions", "votes", "purchases", "profiles", "products", "analytics" | **Full-stack** | Custom data = Supabase |
| "Real-time", "live updates", "notifications when X happens" | **Full-stack** | Needs Supabase realtime |

## Example: Inferring Architecture

**User:** "I want an app where my community can ask me questions and I answer them for money"

**Analysis:**
- Who uses it? Community members → **B2C**
- What data? Questions, answers, payments → **Custom data = Full-stack**
- Real-time? Nice to have for seeing new questions → **Supabase with realtime**

**Result:** B2C app with Supabase database

---

**User:** "I need a dashboard to see how many members I have and their activity"

**Analysis:**
- Who uses it? Just the creator → **B2B**
- What data? Reading from Whop API, no custom storage → **Simple**

**Result:** B2B dashboard, no database needed

---

**User:** "A place for members to share and download files"

**Analysis:**
- Who uses it? Community members → **B2C**
- What data? Files → **Whop-native file storage**

**Result:** B2C app using Whop Files API, no external database

## App Type Summary

### B2C (Experience App)
- Community members/customers use it
- Has `/experiences/[experienceId]` route
- May also have `/dashboard/[companyId]` for admin settings
- Examples: Q&A, games, forums, content viewers, marketplaces

### B2B (Dashboard App)
- Only creator/admin uses it
- Only has `/dashboard/[companyId]` route
- No `/experiences` route
- Examples: Analytics, automation, CRM, email tools, settings panels

description: Build a complete Whop app from an empty repository. Use when starting a new Whop app, scaffolding 
a project, or when user says "build me a Whop app" or describes an app idea.
Guide users from an empty repo to a deployed Whop app.
## Step 1: Understand the App Idea
Ask clarifying questions if the user's requirements are unclear:
1. **What does your app do?** (Get a clear description)
2. **Who uses it?** (Determines B2B vs B2C)
3. **What data needs to be stored?** (Determines architecture)
## Step 2: Determine App Type
### B2C (Experience App)
**Choose if:** Community members/customers will use the app
- Has `/experiences/[experienceId]` route (required)
- Examples: Q&A apps, games, forums, content viewers, community tools
**Choose if:** Only creators/admins use the app (no customer-facing UI)

- Examples: Analytics dashboards, automation tools, email senders, CRM, offer managers
**Quick test:** "Will community members interact with this app?"
- Yes → B2C
- No (admin only) → B2B
## Architecture Summary

### Simple (No Storage)
- Frontend-only or frontend + API routes
- No data persistence between sessions
- Examples: Calculators, embeds, widgets, config panels

### Whop-Native
- Use Whop's built-in storage APIs
- Files: `client.files.upload()`
- Posts: Whop Forums API
- Messages: Whop Chat API

### Full-Stack (Supabase)
- Custom data models
- PostgreSQL database
- Optional real-time subscriptions
- Examples: Q&A apps, marketplaces, analytics with custom metrics

## Scaffold the Project

Once architecture is determined:

```bash
pnpm create next-app@latest \
    -e https://github.com/whopio/whop-nextjs-app-template \
    my-whop-app

cd my-whop-app
pnpm install
```

### Project Structure

```
my-whop-app/
├── app/
│   ├── experiences/[experienceId]/page.tsx  # B2C: Customer view
│   ├── dashboard/[companyId]/page.tsx       # Admin view
│   ├── discover/page.tsx                    # Public marketing
│   ├── api/                                 # API routes
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── lib/
│   └── whop-sdk.ts                          # SDK setup
├── components/
├── .env.development
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

## Step 5: Follow Sub-Guides

Based on app type and architecture, follow these guides in order:
## Next Steps by App Type

### B2B Apps
1. [app-scaffold-b2b.md](app-scaffold-b2b.md) - Dashboard template
2. [app-database.md](app-database.md) - If full-stack
3. [app-whop-dashboard.md](app-whop-dashboard.md) - Configure Whop Developer Dashboard
4. [app-deployment.md](app-deployment.md) - Deploy to Vercel

### B2C Apps
1. [app-scaffold-b2c.md](app-scaffold-b2c.md) - Experience template
2. [app-database.md](app-database.md) - If full-stack
3. [app-whop-dashboard.md](app-whop-dashboard.md) - Configure Whop
4. [app-deployment.md](app-deployment.md) - Deploy to Vercel

### Optional
- [app-store-submission.md](app-store-submission.md) - Submit/Publish to App Store

## End-to-End Checklist

- [ ] Understand app idea (discovery or expert fast-path)
- [ ] Determine app type (B2B/B2C)
- [ ] Determine architecture (Simple/Whop-native/Full-stack)
- [ ] Scaffold project from template
- [ ] Configure routes for app type
- [ ] Set up SDK in `lib/whop-sdk.ts`
- [ ] Add authentication to protected routes
- [ ] Set up database if full-stack
- [ ] Configure Whop Developer Dashboard
- [ ] Deploy to Vercel
- [ ] Test in production Whop iframe

## Related Rules

- [sdk-setup.md](sdk-setup.md) - SDK configuration
- [views-structure.md](views-structure.md) - Page structure
- [auth-verify-token.md](auth-verify-token.md) - Authentication
- [ui-frosted.md](ui-frosted.md) - UI components
