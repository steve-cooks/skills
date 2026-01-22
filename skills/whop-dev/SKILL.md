---
name: whop-dev
description: Build Whop apps with Next.js. Includes a "build from scratch" guide for going from app idea to deployed app, plus best practices for authentication, payments, webhooks, and UI. Use when building apps for the Whop platform. Triggers on "build me a Whop app", @whop/sdk, verifyUserToken, checkAccess, checkout flows, webhook handling, Experience/Dashboard pages, Frosted UI.
metadata:
  tags: whop, nextjs, payments, sdk, frosted-ui, authentication
  author: Steve | https://whop.com/@stevecooks | https://x.com/steve_cook
  version: "1.1.0"
---

# Whop App Development

Build apps that run inside the Whop platform - handling authentication, payments, webhooks, and UI all with best practices.

## When to Use

Use this skill when:

- Building a new Whop app from scratch
- Setting up `@whop/sdk` or `@whop/react`
- Implementing authentication with `verifyUserToken` or `checkAccess`
- Creating payment flows, checkout, or webhooks
- Building Experience, Dashboard, or Discover pages
- Working with Frosted UI components

## How to Use

Read individual rule files for detailed explanations and code examples. Start with the "Build From Scratch" guides if you are in a brand new repo, or jump to specific rules in the Quick Reference.

---

## Build From Scratch

New to Whop? Follow these guides to go from idea to deployed app:

| Step | Guide | Description |
|------|-------|-------------|
| 1 | [app-from-scratch.md](rules/app-from-scratch.md) | **START HERE** - Plan your app type and architecture |
| 2a | [app-scaffold-b2b.md](rules/app-scaffold-b2b.md) | B2B: Dashboard-only apps for admins |
| 2b | [app-scaffold-b2c.md](rules/app-scaffold-b2c.md) | B2C: Experience apps for members |
| 3 | [app-database.md](rules/app-database.md) | Add Supabase (if needed) |
| 4 | [app-whop-dashboard.md](rules/app-whop-dashboard.md) | Configure in Whop Developer Dashboard |
| 5 | [app-deployment.md](rules/app-deployment.md) | Deploy to Vercel |
| 6 | [app-store-submission.md](rules/app-store-submission.md) | Submit to App Store (optional) |

---

## Quick Reference

### Authentication & Security (CRITICAL)

| Rule | Description |
|------|-------------|
| [auth-verify-token.md](rules/auth-verify-token.md) | MUST verify user tokens server-side |
| [auth-access-check.md](rules/auth-access-check.md) | MUST check access levels before operations |
| [security-checklist.md](rules/security-checklist.md) | Security best practices checklist |
| [auth-headers.md](rules/auth-headers.md) | Extract tokens from headers |
| [auth-oauth.md](rules/auth-oauth.md) | OAuth for external apps |
| [input-sanitization.md](rules/input-sanitization.md) | XSS prevention |

### SDK Setup (CRITICAL)

| Rule | Description |
|------|-------------|
| [sdk-setup.md](rules/sdk-setup.md) | Initialize Whop SDK (required) |
| [sdk-iframe.md](rules/sdk-iframe.md) | Client-side iframe SDK |

### Payments (HIGH)

| Rule | Description |
|------|-------------|
| [payments-checkout.md](rules/payments-checkout.md) | Create checkout flows |
| [payments-webhooks.md](rules/payments-webhooks.md) | Handle payment webhooks |
| [payments-transfers.md](rules/payments-transfers.md) | Send payouts to users |
| [payments-billing.md](rules/payments-billing.md) | Billing portal & saved methods |

### App Views (HIGH)

| Rule | Description |
|------|-------------|
| [views-structure.md](rules/views-structure.md) | Experience, Dashboard, Discover pages |
| [views-routing.md](rules/views-routing.md) | Dynamic routing patterns |

### Members & CRM (HIGH)

| Rule | Description |
|------|-------------|
| [api-memberships.md](rules/api-memberships.md) | Memberships & user management |

### UI & Design (MEDIUM)

| Rule | Description |
|------|-------------|
| [ui-frosted.md](rules/ui-frosted.md) | Frosted UI components |
| [ui-dark-mode.md](rules/ui-dark-mode.md) | Dark mode handling |
| [ui-tailwind.md](rules/ui-tailwind.md) | Tailwind CSS setup |

### API Patterns (MEDIUM)

| Rule | Description |
|------|-------------|
| [api-structure.md](rules/api-structure.md) | API route patterns |
| [api-parallel.md](rules/api-parallel.md) | Parallel data fetching |

### Storage & Files (MEDIUM)

| Rule | Description |
|------|-------------|
| [files-upload.md](rules/files-upload.md) | Whop native file storage |

### Engagement (MEDIUM)

| Rule | Description |
|------|-------------|
| [notifications.md](rules/notifications.md) | Push notifications |
| [engagement-forums.md](rules/engagement-forums.md) | Forum posts & comments |
| [engagement-chat.md](rules/engagement-chat.md) | Chat messages |

### Development (LOW)

| Rule | Description |
|------|-------------|
| [dev-proxy.md](rules/dev-proxy.md) | Local development proxy |
| [dev-sandbox.md](rules/dev-sandbox.md) | Sandbox testing |

---

## App Views

Whop apps have three entry points:

| View | Route | Purpose |
|------|-------|---------|
| Experience | `/experiences/[experienceId]` | Customer-facing UI |
| Dashboard | `/dashboard/[companyId]` | Admin management |
| Discover | `/discover` | Public marketing page |

## Tech Stack

**Required:**
- `@whop/sdk` - Server-side API client
- `@whop/react` - React hooks and Frosted UI

**Recommended:**
- Next.js (App Router)
- TypeScript
- Tailwind CSS + Frosted UI

**Flexible:**
- Database: Supabase recommended, but any works
- UI: Frosted UI, Shadcn, Radix - your choice

## Platforms (Advanced)

For marketplaces with connected accounts:
- [Enroll connected accounts](https://docs.whop.com/developer/platforms/enroll-connected-accounts)
- [Collect payments](https://docs.whop.com/developer/platforms/collect-payments-for-connected-accounts)
- [Enable payouts](https://docs.whop.com/developer/platforms/render-payout-portal)

## Resources

- [Whop Developer Docs](https://docs.whop.com/developer/getting-started)
- [API Reference](https://docs.whop.com/developer/api/getting-started)
- [Frosted UI Storybook](https://storybook.whop.com/)
- MCP Server: `https://mcp.whop.com/mcp` (Cursor) or `https://mcp.whop.com/sse` (Claude)

## Related Skills

For specialized patterns, use alongside:
- `supabase-postgres-best-practices` - Database optimization
- `vercel-react-best-practices` - React/Next.js performance
- `web-design-guidelines` - UI/UX review
