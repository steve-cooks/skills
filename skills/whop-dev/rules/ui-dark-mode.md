---
name: ui-dark-mode
description: Preventing white flash in dark mode
metadata:
  tags: ui, dark-mode, theme, flash
---

Whop supports dark and light themes. Prevent the white flash (FOUC) with this pattern.

## Root Layout Setup

```typescript
// app/layout.tsx
import { WhopApp } from "@whop/react/components";
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "My Whop App",
  description: "App description",
};

// Inline script to set theme before render
const themeScript = `(function(){try{var c=document.cookie.match(/whop-frosted-theme=appearance:(?<a>light|dark)/);var t=c?c.groups.a:(window.matchMedia('(prefers-color-scheme:dark)').matches?'dark':'light');document.documentElement.classList.add(t);document.documentElement.style.colorScheme=t;if(t==='dark'){document.documentElement.style.backgroundColor='#111';}}catch(e){}})();`;

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <head>
        <script dangerouslySetInnerHTML={{ __html: themeScript }} />
      </head>
      <body>
        <WhopApp appearance="inherit">{children}</WhopApp>
      </body>
    </html>
  );
}
```

## Global CSS

```css
/* app/globals.css */
@layer theme, base, frosted_ui, components, utilities;

@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);
@import "@whop/react/styles.css" layer(frosted_ui);

@config '../tailwind.config.ts';

/* Hide until theme is applied - prevents flash */
html:not(.light):not(.dark) {
  visibility: hidden;
}

/* Force dark background immediately */
html.dark {
  background-color: #111 !important;
}

html.dark body {
  background-color: #111 !important;
}

body {
  background: var(--background);
  color: var(--foreground);
}
```

## How It Works

1. **Inline script** runs before React hydration
2. Reads theme from Whop cookie or system preference
3. Adds `light` or `dark` class to `<html>`
4. CSS hides content until class is applied
5. Dark mode background set immediately to prevent flash

## Using Theme in Components

```typescript
"use client";

import { useIframeSdk } from "@whop/react";

export function ThemeAwareComponent() {
  const iframeSdk = useIframeSdk();
  const theme = iframeSdk.getTheme(); // "light" | "dark"
  
  return (
    <div className={theme === "dark" ? "bg-gray-900" : "bg-white"}>
      Theme-aware content
    </div>
  );
}
```

## Important

- The inline script must be in `<head>` before body renders
- Use `suppressHydrationWarning` on `<html>` to prevent React warnings
- `WhopApp` with `appearance="inherit"` respects the set theme
- Never use `next/themes` - use Whop's built-in theme system
