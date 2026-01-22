---
name: input-sanitization
description: Sanitizing user input to prevent XSS and injection attacks
metadata:
  tags: security, sanitization, xss, validation
---

## Why Sanitize

User input can contain malicious content:
- XSS attacks (JavaScript injection)
- HTML injection
- SQL injection (if using raw queries)

Always sanitize before storing or displaying.

## Text Sanitization

```typescript
// lib/sanitize.ts

// Hoist RegExp for performance (don't create in loop)
const SCRIPT_PATTERN = /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi;
const EVENT_HANDLER_PATTERN = /\s*on\w+\s*=/gi;
const JAVASCRIPT_URL_PATTERN = /javascript:/gi;

export function sanitizeText(input: string): string {
  if (typeof input !== "string") {
    return "";
  }
  
  return input
    // Remove script tags
    .replace(SCRIPT_PATTERN, "")
    // Remove event handlers
    .replace(EVENT_HANDLER_PATTERN, "")
    // Remove javascript: URLs
    .replace(JAVASCRIPT_URL_PATTERN, "")
    // Trim whitespace
    .trim();
}

export function sanitizeHtml(input: string): string {
  if (typeof input !== "string") {
    return "";
  }
  
  // Escape HTML entities
  return input
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}
```

## Usage in API Routes

```typescript
import { sanitizeText } from "@/lib/sanitize";

export async function POST(request: NextRequest) {
  const headersList = await headers();
  const { userId } = await whopSdk.verifyUserToken(headersList);
  
  const body = await request.json();
  
  // Sanitize all user text input
  const sanitizedText = sanitizeText(body.text);
  const sanitizedTitle = sanitizeText(body.title);
  
  // Validate length after sanitization
  if (sanitizedText.length > 5000) {
    return NextResponse.json({ error: "Text too long" }, { status: 400 });
  }
  
  // Store sanitized data (example - use your own database)
  await db.items.create({
    data: {
      user_id: userId,
      text: sanitizedText,
      title: sanitizedTitle,
    },
  });
  
  return NextResponse.json({ success: true });
}
```

## URL Validation

```typescript
export function isValidUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    // Only allow http/https
    return ["http:", "https:"].includes(parsed.protocol);
  } catch {
    return false;
  }
}

// Usage
if (body.url && !isValidUrl(body.url)) {
  return NextResponse.json({ error: "Invalid URL" }, { status: 400 });
}
```

## ID Validation

```typescript
// Validate Whop ID format
export function isValidWhopId(id: string, prefix: string): boolean {
  if (typeof id !== "string") return false;
  if (!id.startsWith(`${prefix}_`)) return false;
  if (id.length < prefix.length + 5) return false;
  
  // Only alphanumeric after prefix
  const idPart = id.slice(prefix.length + 1);
  return /^[a-zA-Z0-9]+$/.test(idPart);
}

// Usage
if (!isValidWhopId(experienceId, "exp")) {
  return NextResponse.json({ error: "Invalid experience ID" }, { status: 400 });
}
```

## React Component Display

When displaying user-generated content in React:

```tsx
// SAFE: React auto-escapes by default
<p>{userText}</p>

// DANGEROUS: Never use dangerouslySetInnerHTML with user content
<div dangerouslySetInnerHTML={{ __html: userText }} /> // NEVER DO THIS

// If you must render HTML, sanitize first with DOMPurify
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userHtml) }} />
```

## Input Limits

Always enforce limits:

```typescript
const MAX_TEXT_LENGTH = 5000;
const MAX_TITLE_LENGTH = 200;
const MAX_URL_LENGTH = 2000;

if (body.text.length > MAX_TEXT_LENGTH) {
  return NextResponse.json({ error: "Text exceeds limit" }, { status: 400 });
}
```
