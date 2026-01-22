---
name: app-store-submission
description: Submit your Whop app to the App Store for public listing. Use when ready to publish your app for other Whop creators to discover and install.
metadata:
  tags: app-store, submission, review, publish
---

# Submit to Whop App Store

Get your app listed on the Whop App Store for other creators to discover.

## Before You Start

**You don't need to be listed to use your app.** You can:
- Share your install link directly
- Link to it from your website
- Use it privately on your own Whops

Only submit if you want public discovery.

## Step 1: Complete Your Branding

In Whop Developer Dashboard → Your App → **App Details** tab:

### Required

| Item | Requirements |
|------|--------------|
| **App Icon** | Square image, clear at small sizes |
| **App Name** | Clear, descriptive name |
| **Short Description** | 1-2 sentences explaining what it does |
| **Category** | Select appropriate app store category |

### In Discover Tab

| Item | Requirements |
|------|--------------|
| **Demo Video** | 10-20 second video showing app in use |
| **Screenshots** | 2-3 screenshots of key features |
| **Full Description** | Detailed explanation of features and benefits |

## Step 2: Review Criteria Checklist

Your app must meet ALL criteria to pass review:

### Functionality
- [ ] App works end-to-end without bugs
- [ ] All features are complete (no placeholders)
- [ ] App loads quickly and performs well

### Authentication
- [ ] Uses Whop authentication (no separate login)
- [ ] Doesn't ask for permissions it doesn't need

### Design
- [ ] Works in both light and dark mode
- [ ] Looks polished and professional
- [ ] Uses Frosted UI or consistent design system

### Security
- [ ] Proper access control (members can't see admin data)
- [ ] No exposed API keys or secrets

### User Experience
- [ ] Clear value proposition for creators
- [ ] Not just a redirect to external platform
- [ ] Delivers measurable, useful outcome

## Step 3: Test Thoroughly

Before submitting, test:

1. **Fresh Install**: Install on a new Whop, verify setup flow
2. **Light/Dark Mode**: Toggle theme, check for issues
3. **Different Users**: Test as admin and as member
4. **All Features**: Every button, form, and flow works
5. **Error States**: Handle errors gracefully

## Step 4: Submit for Review

1. Go to your app in Whop Developer Dashboard
2. Navigate to **Discover** tab
3. Verify all branding is complete
4. Click **Publish to App Store**
5. Wait for review (usually a few days)

## Common Rejection Reasons

### Design Issues
- White flash in dark mode
- Inconsistent styling
- Broken layouts

**Fix:** Use Frosted UI, test dark mode, add theme script to layout

### Incomplete Features
- Placeholder text or images
- "Coming soon" features
- Demo/concept without real functionality

**Fix:** Complete all features before submitting

### Authentication Problems
- Asking users to sign in again
- Requesting unnecessary permissions

**Fix:** Use Whop SDK authentication only

### Access Control
- Members can see admin settings
- Data leaking between companies

**Fix:** Verify access level in all routes

### External Redirects
- App is just a link to external site
- Core functionality outside Whop

**Fix:** Build functionality within the Whop iframe

## After Approval

1. You'll receive a DM from Whop
2. Your app appears in the App Store
3. Share on social media to drive installs
4. Monitor for user feedback

## Preview Your Listing

Before submitting, preview at:
```
https://whop.com/apps/<your_app_id>
```

## Tips for Success

1. **Clear Value**: Explain exactly what problem you solve
2. **Quality Video**: Show the app in action, not just slides
3. **Real Screenshots**: Actual app UI, not mockups
4. **Test Everything**: Reviewers will try to break it
5. **Polish Details**: Small things matter (loading states, error messages)

## Need Help?

- [Whop Developer Docs](https://docs.whop.com/developer/getting-started)
- [Whop Discord](https://discord.gg/whop) - #developers channel
