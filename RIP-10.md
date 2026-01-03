# RIP-10: Marketplace Dynamics & Order Types

This document describes the emergent marketplace for AI compute created by the Routstr protocol. By using Nostr as a global, censorship-resistant message bus, Routstr effectively builds a decentralized **Order Book** where supply (Providers) and demand (Clients/Agents) meet without a central coordinator.

## 1. Market Structure

The marketplace consists of three primary order types, mirroring traditional financial markets but adapted for compute resources:

### 1.1. Limit Sell (Fixed Price Providers)

- **Actor**: Standard Providers (e.g., wrappers around OpenAI/Anthropic or stable GPU clusters).
- **Mechanism**: Providers publish Kind 38421 announcements with a fixed `price` per token.
- **Behavior**: "I will sell `gpt-4` inference at `10 sats/1k tokens`."
- **Stability**: These prices change infrequently (hours/days), providing predictable costs for real-time users.

### 1.2. Market Buy (Time-Sensitive Demand)

- **Actor**: Interactive Agents, Chat Clients, IDEs.
- **Mechanism**: Clients query the registry (RIP-02) and select the best available provider *right now*, accepting the current "Limit Sell" price.
- **Behavior**: "I need an answer *now*. I will pay the going rate of `10 sats`."
- **Priority**: Latency > Price.

### 1.3. Limit Buy (Batch Processing / Data Augmentation)

- **Actor**: Batch processing bots, data augmentation pipelines, fine-tuning scripts.
- **Mechanism**: These actors have tasks that are **not time-sensitive**. They hold off on execution until a provider appears with a price below their target threshold.
- **Behavior**: "I have 1M rows to process. I will buy `llama-3-70b` compute only if the price drops below `0.5 sats/1k tokens`."
- **Implementation**:
  - **Node Level**: Routstr nodes implement the **OpenAI Batch API** (`/v1/batches`), allowing users to submit file-based batch jobs. The node internally holds these jobs and executes them only when market conditions meet the user's price criteria.
  - **Client Level**: Specialized clients can also run locally, observing the Kind 38421 stream and dispatching requests one-by-one or in parallel only when a suitable provider is detected.
- **Effect**: This creates a "floor" price in the market, soaking up excess supply when providers lower prices.

## 2. Dynamic Pricing (Real-Time Spot Market)

To maximize utilization, providers with self-hosted hardware (e.g., H100 clusters, mining rigs repurposed for AI) can implement **Dynamic Pricing Algorithms**.

### 2.1. Cost-Based Pricing

- **Inputs**: Electricity cost, hardware amortization, cooling.
- **Logic**: `Price = (Energy_Cost + Margin) * Utilization_Factor`
- **Behavior**: If the GPU is idle, the price drops towards the marginal cost of electricity to attract "Limit Buy" batch jobs. If the GPU is busy, the price rises to capture high-value "Market Buy" traffic.

### 2.2. Market Sell (Real-Time Auctions)

- **Mechanism**: Providers publish high-frequency price updates via ephemeral Nostr events (or a dedicated pricing stream).
- **Discovery**: Smart Clients (RIP-03) and Aggregators monitor these streams.
- **Arbitrage**: If a provider drops their price significantly, automated batch jobs (Limit Buys) immediately route traffic to them, filling the idle capacity.

## 3. Marketplace Efficiency

This hybrid structure creates a highly efficient market:

- **High Availability**: "Limit Sell" providers ensure there is always capacity for urgent tasks.
- **High Utilization**: "Dynamic Pricing" + "Limit Buy" bots ensure that hardware rarely sits idle, as price drops automatically trigger deferrable workloads.
- **Price Discovery**: The intersection of these forces reveals the true global market price for a token of intelligence at any given second.
