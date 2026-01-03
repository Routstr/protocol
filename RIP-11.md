# RIP-11: Specialized Nodes

This document defines standard configuration profiles (or specialized implementations) of the Routstr node software. While the core software is general-purpose, these profiles optimize for specific roles within the network ecosystem.

## 1. General Node (Default)

The standard "full stack" deployment.

- **Function**: Acts as both a Buyer (routing to upstreams) and a Seller (advertising to the network).
- **Use Case**: Small ISPs, enthusiasts, or startups running a comprehensive gateway.
- **Components Active**: All (Wallet, Discovery, Proxy, Advertisement, Admin API).

## 2. Personal Node (Client Gateway)

A private, local-only node designed to serve a single user or organization.

- **Function**: Acts exclusively as a **Buyer** / Client.
- **Visibility**: Does NOT publish advertisements (Kind 38421) or listen on public ports.
- **Features**:
  - **Local Proxy**: Exposes `localhost:8000/v1` for local apps.
  - **Unlimited Key**: Provides a static "Master API Key" (skips per-request funding) for the owner's convenience. The node handles all wallet/funding logic internally.
  - **Privacy**: The node runs locally, so no 3rd party sees the user's IP or request patterns (requests are routed through the network via Tor/Proxy if configured).
  - **Extra Features**: Allows for additional features for companies like usage tracking, custom pricing, etc.

## 3. Worker Node (Compute Provider)

A node optimized solely for **selling** raw compute power from local hardware.

- **Function**: Acts exclusively as a **Seller**.
- **Visibility**: Advertises presence and pricing.
- **Features**:
  - **No Recursive Routing**: It does not proxy to other providers; it *is* the provider.
  - **Dynamic Pricing**: Runs specialized logic (RIP-10) to adjust prices in real-time based on hardware telemetry (GPU temp, utilization) and environmental costs (electricity spot price).
  - **Lean Stack**: May run a stripped-down implementation (e.g., Rust/Go) for maximum throughput and minimal overhead, interacting directly with inference engines like vLLM or Ollama.

## 4. Routing Node (Aggregator)

A node focused on traffic management, reliability, and arbitrage.

- **Function**: Acts as a high-volume **Reseller**.
- **Visibility**: Advertises highly reliable, stable endpoints.
- **Features**:
  - **No Local Compute**: Does not run local models.
  - **Recursive Routing**: Its primary value add is **smart routing** (RIP-06). It maintains vast, high-performance web-of-trust indexes to find the best upstream nodes for every request.
  - **Load Balancing**: Aggregates traffic from many users and distributes it across many Worker Nodes, smoothing out volatility.
  - **Verification**: May run active Auditing (RIP-04) and Evaluations (RIP-09) to ensure the quality of its downstream supply chain.
