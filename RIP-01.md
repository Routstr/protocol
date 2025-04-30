# RIP-01: OpenAI-API Proxy with Cashu Payments

Defines the HTTP proxy interface forwarding OpenAI-compatible API requests, with per-request micropayment handling via Cashu tokens.

## Endpoints

### POST /{path:path}

Forward proxied requests to the upstream AI service at `UPSTREAM_BASE_URL`.

#### Authorization

- Header: `Authorization: Bearer <api-key>` or `Bearer <cashu-token>`
- API keys: prefixed `sk-`, validated against stored hashed keys.
- Cashu tokens: prefixed `cashu`, redeemed via Cashu L3 mint.

#### Payment Flow

1. **Validation**: Verify bearer key and ensure balance ≥ `COST_PER_REQUEST` msats.
2. **Charge Base**: Deduct `COST_PER_REQUEST` from user balance; record request count.
3. **Forward**: Send request upstream, applying `UPSTREAM_API_KEY` override if set.
4. **Token-Based Pricing (Optional)**: For chat/completions when `MODEL_BASED_PRICING`=true:
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

## Configuration

- `COST_PER_REQUEST`: Base cost in msats (env `COST_PER_REQUEST`).
- `MODEL_BASED_PRICING`: Enable token pricing (env `MODEL_BASED_PRICING`).
- `COST_PER_1K_INPUT_TOKENS` & `COST_PER_1K_OUTPUT_TOKENS`: Fallback pricing if `models.json` not present.
