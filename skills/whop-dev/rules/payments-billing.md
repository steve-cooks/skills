---
name: payments-billing
description: Setting up billing portal and saving payment methods
metadata:
  tags: payments, billing, subscriptions, portal
---

Set up billing portal and save payment methods for recurring charges.

## Billing Portal

Render a billing portal for users to manage their subscriptions:

```typescript
import { whopSdk } from "@/lib/whop-sdk";

// Get billing portal URL
const portal = await whopSdk.billingPortal.create({
  userId: "user_xxxxx",
  returnUrl: "https://yourapp.com/dashboard",
});

// Redirect user to portal
redirect(portal.url);
```

## Save Payment Method

Save a payment method for future charges:

```typescript
// Create setup intent
const setup = await whopSdk.setupIntents.create({
  userId: "user_xxxxx",
});

// Use setup.clientSecret on frontend with Stripe Elements
// to collect payment method
```

## Frontend Setup Form

```tsx
"use client";

import { loadStripe } from "@stripe/stripe-js";
import { Elements, PaymentElement, useStripe, useElements } from "@stripe/react-stripe-js";

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_KEY!);

function SetupForm({ clientSecret }: { clientSecret: string }) {
  const stripe = useStripe();
  const elements = useElements();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!stripe || !elements) return;
    
    const { error } = await stripe.confirmSetup({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/setup-complete`,
      },
    });
    
    if (error) {
      console.error(error.message);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      <button type="submit" disabled={!stripe}>
        Save Payment Method
      </button>
    </form>
  );
}

export function PaymentSetup({ clientSecret }: { clientSecret: string }) {
  return (
    <Elements stripe={stripePromise} options={{ clientSecret }}>
      <SetupForm clientSecret={clientSecret} />
    </Elements>
  );
}
```

## Charge Saved Method

```typescript
// Charge a saved payment method
const charge = await whopSdk.charges.create({
  userId: "user_xxxxx",
  amount: 1000, // $10.00 in cents
  currency: "usd",
  description: "One-time charge",
});
```

## Reference

- [Save Payment Methods](https://docs.whop.com/developer/guides/save-payment-methods)
- [Billing Portal Setup](https://docs.whop.com/developer/guides/setup-billing-portal)
