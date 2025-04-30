# RIP-03: Frontend Discovery

Defines the web or app interface for browsing and filtering available inference nodes based on social network and evolving evals.

## Features

- List active nodes from Nostr Kind 40500.
- Filter by:
  - Supported model(s)
  - Region
  - Price range
  - Social network proximity (follow graph)
  - Average rating (from evals)
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
