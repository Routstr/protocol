# RIP-03: Web Dashboard

This document defines a fully client-side, 100% local web or app interface for provider discovery, chat, proxy interaction and wallet management.
There is no backend: all logic and data flow run in the user's browser or device. The frontend includes a chat application, discovery interface, landing page, and documentation. Authentication and filtering leverage Nostr for login and social graph-based filtering. Users can publish their own node information or interact with available nodes directly from the interface.

## Features

- 100% local frontend: no backend, all data and logic run on the client.
- Includes:
  - Chat application
  - Discovery interface
  - Landing page
  - Documentation
  - NIP-60 Wallet
  - Auth Key management
- Nostr-based login and filtering (social graph proximity)
- List active nodes from Nostr Kind 38421.
- Filter by:
  - Supported model(s)
  - Region
  - Price range
  - Social network proximity (follow graph)
  - Average rating (from evals)
- Publish personal node information or interact with nodes
- Future: Toggle eval visibility once RIP-04 is live.

## UI Components

- **Search Bar**: Free text for node-id or description.
- **Filters Panel**: Sliders and checkboxes.
- **Node Card**:
  - Description
  - Models
  - Price
  - Rating summary
  - Connect button (opens external client)

## Data Flow

1. Subscribe to Nostr Kind 40500 & 31555 events.
2. Maintain in-memory index, update on new events.
3. Join with evals to compute live scores.
4. Render node list with applied filters.

## Metrics

- Query performance: ≤200 ms for 1 k nodes.
- Real-time updates: UI refresh on new events.

---

## TODO

[ ] - unify chat, docs, api key management and landingpage
[ ] - build marketplace interface
[ ] - everything running local, no cloud function
