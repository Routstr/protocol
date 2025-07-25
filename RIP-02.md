# RIP-02: Node Listing

Nodes announce their presence and capabilities via Nostr events for client discovery, following NIP-91.

## Provider Information Event

**Kind**: 38421

```json
{
  "kind": 38421,
  "pubkey": "<routstr-provider-pubkey>",
  "created_at": <unix-timestamp>,
  "content": "<optional-kind:0-style-metadata>",
  "tags": [
    ["d", "<unique-provider-identifier>"],
    ["u", "https://..."],      // HTTP endpoint
    ["u", "<tor-onion-address>"],  // Tor hidden service endpoint
    ["models", "gpt-4o-mini", "gpt-4o", "claude-3-5-sonnet", "..."],
    ["mint", "https://mint.example.com"],
    ["version", "0.0.1"]
  ]
}
```

### Model Capabilities (Optional)

Providers MAY include additional tags to describe model-specific capabilities:

```json
{
  "tags": [
    ["model-cap", "gpt-4o", "max_tokens:128000,vision:true,tools:true"],
    ["model-cap", "claude-3-5-sonnet", "max_tokens:200000,vision:true,tools:true"]
  ]
}
```

## Recommendation Event

**Kind**: 38000 (as defined in NIP-87)

```json
{
  "kind": 38000,
  "pubkey": "<recommender-user-pubkey>",
  "tags": [
    ["k", "38421"],
    ["d", "<d-identifier>"],
    ["u", "<recommended-routstr-http-url>"],
    ["a", "38421:routstr-provider-pubkey:<d-identifier>", "wss://relay1"]
  ],
  "content": "Fast and reliable AI proxy with great model selection"
}
```

## Discovery

Clients can discover providers by:

1. Querying for `kind:38000` events with `#k` filter for `38421` (recommendations)
2. Directly querying for `kind:38421` events (provider announcements)

Provider events are replaceable by the `d` tag, with newer events overriding older listings.
