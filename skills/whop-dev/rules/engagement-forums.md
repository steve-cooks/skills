---
name: engagement-forums
description: Creating and managing forum posts via Whop SDK
metadata:
  tags: forums, posts, comments, engagement
---

## Required Permission

Your app must request the `forum:post:create` permission to create posts.

## Create Forum Post

Forum posts are created within an **experience** (not a forum directly).

```typescript
import { whopsdk } from "@/lib/whop-sdk";

const post = await whopsdk.forumPosts.create({
  experience_id: "exp_xxxxx", // Required: the experience to post in
  title: "Welcome to the community!",
  content: "This is the post content with **markdown** support.",
});

console.log(post.id); // Post ID
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `experience_id` | string | Yes | The experience to create the post in |
| `title` | string | No | Post title (visible even if paywalled) |
| `content` | string | No | Markdown content (hidden if paywalled) |
| `attachments` | object[] | No | File attachments `[{ id: "file_xxx" }]` |
| `poll` | object | No | Poll configuration |
| `parent_id` | string | No | Parent post ID for replies/comments |
| `is_pinned` | boolean | No | Pin post to top |

## List Forum Posts

```typescript
// List posts in an experience
for await (const post of whopsdk.forumPosts.list({
  experience_id: "exp_xxxxx",
})) {
  console.log(post.title, post.content);
}
```

## Get Single Post

```typescript
const post = await whopsdk.forumPosts.retrieve("post_id");
```

## Create Comment (Reply)

Comments are posts with a `parent_id`:

```typescript
const comment = await whopsdk.forumPosts.create({
  experience_id: "exp_xxxxx",
  parent_id: "parent_post_id", // The post being replied to
  content: "Great post! Thanks for sharing.",
});
```

## With Attachments

```typescript
// Upload file first
const file = await whopsdk.files.create({
  file: imageBuffer,
  filename: "screenshot.png",
});

// Attach to post
const post = await whopsdk.forumPosts.create({
  experience_id: "exp_xxxxx",
  title: "Check out this screenshot",
  content: "Here's what I found",
  attachments: [{ id: file.id }],
});
```

## With Poll

```typescript
const post = await whopsdk.forumPosts.create({
  experience_id: "exp_xxxxx",
  title: "What feature should we build next?",
  content: "Vote for your favorite!",
  poll: {
    options: [
      { text: "Dark mode" },
      { text: "Mobile app" },
      { text: "API improvements" },
    ],
    allow_multiple: false,
    ends_at: "2024-12-31T23:59:59Z", // Optional end date
  },
});
```

## Pinned Posts

```typescript
const post = await whopsdk.forumPosts.create({
  experience_id: "exp_xxxxx",
  title: "Important Announcement",
  content: "This will be pinned to the top.",
  is_pinned: true,
});
```

## Post Response

```typescript
{
  id: "post_xxxxx",
  title: "Post Title",
  content: "Markdown content",
  created_at: "2023-12-01T05:00:00.401Z",
  updated_at: "2023-12-01T05:00:00.401Z",
  is_edited: false,
  is_poster_admin: true,
  is_pinned: false,
  parent_id: null,
  user: {
    id: "user_xxxxx",
    username: "johndoe42",
    name: "John Doe"
  },
  view_count: 42,
  like_count: 10,
  comment_count: 5
}
```

## API Route Example

```typescript
// POST /api/forum/create-post
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";

export async function POST(request: NextRequest) {
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { experienceId, title, content } = body;
  
  // Verify user has access to the experience
  const { has_access } = await whopsdk.users.checkAccess(
    experienceId,
    { id: userId }
  );
  
  if (!has_access) {
    return NextResponse.json({ error: "Access denied" }, { status: 403 });
  }
  
  const post = await whopsdk.forumPosts.create({
    experience_id: experienceId,
    title,
    content,
  });
  
  return NextResponse.json({ postId: post.id });
}
```

## Reference

- [Forums Guide](https://docs.whop.com/developer/guides/forums)
- [Create Forum Post API](https://docs.whop.com/api-reference/forum-posts/create-forum-post)
- [List Forum Posts API](https://docs.whop.com/api-reference/forum-posts/list-forum-posts)
