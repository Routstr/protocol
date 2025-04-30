# Routstr Improvement Protocols (RIPs)

![RIP](https://www.stuttgarter-zeitung.de/media.media.2418d951-7f77-41ba-a726-08d3a8043fd9.16x9_1024.jpg)

Routstr Improvement Protocols (RIPs) are modular specification documents defining the events, behaviors, and interfaces for the decentralized AI inference network. They complement the core protocol by specifying discrete improvements, standards, and guidelines that implementations can adopt to enhance interoperability, security, and user experience.

## List of RIPs

| ID     | Title                                            | Description                                                                  |
|--------|--------------------------------------------------|------------------------------------------------------------------------------|
| RIP-01 | [OpenAI-API Proxy with Cashu Payments](RIP-01.md)         | Proxy spec for OpenAI-compatible API requests, with per-request Cashu payment |
| RIP-02 | [Node Listing](RIP-02.md)                                   | Nostr event spec for announcing inference node presence and capabilities     |
| RIP-03 | [Frontend Discovery](RIP-03.md)                          | Specification for the discovery UI to browse and filter available nodes     |
| RIP-04 | [Evaluations & Quality Control](RIP-04.md)               | Guidelines for randomized, anonymized evaluations to ensure provider quality |
| RIP-05 | [Smart Clients & Token Management](RIP-05.md)              | Client behaviors for token cycling, proxy/Tor usage, and provider optimization |

Refer to each RIP file in this directory for detailed implementation guidance.
