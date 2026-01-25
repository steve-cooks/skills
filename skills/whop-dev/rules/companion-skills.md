---
name: companion-skills
description: Install and delegate to specialized skills for complete Whop app development
impact: HIGH
impactDescription: Enables whop-dev to stay lean while leveraging expert skills for design, performance, and database
metadata:
  tags: skills, setup, ecosystem, handoff
---

# Companion Skills

whop-dev focuses on Whop-specific patterns (SDK, auth, payments, webhooks). For broader development concerns, delegate to specialized companion skills.

## Required Skills

Install these skills alongside whop-dev for complete Whop app development:

| Skill | Purpose | Install Command |
|-------|---------|-----------------|
| `frontend-design` | Bold, distinctive UI design | `npx skills add https://github.com/anthropics/skills --skill frontend-design` |
| `vercel-react-best-practices` | React/Next.js performance (57 rules) | `npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices` |
| `supabase-postgres-best-practices` | Database optimization (30 rules) | `npx skills add https://github.com/supabase/agent-skills --skill supabase-postgres-best-practices` |
| `web-design-guidelines` | UI/UX compliance auditing | `npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines` |

## Auto-Install Command

Install all companion skills at once:

```bash
npx skills add https://github.com/anthropics/skills --skill frontend-design && \
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices && \
npx skills add https://github.com/supabase/agent-skills --skill supabase-postgres-best-practices && \
npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines
```

## When to Hand Off

### → frontend-design

Hand off when user needs:
- New UI components, pages, or layouts
- Visual design decisions (typography, color, spacing)
- Animations and micro-interactions
- Distinctive/memorable aesthetics (avoiding generic "AI slop")

**Trigger phrases**: "make it look good", "design a page", "create a component", "style this", "make it beautiful", "UI for..."

### → vercel-react-best-practices

Hand off when user needs:
- React component optimization
- Data fetching patterns (async, parallel, suspense)
- Bundle size reduction
- Re-render prevention
- Server component patterns

**Trigger phrases**: "optimize", "performance", "slow", "bundle size", "re-renders", "data fetching"

### → supabase-postgres-best-practices

Hand off when user needs:
- SQL query optimization
- Index design
- Schema design decisions
- Connection pooling setup
- Row-Level Security (RLS) patterns

**Trigger phrases**: "database", "query slow", "index", "schema", "RLS", "Supabase", "Postgres"

### → web-design-guidelines

Hand off when user needs:
- UI/UX review or audit
- Accessibility checks
- Design system compliance
- Best practices validation

**Trigger phrases**: "review my UI", "check accessibility", "audit design", "review UX"

## Handoff Pattern

When a task falls into another skill's domain:

1. **Acknowledge** the Whop context is set up correctly
2. **Delegate** by invoking the appropriate skill
3. **Return** to whop-dev for Whop-specific integration

Example flow:
```
User: "Build a Q&A page for my Whop app"

1. whop-dev: Set up /experiences/[experienceId] route with auth
2. → frontend-design: Design the Q&A interface
3. → supabase-postgres-best-practices: Design questions/answers schema
4. whop-dev: Wire up Whop payments for paid questions
5. → vercel-react-best-practices: Optimize data fetching
```

## Skill Responsibilities

### whop-dev owns:
- Whop SDK setup and configuration
- User authentication (`verifyUserToken`, `checkAccess`)
- Payment flows (checkout, webhooks, transfers)
- App views structure (Experience, Dashboard, Discover)
- Whop API integration (memberships, notifications, files)
- Webhook handling and security

### Companion skills own:
- **frontend-design**: Visual aesthetics, typography, color, animation
- **vercel-react-best-practices**: React patterns, performance, bundle optimization
- **supabase-postgres-best-practices**: Database queries, schema, indexes, RLS
- **web-design-guidelines**: UI/UX compliance, accessibility, design auditing

## Integration Notes

- Companion skills are framework-agnostic; whop-dev provides Whop-specific context
- When building from scratch, install companion skills FIRST before scaffolding
- Skills complement each other - use multiple skills for complex features
- whop-dev patterns (auth, payments) should wrap companion skill outputs
