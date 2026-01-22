---
name: app-database
description: Add Supabase database to a Whop app for custom data storage. Use when app needs to store custom data like votes, products, user profiles, or any app-specific entities.
impact: HIGH
impactDescription: Enables persistent data storage - required for most full-stack apps
metadata:
  tags: database, supabase, full-stack, storage, real-time
---

# Add Database (Supabase)

For apps that need custom data storage beyond Whop's native APIs.

## When to Use

- Custom data models (votes, products, user profiles)
- App-specific entities not covered by Whop APIs
- Real-time subscriptions needed
- Complex queries or relationships

## When NOT to Use

- Only need file storage → Use `client.files.upload()`
- Only need posts/comments → Use Whop Forums API
- Only need messages → Use Whop Chat API

## Step 1: Create Supabase Project

1. Go to [supabase.com](https://supabase.com) and sign up/login
2. Click "New Project"
3. Choose organization and enter project name
4. Set a strong database password (save it!)
5. Select region closest to your users
6. Click "Create new project" and wait for setup

## Step 2: Get Credentials

In Supabase Dashboard → Settings → API:

| Key | Where to find | Use |
|-----|---------------|-----|
| Project URL | Under "Project URL" | `SUPABASE_URL` |
| anon key | Under "Project API keys" | `NEXT_PUBLIC_SUPABASE_ANON_KEY` |
| service_role key | Under "Project API keys" | `SUPABASE_SERVICE_ROLE_KEY` (SECRET!) |

## Step 3: Install Dependencies

```bash
pnpm add @supabase/supabase-js
```

## Step 4: Environment Variables

```bash
# .env.local

# Server-side (SECRET - never expose)
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Client-side (safe for browser - for real-time only)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Step 5: Create Supabase Clients

### lib/supabase.ts (Server-side)

```typescript
import { createClient } from "@supabase/supabase-js";

if (!process.env.SUPABASE_URL || !process.env.SUPABASE_SERVICE_ROLE_KEY) {
  throw new Error("Missing Supabase environment variables");
}

export const supabaseAdmin = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY,
  { auth: { persistSession: false } }
);
```

### lib/supabase-client.ts (Client-side for real-time)

```typescript
"use client";

import { createClient } from "@supabase/supabase-js";

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabaseClient = createClient(supabaseUrl, supabaseAnonKey, {
  realtime: {
    params: { eventsPerSecond: 10 },
  },
});
```

## Step 6: Create Database Schema

In Supabase Dashboard → SQL Editor, run your schema:

```sql
-- Create items table with proper types
CREATE TABLE items (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  company_id TEXT NOT NULL,
  experience_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  status TEXT DEFAULT 'active',
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- CRITICAL: Index all columns used in WHERE clauses
-- Without indexes, queries do full table scans (100-1000x slower)
CREATE INDEX idx_items_company_id ON items(company_id);
CREATE INDEX idx_items_experience_id ON items(experience_id);
CREATE INDEX idx_items_user_id ON items(user_id);

-- Composite index for common query pattern
CREATE INDEX idx_items_exp_created ON items(experience_id, created_at DESC);

-- Enable Row Level Security (defense in depth)
ALTER TABLE items ENABLE ROW LEVEL SECURITY;

-- RLS Policy: Users can only access items in their experience
-- This is a safety net - always verify access in API routes too
CREATE POLICY "users_own_experience_items"
ON items FOR ALL
USING (
  experience_id = current_setting('app.experience_id', true)
);

-- Enable Realtime (optional)
ALTER PUBLICATION supabase_realtime ADD TABLE items;
```

## Schema Patterns for Whop Apps

ALWAYS include these columns for multi-tenancy:

| Column | Type | Purpose |
|--------|------|---------|
| `company_id` | TEXT | Isolate data by company (biz_xxx) |
| `experience_id` | TEXT | Isolate data by experience (exp_xxx) |
| `user_id` | TEXT | Track who created/owns the record |

## Step 7: Use in API Routes

MUST verify user token and check access before any database operation.

### app/api/items/route.ts

```typescript
import { NextRequest, NextResponse } from "next/server";
import { headers } from "next/headers";
import { whopsdk } from "@/lib/whop-sdk";
import { supabaseAdmin } from "@/lib/supabase";

export async function GET(request: NextRequest) {
  // MUST: Verify user first
  const headersList = await headers();
  const { userId } = await whopsdk.verifyUserToken(headersList);

  const { searchParams } = new URL(request.url);
  const experienceId = searchParams.get("experienceId");

  if (!experienceId) {
    return NextResponse.json({ error: "experienceId required" }, { status: 400 });
  }

  // MUST: Check user has access to this experience
  const { has_access } = await whopsdk.users.checkAccess(experienceId, { id: userId });
  if (!has_access) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  // Now safe to query - filter by experience_id
  const { data, error } = await supabaseAdmin
    .from("items")
    .select("id, title, description, status, user_id, created_at")
    .eq("experience_id", experienceId)
    .order("created_at", { ascending: false });

  if (error) {
    console.error("Database error:", error);
    return NextResponse.json({ error: "Database error" }, { status: 500 });
  }

  return NextResponse.json({ items: data });
}

export async function POST(request: NextRequest) {
  // MUST: Verify user first
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { experienceId, companyId, title, description } = body;

  // Validate input
  if (!experienceId || !companyId || !title) {
    return NextResponse.json({ error: "Missing required fields" }, { status: 400 });
  }

  // MUST: Check user has access
  const { has_access } = await whopsdk.users.checkAccess(experienceId, { id: userId });
  if (!has_access) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  const { data, error } = await supabaseAdmin
    .from("items")
    .insert({
      user_id: userId, // Use verified userId, not from body
      experience_id: experienceId,
      company_id: companyId,
      title,
      description,
    })
    .select("id, title, description, status, created_at")
    .single();

  if (error) {
    console.error("Database error:", error);
    return NextResponse.json({ error: "Database error" }, { status: 500 });
  }

  return NextResponse.json({ item: data });
}
```

## Step 8: Real-time Subscriptions (Optional)

```typescript
"use client";

import { useEffect, useState } from "react";
import { supabaseClient } from "@/lib/supabase-client";

export function useRealtimeItems(experienceId: string) {
  const [items, setItems] = useState<Item[]>([]);

  useEffect(() => {
    const channel = supabaseClient
      .channel(`items-${experienceId}`)
      .on(
        "postgres_changes",
        {
          event: "*",
          schema: "public",
          table: "items",
          filter: `experience_id=eq.${experienceId}`,
        },
        () => {
          // Refresh via API (validates access)
          fetchItems();
        }
      )
      .subscribe();

    fetchItems();

    return () => {
      supabaseClient.removeChannel(channel);
    };
  }, [experienceId]);

  async function fetchItems() {
    const res = await fetch(`/api/items?experienceId=${experienceId}`);
    if (res.ok) {
      const data = await res.json();
      setItems(data.items || []);
    }
  }

  return items;
}
```

## TypeScript Types

```typescript
// lib/types.ts
export interface Item {
  id: number;
  company_id: string;
  experience_id: string;
  user_id: string;
  title: string;
  description: string | null;
  status: string;
  metadata: Record<string, unknown> | null;
  created_at: string;
  updated_at: string;
}
```

## Security Checklist

- [ ] `SUPABASE_SERVICE_ROLE_KEY` is server-side only (NEVER in client code)
- [ ] All API routes verify user token FIRST
- [ ] All API routes check access to experience/company
- [ ] All queries filter by `experience_id` or `company_id`
- [ ] RLS enabled on all tables (defense in depth)
- [ ] Indexes on all columns used in WHERE clauses
- [ ] Select only needed columns (not `SELECT *`)

## Advanced Patterns

For complex database patterns, see the `supabase-postgres-best-practices` skill:
- Query optimization and indexing strategies
- RLS performance optimization
- Connection pooling
- Batch operations

## Reference

- [Supabase Docs](https://supabase.com/docs)
- [Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Realtime Subscriptions](https://supabase.com/docs/guides/realtime)
