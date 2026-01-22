---
name: payments-transfers
description: Sending payouts to users and companies via Whop transfers
metadata:
  tags: payments, transfers, payouts, money
---

Transfers allow you to send money from your Whop balance to users and companies.

## Required Permission

Your app must request the `payout:transfer_funds` permission to create transfers.

## Create a Transfer

```typescript
import { whopsdk } from "@/lib/whop-sdk";

const transfer = await whopsdk.transfers.create({
  amount: 6.9,              // Amount in dollars (NOT cents) - this is $6.90
  currency: "usd",
  origin_id: "biz_xxxxx",   // Your app's owning company ID
  destination_id: "user_xxxxx", // User, company, or ledger account
  notes: "Revenue share from MyApp", // Max 50 characters
  idempotence_key: "unique_key_123", // Prevents duplicate transfers
});

console.log(transfer.id); // ctt_xxxxxxxxxxxxxx
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | number | Yes | Amount in **dollars** (decimal), e.g., `6.9` = $6.90 |
| `currency` | string | Yes | Currency code (e.g., `"usd"`) |
| `origin_id` | string | Yes | Source account (`biz_xxx`, `user_xxx`, or `ldgr_xxx`) |
| `destination_id` | string | Yes | Destination account (`biz_xxx`, `user_xxx`, or `ldgr_xxx`) |
| `idempotence_key` | string | No | Unique key to prevent duplicate transfers |
| `notes` | string | No | Notes for the transfer (max 50 chars) |
| `metadata` | object | No | Custom metadata to attach |

## Destination Types

| Prefix | Type | Example |
|--------|------|---------|
| `user_` | User account | `user_xxxxxxxxxxxxx` |
| `biz_` | Company account | `biz_xxxxxxxxxxxxx` |
| `ldgr_` | Ledger account | `ldgr_xxxxxxxxxxxxx` |

## API Route Example

```typescript
// POST /api/payout
import { whopsdk } from "@/lib/whop-sdk";
import { headers } from "next/headers";

const COMPANY_ID = process.env.WHOP_COMPANY_ID!; // Your app's company

export async function POST(request: NextRequest) {
  const [headersList, body] = await Promise.all([
    headers(),
    request.json(),
  ]);
  
  // Verify the requesting user is admin
  const { userId } = await whopsdk.verifyUserToken(headersList);
  const { has_access, access_level } = await whopsdk.users.checkAccess(
    COMPANY_ID,
    { id: userId }
  );
  
  if (!has_access || access_level !== "admin") {
    return NextResponse.json({ error: "Admin access required" }, { status: 403 });
  }
  
  const { recipientId, amount, paymentId } = body;
  
  if (!recipientId || !amount || amount <= 0) {
    return NextResponse.json({ error: "Invalid payout request" }, { status: 400 });
  }
  
  // Create transfer with idempotence key
  const transfer = await whopsdk.transfers.create({
    amount: amount, // In dollars (e.g., 10.50 = $10.50)
    currency: "usd",
    origin_id: COMPANY_ID,
    destination_id: recipientId,
    notes: "Payout from MyApp",
    // Derive from payment ID to prevent double payouts
    idempotence_key: `payout_${paymentId}`,
    metadata: {
      initiated_by: userId,
    },
  });
  
  return NextResponse.json({ 
    success: true, 
    transferId: transfer.id // ctt_xxxxx
  });
}
```

## Idempotence Keys

Use `idempotence_key` to prevent duplicate transfers. Keys are scoped to your app.

```typescript
// For revenue sharing, derive from payment ID
const transfer = await whopsdk.transfers.create({
  amount: 10.0,
  currency: "usd",
  origin_id: companyId,
  destination_id: userId,
  idempotence_key: `revenue_share_${paymentId}`, // Won't duplicate
});
```

If you call the API twice with the same `idempotence_key`, the second call won't create a duplicate transfer.

## Response

```typescript
{
  id: "ctt_xxxxxxxxxxxxxx",
  amount: 6.9,
  currency: "usd",
  created_at: "2023-12-01T05:00:00.401Z",
  fee_amount: 0,
  notes: "Revenue share from MyApp",
  metadata: {},
  origin_ledger_account_id: "ldgr_xxxxxxxxxxxxx",
  destination_ledger_account_id: "ldgr_xxxxxxxxxxxxx",
  origin: { id: "biz_xxxxx", name: "My Company" },
  destination: { id: "user_xxxxx", name: "John Doe", username: "johndoe42" }
}
```

## Withdrawals

Recipients can withdraw funds after completing KYC. Whop supports 241+ territories with multiple payout methods: ACH, Crypto, Venmo, CashApp, and more.

## Reference

- [Send Payouts Guide](https://docs.whop.com/developer/guides/payouts)
- [Create Transfer API](https://docs.whop.com/api-reference/transfers/create-transfer)
