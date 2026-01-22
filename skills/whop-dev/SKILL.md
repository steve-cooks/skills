---
name: whop-dev
description: Best practices for building Whop apps with Next.js. Use when creating apps that run inside the Whop platform, handling authentication, payments, webhooks, app views, or using @whop/sdk, @whop/react, Frosted UI. Triggers on Whop SDK setup, user token verification, checkout flows, webhook handling, Experience/Dashboard pages.
metadata:
  tags: whop, nextjs, payments, sdk, frosted-ui, authentication
  author: Steve | https://whop.com/@stevecooks | https://x.com/steve_cook
  version: "1.0.0"
---

## When to Apply

- Building a new Whop app
- Setting up Whop SDK authentication
- Creating payment flows or webhooks
- Building Experience, Dashboard, or Discover pages
- Using Frosted UI components
- Working with Whop's iframe environment

## Tech Stack

**Required:**
- `@whop/sdk` - Server-side API client
- `@whop/react` - React hooks and Frosted UI components

**Recommended (not mandatory):**
- Next.js (App Router) - Any recent version
- React 18+ 
- TypeScript
- Frosted UI + Tailwind CSS (for visual consistency with Whop)

**Flexible:**
- UI Library: Frosted UI recommended, but Radix, Shadcn, or any library works
- Database: Optional, any database (Whop is database-agnostic)
- Styling: Tailwind recommended with Frosted, but any CSS approach works

## App Views

| View | Route | Purpose |
|------|-------|---------|
| Experience | `/experiences/[experienceId]` | Customer-facing app UI |
| Dashboard | `/dashboard/[companyId]` | Admin management |
| Discover | `/discover` | Public marketing page |

## Quick Reference

### Authentication & Security (CRITICAL)
- [auth-verify-token.md](rules/auth-verify-token.md) - Verify user tokens server-side
- [auth-access-check.md](rules/auth-access-check.md) - Check access levels
- [auth-headers.md](rules/auth-headers.md) - Extract tokens from headers
- [auth-oauth.md](rules/auth-oauth.md) - OAuth for external apps
- [security-checklist.md](rules/security-checklist.md) - Security best practices
- [input-sanitization.md](rules/input-sanitization.md) - XSS prevention

### SDK Setup (CRITICAL)
- [sdk-setup.md](rules/sdk-setup.md) - Initialize Whop SDK
- [sdk-iframe.md](rules/sdk-iframe.md) - Client-side iframe SDK

### Payments (HIGH)
- [payments-checkout.md](rules/payments-checkout.md) - Checkout flows
- [payments-webhooks.md](rules/payments-webhooks.md) - Webhook handling
- [payments-transfers.md](rules/payments-transfers.md) - Send payouts
- [payments-billing.md](rules/payments-billing.md) - Billing portal & saved methods

### App Views (HIGH)
- [views-structure.md](rules/views-structure.md) - Page structure
- [views-routing.md](rules/views-routing.md) - Dynamic routing

### CRM & Members (HIGH)
- [api-memberships.md](rules/api-memberships.md) - Memberships & user management

### UI/Design (MEDIUM)
- [ui-frosted.md](rules/ui-frosted.md) - Frosted UI components
- [ui-dark-mode.md](rules/ui-dark-mode.md) - Dark mode handling
- [ui-tailwind.md](rules/ui-tailwind.md) - Tailwind v4 setup

### API Routes (MEDIUM)
- [api-structure.md](rules/api-structure.md) - Route patterns
- [api-parallel.md](rules/api-parallel.md) - Parallel fetching

### Files & Storage (MEDIUM)
- [files-upload.md](rules/files-upload.md) - Whop native file storage

### Engagement (MEDIUM)
- [notifications.md](rules/notifications.md) - Push notifications
- [engagement-forums.md](rules/engagement-forums.md) - Forum posts & comments
- [engagement-chat.md](rules/engagement-chat.md) - Chat messages

### Development (MEDIUM)
- [dev-proxy.md](rules/dev-proxy.md) - Development proxy
- [dev-sandbox.md](rules/dev-sandbox.md) - Sandbox testing

## Platforms (Advanced)

For building marketplaces with connected accounts, see:
- [Enroll connected accounts](https://docs.whop.com/developer/platforms/enroll-connected-accounts)
- [Collect payments for connected accounts](https://docs.whop.com/developer/platforms/collect-payments-for-connected-accounts)
- [Enable connected account payouts](https://docs.whop.com/developer/platforms/render-payout-portal)

## Official Documentation

- [Whop Developer Docs](https://docs.whop.com/developer/getting-started)
- [Whop API Reference](https://docs.whop.com/developer/api/getting-started)
- [Frosted UI Storybook](https://storybook.whop.com/)
- MCP: `https://mcp.whop.com/mcp` (Cursor) or `https://mcp.whop.com/sse` (Claude)

## Related Skills

```bash
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices
npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines
```
