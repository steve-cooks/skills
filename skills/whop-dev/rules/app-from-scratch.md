---
name: app-from-scratch
description: Build a Whop app from an empty repository. ONLY use when starting fresh - empty folder, no existing project. Triggers on "build me a Whop app" in empty repo, "I want to create a Whop app", or "start a new Whop app". Do NOT use for existing projects - use specific rules instead.
metadata:
  tags: scaffold, new-app, getting-started, template, empty-repo
---

# Build a Whop App From Scratch

**When to use:** Empty repo, no existing project. User wants to build a new Whop app from nothing.

**When NOT to use:** Existing codebase, already has a Whop app, just needs to add features → use specific rules directly (sdk-setup, auth-verify-token, payments-checkout, etc.)

---

## Detect User Experience

**Technical user signals:**
- Uses terms like SDK, webhooks, B2B/B2C, API
- Says "scaffold me a B2C app with Supabase"
- Knows what they want, just needs it built

→ Move faster, less questions, build efficiently.

**Non-technical / vibe coder signals:**
- Describes idea in plain language ("I want to build...")
- Asks "how do I start?"
- Unfamiliar with Whop concepts

→ Full guided flow with discovery questions.

**If unclear, ask:**
> "Want me to walk you through each step, or just scaffold and build?"

---

## Phase 1: Discovery

Before writing code, understand what they want.

**For non-technical users, ask plain-language questions:**
> "Tell me about the app you want to build. What should it do?"

Then clarify as needed:
- "Walk me through what happens when someone uses your app."
- "Will your community members use this, or just you/your team?"
- "Does it need to save anything? (user data, posts, files, etc.)"

**For technical users, confirm quickly:**
> "B2B or B2C? Need a database?"

Keep going until you understand:
1. **Who uses it**: Members (B2C) or just admins (B2B)?
2. **What data**: None, Whop-native (files/forums/chat), or Supabase?
3. **Core features**: The 2-3 main things it does.

## Phase 2: Confirm Plan

Summarize back:
> "Here's the plan:
> - **Type**: [B2C / B2B]
> - **What it does**: [description]
> - **Storage**: [None / Whop APIs / Supabase]
> 
> Sound right?"

Wait for confirmation.

## Phase 3: Setup

### 1. Install Companion Skills
```bash
npx skills add https://github.com/anthropics/skills --skill frontend-design && \
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices && \
npx skills add https://github.com/supabase/agent-skills --skill supabase-postgres-best-practices && \
npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines
```

### 2. Create Whop App (User Action Required)

Guide user:
> "Create your app in Whop:
> 1. Go to [whop.com/dashboard/developer](https://whop.com/dashboard/developer)
> 2. Click **Create App**, name it
> 3. Copy the environment variables and paste them here"

**Wait for API keys before continuing.**

### 3. Scaffold Project
```bash
pnpm create next-app@latest -e https://github.com/whopio/whop-nextjs-app-template [app-name]
cd [app-name] && pnpm install
```

Create `.env.development.local` with their keys:
```bash
WHOP_API_KEY=[their key]
NEXT_PUBLIC_WHOP_APP_ID=[their app id]
```

### 4. Configure App Views (User Action Required)

> "In Whop dashboard → Hosting section, set these paths:"

| Type | App Path | Dashboard Path | Discover Path |
|------|----------|----------------|---------------|
| B2C | `/experiences/[experienceId]` | `/dashboard/[companyId]` (optional) | `/discover` |
| B2B | *leave empty* | `/dashboard/[companyId]` | `/discover` |

**Wait for user to save settings.**

### 5. Test Locally

> "Run `pnpm dev`, then:
> 1. In Whop dashboard, click **Install App** → install to your whop
> 2. You'll see an error - enable **Localhost mode** (settings icon, top right)
> 
> You should see your local app in Whop. Working?"

Troubleshoot if needed.

## Phase 4: Build (Autonomous Loop)

After setup is complete, build the entire app autonomously. **Do not stop to ask user to test individual features.**

### Build Loop

```
while (features_remaining):
    1. Implement next feature
    2. Test it yourself (run dev server, check for errors)
    3. Fix any issues
    4. Move to next feature

until: All planned features are working
```

**Read scaffold guide for app type:**
- B2B → [app-scaffold-b2b.md](app-scaffold-b2b.md)
- B2C → [app-scaffold-b2c.md](app-scaffold-b2c.md)

**Hand off to companion skills as needed:**
- UI/design → `frontend-design`
- Database schema → `supabase-postgres-best-practices` + [app-database.md](app-database.md)
- React performance → `vercel-react-best-practices`

**Self-testing during build:**
- Run `pnpm dev` and verify pages load
- Check browser console for errors
- Test auth flows work (verifyUserToken)
- Test data operations (create, read, update)
- Verify both light and dark mode

**CRITICAL: Never deliver a broken product.** If something doesn't work, fix it before moving on. The user should always receive a working product.

### When to Pause for User

Only interrupt the build loop if:
- Need API keys or env vars from user
- Need user to configure something in Whop dashboard
- Blocked by a decision only user can make
- Genuinely stuck and need clarification

Otherwise, keep building until done.

## Phase 5: Present Working Product

Once all features are built and tested:

> "Your app is ready! Here's what I built:
> - [Feature 1]: [how it works]
> - [Feature 2]: [how it works]
> - [Feature 3]: [how it works]
> 
> Run `pnpm dev` and test it out in Whop (Localhost mode). Let me know if you want any changes."

**Wait for user feedback.** They may want:
- Design tweaks → hand off to `frontend-design`
- New features → add to the loop
- Bug fixes → fix and re-test
- Ready to deploy → proceed to Phase 6

## Phase 6: Deploy

When user approves the local version:

1. Deploy to Vercel/Railway/Cloudflare
2. Set env vars on host
3. Guide user to set **Base URL** in Whop dashboard
4. Disable Localhost mode

> "Your app is live at [url]!"

## Phase 7: Polish (Optional)

If user wants to launch on App Store:
- Verify light AND dark mode work
- Check all features end-to-end
- Remove any placeholder content
- Verify access control (members can't see admin stuff)

Hand off to `web-design-guidelines` for final UI/UX review.

For App Store listing → [app-store-submission.md](app-store-submission.md)

---

## Architecture Reference

| User Need | Solution |
|-----------|----------|
| Upload/share files | Whop Files API - no database needed |
| Posts, discussions | Whop Forums API - no database needed |
| Chat, messages | Whop Chat API - no database needed |
| Calculator, display info | Stateless - no storage needed |
| Custom data (profiles, products, votes) | Supabase database |

## Related Rules

| Topic | Rule |
|-------|------|
| SDK setup | [sdk-setup.md](sdk-setup.md) |
| Authentication | [auth-verify-token.md](auth-verify-token.md) |
| Payments | [payments-checkout.md](payments-checkout.md) |
| Webhooks | [payments-webhooks.md](payments-webhooks.md) |
| Database | [app-database.md](app-database.md) |
| Companion skills | [companion-skills.md](companion-skills.md) |

## Reference

- [Whop B2B Apps Guide](https://docs.whop.com/whop-apps/b2b-apps)
- [Whop Developer Docs](https://docs.whop.com/developer/getting-started)
