# RIP-01: OpenAI-API Proxy with Cashu Payments

Defines the HTTP proxy interface forwarding OpenAI-compatible API requests, with per-request micropayment handling via Cashu tokens.

## Endpoints

### POST /v1/{path:path} e.g. /v1/chat/completions

Forward proxied requests to the upstream AI service at `UPSTREAM_BASE_URL`.
This is usually your llamacpp or vllm server.

### GET /v1/info

Get the node information including name, description, version, npub, mints, and connection URLs (HTTP/Onion).

### GET /v1/models

Get the list of supported models with their pricing information (both in USD and Sats).

### GET /v1/balance/create

Create a new balance (API key) from a Cashu token.

- **Params**: `initial_balance_token` (Cashu token to redeem)
- **Returns**: `{"api_key": "sk-...", "balance": 1000}`

### GET /v1/balance/info

Get the balance of the Cashu token and the associated auth key.

- **Headers**: `Authorization: Bearer <api-key>`

### POST /v1/balance/topup

Top up the balance of that temporary wallet with another Cashu token.

- **Headers**: `Authorization: Bearer <api-key>`
- **Body**: `{"cashu_token": "cashuA..."}`

### POST /v1/balance/refund

Recieve the full amount of the remaining balance as a Cashu token or to a Lightning Address.

- **Headers**: `Authorization: Bearer <api-key>`
- **Returns**: `{"token": "cashuA..."}` or `{"recipient": "user@domain.com"}`

#### Authorization

- Header: `Authorization: Bearer <cashu-token or api-key (sk-...)>`
- API keys: prefixed `sk-...`, validated against stored hashed keys.
- Cashu tokens: prefixed `cashuB...`, redeemed via Cashu mint. If a new token is presented, a new account is created.

#### Payment Flow

1. **Balance**: The cashu token is redeemed and the balance is credited internally (allows reuse of the same token for multiple requests).
2. **Api-Key**: Each balance has an associated API key (`sk-...`) that allows for management and top-ups.
3. **Token based pricing**: For chat/completions:
   - Calculate `max_cost` based on model context length or `max_tokens`.
   - Reserve `max_cost` from balance.
   - Forward request to upstream.
   - Parse upstream `usage.prompt_tokens` and `usage.completion_tokens`.
   - Compute actual cost.
   - Finalize charge (credit back difference from reserved amount) or refund excess.

#### Responses

- **Streaming** (`text/event-stream`): Proxy SSE chunks.
- **Non-Streaming** (`application/json`): Standard JSON response.

#### Error Codes

- `401 Unauthorized`: Missing or invalid API key/Cashu token.
- `402 Payment Required`: Insufficient balance.
- `403 Forbidden`: Invalid API key/Cashu token.
- `500 Internal Server Error`: Unexpected errors.

## Tor Routing

Every node can be deployed as a Tor hidden service. The `onion_url` is returned in `/v1/info`.

## Multi model support

Each node can serve multiple models.
These models are defined in `/v1/models` including node information and pricing.

## Upstream API

The upstream API is defined via `UPSTREAM_BASE_URL` or configured via the database (supporting multiple upstream providers like OpenRouter, local vLLM, etc.).

## X-Cashu Header

Every proxy supports `X-Cashu` headers as a direct payment protocol (without creating a persistent account).

1. Client sends request with `X-Cashu: <cashu-token>` header.
2. Proxy redeems the token.
3. Proxy calculates `max_cost` and verifies the token amount covers it.
4. Proxy forwards request to upstream (buffering the response).
5. Proxy calculates actual cost from usage.
6. Proxy calculates `refund_amount = original_amount - actual_cost`.
7. If `refund_amount > 0`, Proxy mints a new token.
8. Proxy sends response with `X-Cashu: <refund-token>` header containing the change.
