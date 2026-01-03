# Routstr Improvement Protocols (RIPs)

![RIP](https://www.stuttgarter-zeitung.de/media.media.2418d951-7f77-41ba-a726-08d3a8043fd9.16x9_1024.jpg)

Routstr Improvement Protocols (RIPs) are modular specification documents defining the events, behaviors, and interfaces for the decentralized AI inference network. They complement the core protocol by specifying discrete improvements, standards, and guidelines that implementations can adopt to enhance interoperability, security, and user experience.

## List of RIPs

| ID | Title | Description |
|---|---|---|
| RIP-01 | [Proxy / Payments](RIP-01.md) | Standard for OpenAI-compatible API Proxy with integrated Cashu micropayments. |
| RIP-02 | [Discovery](RIP-02.md) | Nostr event specifications for Provider Announcements (Kind 38421) and Recommendations (Kind 38000). |
| RIP-03 | [Client Specification](RIP-03.md) | Standards for client behavior, including ephemeral auth, wallet management, and auto-topup. |
| RIP-04 | [Auditing](RIP-04.md) | Protocols for basic Uptime, Reliability, and Feature Verification checks. |
| RIP-05 | [Pricing Algorithm](RIP-05.md) | Standardized unit of account (Millisats), cost calculation logic, and client-side verification. |
| RIP-06 | [Routing Algorithms](RIP-06.md) | Algorithms for provider selection, scoring (Price/Trust/Perf), and failover hierarchies. |
| RIP-07 | [Admin API](RIP-07.md) | Internal API specification for node management (Settings, Upstreams, Models, Wallet). |
| RIP-08 | [Payment Methods](RIP-08.md) | Gateways for funding sessions via Lightning (Bolt11), NWC, Bolt12, and On-Chain Bitcoin. |
| RIP-09 | [Evaluations](RIP-09.md) | Advanced "Mystery Shopping" benchmarks for Model Authenticity, Performance, and Safety. |
| RIP-10 | [Marketplace](RIP-10.md) | Dynamics of the decentralized order book: Limit Sell, Market Buy, Limit Buy (Batch), and Spot Markets. |
| RIP-11 | [Specialized Nodes](RIP-11.md) | Configuration profiles for General, Personal, Worker, and Routing nodes. |

Refer to each RIP file in this directory for detailed implementation guidance.
