# RIP-03: Clients

This document defines the standard behavior and architectural requirements for a Routstr client. A Routstr client acts as the bridge between user applications (agents, IDEs, scripts) and the distributed network of AI service providers.

The client is designed to be a drop-in replacement for standard AI SDKs while handling the complexities of decentralized discovery, payment, and authentication transparently.

## 1. OpenAI-API Backwards Compatibility

Routstr clients MUST maintain high fidelity with the OpenAI API specification to ensure broad compatibility with the existing ecosystem of tools.

- **Interface parity**: The client implements standard endpoints (`/v1/chat/completions`, `/v1/embeddings`, `/v1/models`) matching the OpenAI schema.
- **Drop-in replacement**: Users can utilize the official `openai` Python/Node.js library by simply changing the `base_url` to the Routstr client's local proxy address (e.g., `http://localhost:xxxx/v1`) and inserting a cashu token as api key.
- **Transparent Proxying**: The client translates standard requests into Routstr-compatible network calls, injecting necessary headers for payment and routing without altering the request body structure.

## 2. Ephemeral Authentication (Cashu-as-Key)

Authentication and funding are unified into a single mechanism. The client passes either a temporary balance key (`sk-...`) or a Cashu token (`cashuA...`) in the `Authorization` header.

For the full specification of authentication methods and token types, refer to **[RIP-01: Authorization](RIP-01.md#authorization)**.

- **Implicit Session**: The provider accepts this token as both payment and a temporary identity. The token's secret effectively acts as the account identifier for the duration of the operation.
- **Privacy by Default**:
  - Upon request completion, the provider deducts the cost.
  - The **entire remaining balance is immediately refunded** as a *new* Cashu token returned to the client in the response headers or body.
  - Because Cashu uses Chaumian blind signatures, this new token is cryptographically unlinkable to the input token.
  - The next request uses this fresh token, ensuring that every interaction appears to come from a completely new, unrelated user.

## 3. Wallet Management & Transaction Cycle

The client manages an internal Cashu wallet to handle the "Overpay-then-Refund" lifecycle required for stateless, trustless operation.

- **Minimum Required Balance (Max Cost)**:
  - Providers publish a `max_cost` for each model (via `/v1/models`). This represents the worst-case cost for a request (max context + max output).
  - This `max_cost` serves as the **Minimum Required Balance** for any interaction.
  - The client MUST ensure the Cashu token used for authentication has a value `>= max_cost`.

- **Token Selection & Consolidation**:
  - Before a request, the client scans its wallet for a token satisfying the `max_cost` requirement.
  - If no single token is sufficient, the client automatically performs a self-payment (mint interaction) to merge smaller tokens into a single token of sufficient value.

- **Refund Cycle**:
  - Immediately after the API request completes, the client calculates the remaining balance from the ephemeral session.
  - The client calls `/v1/balance/refund` to reclaim the unspent funds.
  - The provider returns a *new* Cashu token, which the client stores back in its wallet (or NIP-60 storage), completing the cycle.

## 4. Node Discovery

The client is responsible for discovering, filtering, and selecting providers from the decentralized network.

- **Nostr Ingestion**: The client subscribes to Nostr relays to listen for Kind `38421` (Provider Advertisement) events.
- **Dynamic Registry**: It maintains an in-memory, real-time registry of active providers.
- **Smart Routing**:
  - **Filtering**: Filters nodes based on model availability (e.g., "DeepSeek-R1"), max price, and region.
  - **Scoring**: Ranks providers by latency, reliability scores (from RIP-04 audits), and social graph trust.
- **Failover**: If a request to the selected provider fails, the client automatically retries with the next best provider in the registry, transparent to the user.
