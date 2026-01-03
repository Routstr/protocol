# RIP-08: Payment Methods

This document defines standard protocols for funding Routstr balances using methods other than direct Cashu tokens. While Cashu is the core internal wallet, these extensions allow clients to interact with providers using broader payment networks like Lightning and On-Chain Bitcoin.

## 1. Overview

To maintain the stateless, privacy-preserving properties of Routstr, all alternative payment methods function as **"Cashu Gateways"**:

1. The client pays via Lightning/On-Chain.
2. The provider's gateway receives the payment.
3. The gateway internally mints or credits a Cashu token/balance.
4. This balance is effectively used as the "bearer asset" for the balance, preserving the standard flow defined in RIP-03.

## 2. Lightning Network (Bolt11)

Allows clients to purchase ephemeral API access via standard Lightning invoices.

### 2.1. Endpoints

- **POST `/lightning/invoice`**:
  - **Purpose**: Create a new balance or top-up an existing one.
  - **Body**: `{"amount_sats": 1000, "purpose": "create"}`
  - **Response**: `{"invoice_id": "...", "bolt11": "lnbc...", "payment_hash": "..."}`
- **GET `/lightning/invoice/{invoice_id}/status`**:
  - **Polling**: Client checks status until `paid`.
  - **Response**: `{"status": "paid", "api_key": "sk-..."}`

### 2.2. Flow

1. Client requests an invoice for 1000 sats.
2. Provider generates a Bolt11 invoice (mapped internally to a Cashu mint quote).
3. Client pays the invoice via their Lightning wallet.
4. Provider detects payment -> Mints internal Cashu tokens -> Associates them with a new temporary API key (`sk-...`).
5. Client receives the `api_key` and uses it for standard `/v1/chat/completions` calls.

## 3. Nostr Wallet Connect (NotImplementedYet)

Allows for seamless, one-click or automated payments without leaving the client interface.

- **NIP-47**: The client connects to the user's NWC-enabled wallet.
- **Auto-Pay**: When a Routstr provider requests payment (e.g., via a 402 Payment Required or during the handshake), the client automatically requests the user's wallet to pay the Bolt11 invoice.
- **Benefit**: Provides a UX similar to a "streaming subscription" without custodial deposits.

## 4. Bolt12 (NotImplementedYet)

Routstr nodes should support Bolt12 offers for static, reusable payment codes.

- **Use Case**: A provider publishes a Bolt12 Offer in their Kind 38421 announcement.
- **Flow**: Client fetches invoice from Offer -> Pays -> Receives credentials via blinded path.

## 5. On-Chain Bitcoin (NotImplementedYet)

For high-value, long-term sessions (e.g., enterprise batch jobs).

- **Mechanism**: Provider generates a unique address.
- **Wait Time**: 1 confirmation required.
- **Conversion**: Funds are swapped to Cashu tokens once confirmed.

## 6. Compatibility with RIP-03

These methods are **additions** to the core protocol. A client using Lightning:

1. Pays Invoice -> Gets `api_key` (which represents the Cashu balance).
2. Uses `api_key` for requests.
3. **Refunds**: When the session ends, the refund (via `/v1/balance/refund`) is returned as a **Cashu Token**.
4. **Off-Ramp**: The client can then swap this Cashu token back to Lightning (melt) using any Cashu mint, effectively "withdrawing" their change to their Lightning wallet.
