# RIP-01: OpenAI-API Proxy with Cashu Payments

Defines the HTTP proxy interface forwarding OpenAI-compatible API requests, with per-request micropayment handling via Cashu tokens.

## Endpoints

### POST /v1/{path:path} e.g. /v1/chat/completions

Forward proxied requests to the upstream AI service at `UPSTREAM_BASE_URL`.
This is usually your llamacpp or vllm server.

### GET /v1/wallet/info

Get the balance of the Cashu token and the associated auth key.

### POST /v1/wallet/topup

Top up the balance of that temporary wallet with another Cashu token.

### POST /v1/wallet/refund

Recieve the full amount of the remaining balance as a Cashu token.

#### Authorization

- Header: `Authorization: Bearer <api-key>` or `Bearer <cashu-token>`
- API keys: prefixed `sk-`, validated against stored hashed keys.
- Cashu tokens: prefixed `cashu`, redeemed via Cashu mint.

#### Payment Flow

1. **Balance**: The cashu token is redeemed and the balance is credited internally (allows reuse of the same token for multiple requests).
2. **Api-Key**: Each balance has an associated API key that allowes for top-ups.
3. **Token based pricing**: For chat/completions when `MODEL_BASED_PRICING`=true:
   - Parse upstream `usage.prompt_tokens` and `usage.completion_tokens`.
   - Compute `input_msats` = prompt_tokens/1000 × model.prompt_price.
   - Compute `output_msats` = completion_tokens/1000 × model.completion_price.
   - Charge or refund the difference via balance adjustment.
   - Emit SSE event `data: {"cost": {...}}` appended to stream.

#### Responses

- **Streaming** (`text/event-stream`): Proxy SSE chunks and inject a final cost event.
- **Non-Streaming** (`application/json`): Embed a top-level `"cost": {...}` field in JSON.

#### Error Codes

- `401 Unauthorized`: Missing or invalid API key/Cashu token.
- `402 Payment Required`: Insufficient balance.
- `502 Bad Gateway`: Upstream service unavailable.
- `500 Internal Server Error`: Unexpected errors.

## Tor Routing

Every node can be deployed as a Tor hidden service.

## Multi model support

Each node can serve multiple models.
These models are defined in the root `/` endpoint.
Including node information and pricing.

## Upstream API

The upstream API is defined in the `UPSTREAM_BASE_URL` environment variable which is authenticated using the `UPSTREAM_API_KEY` environment variable.

Optional the proxy can attach multiple upstream APIs and route using a algorithm to other proxy providers that list their node on nostr.

## X-Cashu Header

Every proxy supports X-Cashu headers as payment protocol. Instead of authenticating using the Bearer API Key header a valid cashu token can be send in the 'X-Cashu: cashuBxxxxx' Header which is then claimed, and a refund is send in the same X-Cashu Header inside the response with the change balance.

---

## TODO

- [x] HTTP proxy interface for OpenAI-compatible API requests
- [x] Integrate per-request micropayment handling via Cashu tokens
- [x] Endpoints:
  - [x] POST /v1/{path:path} (proxy to upstream AI service)
  - [x] GET /v1/wallet/info (get Cashu token balance and API key)
  - [x] POST /v1/wallet/topup (top up wallet with Cashu token)
  - [x] POST /v1/wallet/refund (refund remaining balance as Cashu token)
  - [x] GET /v1/models (full details of all supported models + sats pricing)
- [x] Authorization via Bearer API key or Cashu token
- [x] API key and Cashu token validation and redemption
- [x] Token-based pricing for chat/completions with MODEL_BASED_PRICING
- [x] SSE and JSON cost reporting in responses
- [x] Tor hidden service deployment support
- [x] Admin dashboard for monitoring and settings
- [ ] Routing to multiple upstream providers
- [ ] Optional usage tracking for prepaid balances if requested by user
- [ ] Routing to upstream routstr network based on marketplace algorithm
- [ ] automatic setup of public upstream model providers like openai, anthropic, ...
- [ ] publish nostr listing kind 38421 on startup
