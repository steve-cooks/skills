# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions that extend agent capabilities with specialized knowledge.

## What are Agent Skills?

Agent Skills teach AI coding assistants (Cursor, Claude, GitHub Copilot, etc.) domain-specific best practices. Instead of explaining the same patterns repeatedly, skills provide persistent context that agents automatically apply.

## Available Skills

### whop-dev

Complete guide for building Whop apps with Next.js, `@whop/sdk`, and Frosted UI. Contains 26 rules covering authentication, payments, webhooks, and more.

**Use when:**
- Building apps that run inside the Whop platform
- Setting up `@whop/sdk` authentication
- Creating checkout flows or handling webhooks
- Building Experience, Dashboard, or Discover pages

**Categories covered:**
- Authentication & Security (token verification, access checks)
- Payments (checkout, webhooks, transfers/payouts)
- SDK Setup (server-side and iframe client)
- App Views (Experience, Dashboard, Discover pages)
- Engagement (notifications, forums, chat)
- CRM (memberships, user management)

## Installation

```bash
npx skills add steve-cooks/skills --skill whop-dev
```

## Usage

Once installed, the skill is automatically available. Your AI agent will apply Whop best practices when relevant.

**Examples:**
```
Set up Whop authentication in my app
```
```
Create a checkout flow for my Whop app
```
```
Handle payment webhooks securely
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Main instructions for the agent
- `rules/` - Individual best practice rules

## Resources

- [SKILLS](https://skills.sh)
- [ME@X](https://x.com/steve_cook)
- [ME@Whop](https://whop.com/@stevecooks)
- [Whop Developer Docs](https://docs.whop.com/developer/getting-started)
- [Whop API Reference](https://docs.whop.com/developer/api/getting-started)
- [Frosted UI Storybook](https://storybook.whop.com/)

## License

MIT
