---
name: app-whop-dashboard
description: Configure your app in the Whop Developer Dashboard. Use when setting up API keys, configuring app paths, webhooks, or installing your app for testing.
metadata:
  tags: dashboard, configuration, api-keys, setup, webhooks
---

# Whop Developer Dashboard Setup

Step-by-step guide to configure your app in the Whop Developer Dashboard.

## Step 1: Create Your App

1. Go to [whop.com/dashboard](https://whop.com/dashboard)
2. Navigate to **Developer** section (in sidebar)
3. Find **Apps** section under Webhooks
4. Click **Create App**
5. Enter your app name (can change later)

**Important:** You cannot change the company your app is built in later.

## Step 2: Get Environment Variables

After creating your app, go to the **API** tab to find:

| Variable | Where to find | Description |
|----------|---------------|-------------|
| `WHOP_API_KEY` | API tab | Server-side API key (SECRET!) |
| `NEXT_PUBLIC_WHOP_APP_ID` | API tab | Your app ID (app_xxxxx) |
| `WHOP_WEBHOOK_SECRET` | Webhooks tab | Webhook signing secret (whsec_xxx) |

Copy these to your `.env.local`:

```bash
# .env.local
WHOP_API_KEY=your_api_key_here
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx
WHOP_WEBHOOK_SECRET=whsec_xxxxx
```

## Step 3: Configure Hosting (Paths)

Go to the **Hosting** section in your app settings.

### For B2C Apps (Experience + Dashboard)

| Setting | Value |
|---------|-------|
| Base URL | Your production URL (e.g., `https://myapp.vercel.app`) |
| App path | `/experiences/[experienceId]` |
| Dashboard path | `/dashboard/[companyId]` |
| Discover path | `/discover` |

### For B2B Apps (Dashboard Only)

| Setting | Value |
|---------|-------|
| Base URL | Your production URL (e.g., `https://myapp.vercel.app`) |
| App path | *Leave empty* |
| Dashboard path | `/dashboard/[companyId]` |
| Discover path | `/discover` |

**Note:** The placeholder text in the UI doesn't mean it's set - you must explicitly enter the paths.

## Step 4: Set Up Webhooks (If Needed)

If your app processes payments or needs event notifications:

1. Go to **Webhooks** tab
2. Click **Add Webhook**
3. Enter your webhook URL: `https://your-app.vercel.app/api/webhooks`
4. Select events to listen for:
   - `payment.succeeded` - Payment completed
   - `payment.failed` - Payment failed
   - `membership.created` - New membership
   - `membership.cancelled` - Membership cancelled
5. Copy the webhook secret to `WHOP_WEBHOOK_SECRET`

## Step 5: Install App for Testing

1. Find your **Install Link** on the app details page
2. Click the link or share it
3. Select which Whop to install the app on
4. Complete installation

### Enable Development Mode

After installing, when viewing your app in Whop:

1. Look for the translucent **settings icon** in the top-right of the app frame
2. Click it to open development settings
3. Enable **Localhost mode**
4. Set port to `3000` (or your dev server port)

Now your local development server will load inside the Whop iframe.

## Step 6: Test Your App

1. Run your dev server: `pnpm dev`
2. Open your Whop where the app is installed
3. Navigate to your app
4. With localhost mode enabled, you should see your local app

### Troubleshooting

**App not loading?**
- Verify paths are set correctly (not just placeholder text)
- Check that localhost mode is enabled
- Ensure dev server is running on the correct port

**Authentication errors?**
- Verify `WHOP_API_KEY` is correct
- Check `NEXT_PUBLIC_WHOP_APP_ID` matches your app

**Webhook not receiving events?**
- Verify webhook URL is correct and publicly accessible
- Check webhook secret matches `WHOP_WEBHOOK_SECRET`

## Step 7: Configure for Production

Before deploying:

1. Set **Base URL** to your production domain
2. Disable localhost mode
3. Verify all paths are correct
4. Test webhook endpoints are accessible

## Environment Variables Summary

```bash
# Required
WHOP_API_KEY=your_api_key              # Server-side only
NEXT_PUBLIC_WHOP_APP_ID=app_xxxxx      # Can be public

# If using webhooks
WHOP_WEBHOOK_SECRET=whsec_xxxxx        # Server-side only

# If using Supabase (see app-database.md)
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=xxx
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
```

## Next Steps

- Deploy to Vercel - see [app-deployment.md](app-deployment.md)
- Submit to App Store - see [app-store-submission.md](app-store-submission.md)
