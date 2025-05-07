# TODO

- [x] 100% local frontend (no backend)
- [x] UI components: search bar, filters panel, model card (description, models, price)
- [x] Include chat application, discovery interface, landing page, and documentation
- [x] Support Nostr-based login
- [x] Cashu wallet integration
- [ ] Filter nodes by supported models, region, price range, social proximity, and average rating
- [ ] Allow users to publish personal node information and interact with nodes
- [ ] List active nodes from Nostr Kind 40500
- [ ] Subscribe to Nostr Kind 40500 & 31555 events
- [ ] Maintain in-memory index and update on new events
- [ ] Render node list with applied filters
- [ ] Real-time UI updates on new events

# RIP-03: Frontend Discovery

This document defines a fully client-side, 100% local web or app interface for node discovery, chat, and interaction. There is no backend: all logic and data flow run in the user's browser or device. The frontend includes a chat application, discovery interface, landing page, and documentation. Authentication and filtering leverage Nostr for login and social graph-based filtering. Users can publish their own node information or interact with available nodes directly from the interface.

## Features

- 100% local frontend: no backend, all data and logic run on the client.
- Includes:
  - Chat application
  - Discovery interface
  - Landing page
  - Documentation
- Nostr-based login and filtering (social graph proximity)
- List active nodes from Nostr Kind 40500.
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
