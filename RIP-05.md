# RIP-05: Smart Clients & Token Management

Defines client-side behavior for automated token cycling, proxy/Tor usage, and provider optimization.

## Responsibilities

- Maintain local Cashu wallet and balance.
- Auto-redeem and split tokens into per-request msat units.
- Route inference requests through proxies or Tor to obfuscate origin.
- Continuously monitor discovery for better providers (price, latency, rating).

## Workflow

1. **Initialization**
   - Load master wallet token.
   - Pre-split tokens into request-sized shards.
2. **Request Execution**
   - Select provider based on weighted metrics (price, latency, rating).
   - Choose random proxy or Tor circuit.
   - Attach one token shard as payment credential.
   - Send inference request off-chain.
3. **Monitoring**
   - Track per-provider performance over time.
   - On token exhaustion, auto-top-up from wallet.
   - Adjust provider weights for future selection.

## Metrics & Adaptation

- Dynamic scoring: lower price & latency, higher rating favored.
- Decay older measurements; prioritize recent data.

## Security & Privacy

- Rotate Tor circuits per request.
- Limit request rate per provider to avoid fingerprinting.

---

_End of RIP specs._
