# RIP-06: Routing Algorithms

This document defines standard algorithms for provider selection, scoring, and failover. These algorithms are shared across the ecosystem—used by smart clients for local selection and by routing nodes (aggregators) for recursive request handling.

## 1. Goal

The goal is to deterministically or probabilistically select the optimal provider for a given request from a potentially large set of discovered nodes. "Optimal" is defined by user preferences (price vs. speed vs. quality) and objective network metrics.

## 2. Inputs

Algorithms consume data from three primary sources:

1. **Announcement Events (Kind 38421)**:
    - `geo`: Geolocation tags.
    - `endpoint`: The base URL to fetch detailed info.
2. **Provider Info Endpoint (/v1/info & /v1/models)**:
    - `price`: Cost per token/request (fetched live).
    - `models`: Supported models list (fetched live).
3. **Audit/Evaluation Events**:
    - `uptime`: Reliability score.
    - `latency`: Measured response time.
    - `quality`: Benchmark scores from RIP-09 evals.
4. **Social Graph (Web of Trust)**:
    - `trust_score`: Derived from the user's social graph (distance to provider pubkey).

## 3. Selection Algorithms

### 3.1. Basic Filtering (Hard Constraints)

First, the candidate pool is filtered by hard requirements:

- **Model Match**: Provider must support the requested model (e.g., `deepseek-r1`).
- **Max Price**: Advertised price <= User's `max_price` budget.
- **Min Trust**: Provider pubkey must be within `N` hops of the user's web of trust (optional).

### 3.2. Weighted Scoring (Soft Constraints)

Once hard constraints are met, candidates are scored to determine the optimal choice. Different client implementations may prioritize different metrics based on user needs. Common scoring profiles include:

- **Price-Sensitive**: Prioritizes lowest cost per token.
- **Performance-Maximized**: Prioritizes lowest latency and highest uptime, willing to pay higher rates.
- **Trust-Centric**: Heavily weights social graph proximity (friends and friends-of-friends) over performance or price.
- **Quality-First**: Selection based primarily on high scores from RIP-09 evaluation benchmarks.

Clients are encouraged to allow users to customize these weights or select from preset "profiles".

### 3.3. Probabilistic Selection

To prevent thundering herds (all clients picking the single "best" node), selection should be probabilistic among top candidates:

- **Top-N Random**: Select randomly from the top 5 highest-scoring nodes.
- **Weighted Random**: Selection probability is proportional to the calculated score.

## 4. Recursive Routing & Fallback Hierarchies

### 4.1. Client-Side Fallback

Smart Clients (RIP-03) construct a prioritized list of providers:

1. **Primary**: The selected high-score provider.
2. **Secondary/Tertiary**: Backup providers with slightly lower scores but different failure domains (e.g., different regions).
3. **Failover Logic**: If Primary fails (network error, 5xx, insufficient balance refund), the client automatically retries with Secondary.

### 4.2. Aggregator Routing  (NotImplementedYet)

Routing Nodes (aggregators) use these same algorithms to forward requests to upstream providers transparently.

- **Recursive Depth**: An aggregator effectively acts as a client to the upstream network.
- **Value Add**: Aggregators can provide stability by maintaining diverse, high-availability fallback sets that individual clients might not discover.
