---
name: sdk-iframe
description: Using useIframeSdk for client-side Whop interactions
metadata:
  tags: sdk, iframe, client, react
---

The `useIframeSdk` hook provides client-side communication with the Whop platform.

## Setup

Add the provider to your root layout:

```typescript
// app/layout.tsx
import { WhopIframeSdkProvider } from "@whop/react";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <WhopIframeSdkProvider>{children}</WhopIframeSdkProvider>
      </body>
    </html>
  );
}
```

## Using the Hook

```typescript
"use client";

import { useIframeSdk } from "@whop/react";

export function MyComponent() {
  const iframeSdk = useIframeSdk();
  
  // Use SDK methods...
}
```

## Common Operations

### Open Checkout

```typescript
const handlePurchase = async () => {
  // First create checkout config on server
  const res = await fetch("/api/create-checkout", { method: "POST" });
  const { checkoutConfigurationId } = await res.json();
  
  // Then open checkout modal
  iframeSdk.openCheckout({ checkoutConfigurationId });
};
```

### Get Current User

```typescript
const user = await iframeSdk.getCurrentUser();
// { id: "user_xxxxx", username: "johndoe42", ... }
```

### Navigate Within Whop

```typescript
// Navigate to a Whop page
iframeSdk.navigateTo("/products");

// Navigate to specific experience
iframeSdk.navigateTo(`/experiences/${experienceId}`);
```

### Open External URL

```typescript
// Open in new tab (external links)
iframeSdk.openUrl("https://example.com");
```

### Open User Profile

```typescript
// Open a user's profile modal
iframeSdk.openUserProfile({ userId: "user_xxxxx" });
```

### Show Toast Notification

```typescript
iframeSdk.showToast({
  type: "success", // "success" | "error" | "info" | "warning"
  message: "Operation completed!",
});
```

### Get Theme

```typescript
const theme = iframeSdk.getTheme();
// "light" | "dark"
```

## Full Example: Purchase Button

```typescript
"use client";

import { useIframeSdk } from "@whop/react";
import { useState } from "react";
import { Button } from "@whop/react";

export function PurchaseButton({ 
  itemId, 
  price 
}: { 
  itemId: string; 
  price: number;
}) {
  const iframeSdk = useIframeSdk();
  const [loading, setLoading] = useState(false);
  
  const handleClick = async () => {
    setLoading(true);
    try {
      // Create checkout on server
      const res = await fetch("/api/purchase", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ itemId, price }),
      });
      
      if (!res.ok) throw new Error("Failed to create checkout");
      
      const { checkoutConfigurationId } = await res.json();
      
      // Open Whop checkout modal
      iframeSdk.openCheckout({ checkoutConfigurationId });
    } catch (error) {
      iframeSdk.showToast({
        type: "error",
        message: "Failed to start checkout",
      });
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <Button onClick={handleClick} disabled={loading}>
      {loading ? "Processing..." : `Buy for $${price}`}
    </Button>
  );
}
```

## Theme-Aware Components

```typescript
"use client";

import { useIframeSdk } from "@whop/react";

export function ThemeAwareCard({ children }: { children: React.ReactNode }) {
  const iframeSdk = useIframeSdk();
  const theme = iframeSdk.getTheme();
  
  return (
    <div className={theme === "dark" ? "bg-gray-800" : "bg-white"}>
      {children}
    </div>
  );
}
```

## Important

- `useIframeSdk` is client-side only - use in Client Components (`"use client"`)
- The hook only works when app is running in Whop's iframe
- For server-side operations, use `whopsdk` from `@whop/sdk`
- Always test in the Whop iframe environment, not standalone
- Wrap your app with `WhopIframeSdkProvider` in the root layout

## Reference

- [Iframe SDK Guide](https://docs.whop.com/developer/guides/iframe)
