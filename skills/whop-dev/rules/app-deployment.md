---
name: app-deployment
description: Deploy your Whop app to Vercel. Use when ready to deploy, setting up production environment, or configuring hosting.
metadata:
  tags: deployment, vercel, production, hosting
---

# Deploy to Vercel

Step-by-step guide to deploy your Whop app to production.

## Prerequisites

- GitHub account with your code pushed
- Vercel account (free tier works)
- Environment variables ready

## Step 1: Push to GitHub

If not already done:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/your-repo.git
git push -u origin main
```

## Step 2: Connect to Vercel

1. Go to [vercel.com](https://vercel.com) and sign up/login
2. Click **Add New Project**
3. Select **Import Git Repository**
4. Choose your GitHub repository
5. Vercel auto-detects Next.js settings

## Step 3: Configure Environment Variables

In the Vercel project setup, add your environment variables:

### Required Variables

| Variable | Value | Notes |
|----------|-------|-------|
| `WHOP_API_KEY` | Your API key | From Whop Dashboard |
| `NEXT_PUBLIC_WHOP_APP_ID` | app_xxxxx | From Whop Dashboard |

### If Using Webhooks

| Variable | Value |
|----------|-------|
| `WHOP_WEBHOOK_SECRET` | whsec_xxxxx |

### If Using Supabase

| Variable | Value |
|----------|-------|
| `SUPABASE_URL` | https://xxx.supabase.co |
| `SUPABASE_SERVICE_ROLE_KEY` | Your service role key |
| `NEXT_PUBLIC_SUPABASE_URL` | https://xxx.supabase.co |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Your anon key |

## Step 4: Deploy

1. Click **Deploy**
2. Wait for build to complete (usually 1-2 minutes)
3. Note your deployment URL (e.g., `https://your-app.vercel.app`)

## Step 5: Update Whop Dashboard

1. Go to your app in Whop Developer Dashboard
2. Navigate to **Hosting** section
3. Set **Base URL** to your Vercel deployment URL
4. Verify paths are still correct:
   - B2C: App path = `/experiences/[experienceId]`
   - B2B: App path = empty, Dashboard path = `/dashboard/[companyId]`

## Step 6: Update Webhook URL (If Using)

1. Go to **Webhooks** in Whop Dashboard
2. Update webhook URL to: `https://your-app.vercel.app/api/webhooks`

## Step 7: Verify Deployment

1. Open your Whop where the app is installed
2. Disable localhost mode (settings icon in app frame)
3. Refresh - your production app should load
4. Test all functionality

## Custom Domain (Optional)

1. In Vercel project → **Settings** → **Domains**
2. Add your custom domain
3. Follow DNS configuration instructions
4. Update Base URL in Whop Dashboard to your custom domain

## Automatic Deployments

Vercel automatically deploys when you push to main:

```bash
git add .
git commit -m "Update feature"
git push
```

New deployment will be live in ~1-2 minutes.

## Troubleshooting

### Build Fails

Check build logs in Vercel for errors. Common issues:
- Missing environment variables
- TypeScript errors
- Missing dependencies

### App Shows Error in Whop

- Verify Base URL is correct (no trailing slash)
- Check environment variables are set in Vercel
- View Vercel function logs for errors

### Webhooks Not Working

- Verify webhook URL is publicly accessible
- Check webhook secret matches
- View Vercel function logs for `/api/webhooks`

### Server Actions Not Working

Ensure `next.config.ts` uses the Whop wrapper:

```typescript
import { withWhopAppConfig } from "@whop/react/next.config";
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // your config
};

export default withWhopAppConfig(nextConfig);
```

## Production Checklist

- [ ] All environment variables set in Vercel
- [ ] Base URL updated in Whop Dashboard
- [ ] Webhook URL updated (if using webhooks)
- [ ] Localhost mode disabled
- [ ] App loads correctly in Whop iframe
- [ ] Authentication works
- [ ] All features functional
- [ ] Database operations work (if using Supabase)

## Next Steps

- Submit to App Store - see [app-store-submission.md](app-store-submission.md)
