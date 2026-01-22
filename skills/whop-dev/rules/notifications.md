---
name: notifications
description: Sending push notifications to Whop users
metadata:
  tags: notifications, push, engagement, messaging
---

## Required Permission

Your app must request the `notification:create` permission.

## Creating Notifications

Send push notifications to users in an experience or company.

### Notify Experience Members

```typescript
import { whopsdk } from "@/lib/whop-sdk";

await whopsdk.notifications.create({
  experience_id: "exp_xxxxx",
  title: "New Content Available",
  content: "Check out the latest update!",
});
```

### Notify Company Team

```typescript
// Team members only (admins/moderators)
await whopsdk.notifications.create({
  company_id: "biz_xxxxx",
  title: "Admin Alert",
  content: "New action required",
});
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `experience_id` | string | One of | Target experience members |
| `company_id` | string | One of | Target company team members |
| `title` | string | Yes | Notification title |
| `content` | string | Yes | Notification body text |
| `user_ids` | string[] | No | Only send to specific users (must be in scope) |
| `icon_user_id` | string | No | User whose profile pic is used as icon |
| `subtitle` | string | No | Optional subtitle |

## Target Specific Users

```typescript
// Only notify specific users within the experience
await whopsdk.notifications.create({
  experience_id: "exp_xxxxx",
  title: "You have a new message",
  content: "Someone replied to your question",
  user_ids: ["user_xxxxx", "user_yyyyy"],
});
```

## Custom Icon

```typescript
// Use a specific user's avatar as the notification icon
await whopsdk.notifications.create({
  experience_id: "exp_xxxxx",
  title: "New reply from John",
  content: "John replied to your post",
  icon_user_id: "user_xxxxx", // John's user ID
});
```

## API Route Example

```typescript
// app/api/admin/notify-members/route.ts
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { experienceId, companyId, title, message } = body;
  
  // Verify admin access
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    companyId,
    { id: userId }
  );
  
  if (!has_access || access_level !== "admin") {
    return NextResponse.json({ error: "Admin required" }, { status: 403 });
  }
  
  // Send notification
  await whopsdk.notifications.create({
    experience_id: experienceId,
    title,
    content: message,
  });
  
  return NextResponse.json({ success: true });
}
```

## Client Component

```typescript
"use client";

import { useState } from "react";
import { Button, Input, Textarea } from "@whop/react";

export function NotifyMembersButton({ 
  experienceId, 
  companyId 
}: { 
  experienceId: string;
  companyId: string;
}) {
  const [title, setTitle] = useState("");
  const [message, setMessage] = useState("");
  const [loading, setLoading] = useState(false);

  const handleNotify = async () => {
    if (!title.trim() || !message.trim()) return;
    
    setLoading(true);
    try {
      const res = await fetch("/api/admin/notify-members", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ experienceId, companyId, title, message }),
      });
      
      if (!res.ok) throw new Error("Failed to send");
      
      setTitle("");
      setMessage("");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      <Input 
        placeholder="Notification title" 
        value={title}
        onChange={(e) => setTitle(e.target.value)}
      />
      <Textarea 
        placeholder="Message content"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <Button onClick={handleNotify} disabled={loading}>
        {loading ? "Sending..." : "Notify All Members"}
      </Button>
    </div>
  );
}
```

## Best Practices

- **Rate limit** notification sends to prevent spam
- **Admin only** - verify access before sending
- **Meaningful content** - don't spam users
- **Target specific users** when possible using `user_ids`
- **Custom icons** - use `icon_user_id` for personalized notifications

## Reference

- [Push Notifications Guide](https://docs.whop.com/developer/guides/notifications)
- [Create Notification API](https://docs.whop.com/api-reference/notifications/create-notification)
