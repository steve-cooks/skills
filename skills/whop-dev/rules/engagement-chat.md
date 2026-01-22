---
name: engagement-chat
description: Sending and receiving chat messages via Whop SDK
metadata:
  tags: chat, messages, engagement, realtime
---

## Required Permissions

- `chat:read` - Read chat messages
- `chat:write` - Send messages

## Send Chat Message

```typescript
import { whopsdk } from "@/lib/whop-sdk";

const message = await whopsdk.messages.create({
  channel_id: "channel_xxxxx",
  content: "Hello from my app!",
});

console.log(message.id); // Message ID
```

## List Messages

```typescript
// List messages in a channel with pagination
for await (const message of whopsdk.messages.list({
  channel_id: "channel_xxxxx",
})) {
  console.log(message.content, message.created_at);
}
```

## Send with Attachments

```typescript
// Upload file first
const file = await whopsdk.files.create({
  file: imageBuffer,
  filename: "photo.jpg",
});

// Send with attachment
const message = await whopsdk.messages.create({
  channel_id: "channel_xxxxx",
  content: "Check out this image!",
  attachments: [{ id: file.id }],
});
```

## API Route Example

```typescript
// POST /api/send-message
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { channelId, content } = body;
  
  // Validate input
  if (!channelId || !content?.trim()) {
    return NextResponse.json({ error: "Invalid input" }, { status: 400 });
  }
  
  // Verify user has access to the channel's experience
  // (Channels belong to experiences)
  
  const message = await whopsdk.messages.create({
    channel_id: channelId,
    content: content.trim(),
  });
  
  return NextResponse.json({ messageId: message.id });
}
```

## List Chat Channels

```typescript
// List channels in an experience
for await (const channel of whopsdk.chatChannels.list({
  experience_id: "exp_xxxxx",
})) {
  console.log(channel.id, channel.name);
}
```

## Get Channel Details

```typescript
const channel = await whopsdk.chatChannels.retrieve("channel_xxxxx");
console.log(channel.name, channel.description);
```

## Support Channels

Support channels are for 1-on-1 support conversations:

```typescript
// List support channels
for await (const channel of whopsdk.supportChannels.list({
  experience_id: "exp_xxxxx",
})) {
  console.log(channel.id, channel.user);
}
```

## Message Object

```typescript
{
  id: "msg_xxxxx",
  content: "Hello world",
  created_at: "2023-12-01T05:00:00.401Z",
  user: {
    id: "user_xxxxx",
    username: "johndoe42",
    name: "John Doe"
  },
  attachments: [
    {
      id: "file_xxxxx",
      url: "https://...",
      filename: "photo.jpg"
    }
  ]
}
```

## Client Component

```typescript
"use client";

import { useState } from "react";
import { Button, Input } from "@whop/react";

export function ChatInput({ channelId }: { channelId: string }) {
  const [content, setContent] = useState("");
  const [sending, setSending] = useState(false);

  const handleSend = async () => {
    if (!content.trim()) return;
    
    setSending(true);
    try {
      await fetch("/api/send-message", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ channelId, content }),
      });
      
      setContent("");
    } finally {
      setSending(false);
    }
  };

  return (
    <div className="flex gap-2">
      <Input
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Type a message..."
        onKeyDown={(e) => e.key === "Enter" && handleSend()}
      />
      <Button onClick={handleSend} disabled={sending || !content.trim()}>
        Send
      </Button>
    </div>
  );
}
```

## Reference

- [Chat Guide](https://docs.whop.com/developer/guides/chat)
- [Create Message API](https://docs.whop.com/api-reference/messages/create-message)
- [List Messages API](https://docs.whop.com/api-reference/messages/list-messages)
