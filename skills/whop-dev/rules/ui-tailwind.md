---
name: ui-tailwind
description: Tailwind CSS v4 setup with Frosted theme
metadata:
  tags: ui, tailwind, css, styling
---

## Tailwind v4 Configuration

### tailwind.config.ts

```typescript
import { frostedThemePlugin } from "@whop/react/tailwind";

export default {
  plugins: [frostedThemePlugin()],
};
```

### postcss.config.mjs

```javascript
export default {
  plugins: ["@tailwindcss/postcss"],
};
```

### globals.css

```css
/* CSS Layer order matters! */
@layer theme, base, frosted_ui, components, utilities;

@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/preflight.css" layer(base);
@import "tailwindcss/utilities.css" layer(utilities);
@import "@whop/react/styles.css" layer(frosted_ui);

@config '../tailwind.config.ts';
```

## Frosted Theme Colors

The plugin provides semantic color tokens:

```tsx
// Gray scale (1-12)
<div className="bg-gray-1">Lightest</div>
<div className="bg-gray-6">Border</div>
<div className="bg-gray-12">Darkest text</div>

// Semantic colors
<span className="text-positive">Success</span>
<span className="text-negative">Error</span>
<span className="text-warning">Warning</span>

// Interactive
<button className="bg-accent-9 hover:bg-accent-10">
  Action
</button>
```

## Common Patterns

### Card Pattern

```tsx
<div className="bg-gray-2 border border-gray-6 rounded-lg p-6 hover:border-gray-8 transition-colors">
  Card content
</div>
```

### Form Input

```tsx
<input
  className="w-full bg-gray-2 border border-gray-6 rounded-md px-3 py-2 text-gray-12 placeholder:text-gray-9 focus:border-accent-8 focus:outline-none"
  placeholder="Enter text..."
/>
```

### Hover States

```tsx
<button className="bg-gray-3 hover:bg-gray-4 active:bg-gray-5 transition-colors">
  Hover me
</button>
```

## Dependencies

```json
{
  "devDependencies": {
    "@tailwindcss/postcss": "^4",
    "tailwindcss": "^4"
  }
}
```

## Next.js Config

For optimal bundle size, add package imports optimization:

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  experimental: {
    optimizePackageImports: ["@whop/react", "frosted-ui"],
  },
};

export default nextConfig;
```

## Important

- Layer order in CSS is critical - don't reorder
- Use `@config` directive to link tailwind.config.ts
- Frosted colors auto-adjust for dark/light mode
- Prefer semantic colors over hardcoded values
