---
name: ui-frosted
description: Using Frosted UI components - Whop's design system
metadata:
  tags: ui, frosted-ui, components, design-system, radix
---

Frosted UI is Whop's design system built on Radix UI primitives and Tailwind CSS. It provides 60+ accessible components with automatic dark mode support.

## When to Use Frosted UI

**Recommended when:**
- Building apps displayed in the Whop iframe (visual consistency)
- Want automatic dark/light mode that matches Whop
- Need accessible components out of the box

**Alternatives are fine when:**
- Building standalone apps outside Whop
- Have existing component library (Shadcn, Radix, etc.)
- Need custom design system

## Installation

```bash
pnpm add @whop/react
```

The `@whop/react` package includes Frosted UI and all necessary dependencies.

## Common Components

### Buttons

```tsx
import { Button } from "@whop/react";

<Button variant="primary">Primary Action</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="destructive">Delete</Button>
<Button isLoading>Loading...</Button>
<Button disabled>Disabled</Button>
```

### Form Inputs

```tsx
import { Input, Textarea, Select } from "@whop/react";

<Input placeholder="Enter text..." />
<Input type="email" placeholder="Email" />
<Textarea placeholder="Long text..." rows={4} />

<Select>
  <Select.Trigger placeholder="Select option" />
  <Select.Content>
    <Select.Item value="a">Option A</Select.Item>
    <Select.Item value="b">Option B</Select.Item>
  </Select.Content>
</Select>
```

### Cards

```tsx
import { Card } from "@whop/react";

<Card className="p-4">
  <Card.Header>
    <Card.Title>Card Title</Card.Title>
    <Card.Description>Description text</Card.Description>
  </Card.Header>
  <Card.Content>
    {/* Content */}
  </Card.Content>
</Card>
```

### Dialogs

```tsx
import { Dialog, Button } from "@whop/react";

<Dialog>
  <Dialog.Trigger asChild>
    <Button>Open Dialog</Button>
  </Dialog.Trigger>
  <Dialog.Content>
    <Dialog.Title>Confirm Action</Dialog.Title>
    <Dialog.Description>Are you sure?</Dialog.Description>
    <Dialog.Footer>
      <Dialog.Close asChild>
        <Button variant="ghost">Cancel</Button>
      </Dialog.Close>
      <Button variant="primary">Confirm</Button>
    </Dialog.Footer>
  </Dialog.Content>
</Dialog>
```

### Toast Notifications

```tsx
import { toast } from "@whop/react";

// Success
toast.success("Operation completed!");

// Error
toast.error("Something went wrong");

// Custom
toast("Custom message", {
  description: "Additional details",
  duration: 5000,
});
```

## Using Other UI Libraries

If using Shadcn, Radix, or other libraries instead:

```tsx
// Shadcn/Radix work fine - just handle dark mode yourself
import { Button } from "@/components/ui/button"; // Shadcn
import * as Dialog from "@radix-ui/react-dialog"; // Radix directly

// Make sure to support dark mode for Whop iframe
<div className="dark:bg-gray-900 dark:text-white">
  {/* Your UI */}
</div>
```

## Tailwind Integration

Frosted UI includes a Tailwind plugin for theme colors:

```tsx
// Use Frosted color tokens (12-step scale)
<div className="bg-gray-2 text-gray-12 border-gray-6">
  Themed content
</div>

// Semantic colors
<span className="text-positive">Success</span>
<span className="text-negative">Error</span>
<span className="text-warning">Warning</span>
```

## Companion Skill Handoff

**For custom design beyond Frosted UI components:**
- → `frontend-design` for bold, distinctive UI aesthetics, custom layouts, animations
- → `web-design-guidelines` for UI/UX review and accessibility auditing

Frosted UI provides the building blocks; companion skills help create memorable experiences.

## Reference

- [Frosted UI Docs](https://docs.whop.com/developer/guides/frosted_ui)
- [Storybook](https://storybook.whop.com/) - Full component documentation
