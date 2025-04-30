# RIP-02: Node Listing

Nodes announce their presence and capabilities via a Nostr event for client discovery.

**Kind**: 40500

```json
{
  "kind": 40500,
  "created_at": <unix-timestamp>,
  "tags": [
    ["d", "<node-id>"],       // Unique node identifier
    ["p", "<operator-pubkey>"],
    ["url", "https://..."],    // Inference endpoint
    ["model", "<model-id>"],   // Repeatable
    ["price", "<msats>"],      // mSAT per request
    ["region", "<ISO-region>"],
    ["latency_ms", "<avg-ms>"]
  ],
  "content": "Human-readable description"
}
```

Clients subscribe to Kind 40500, index by `d` tag, and override older listings with newer events.
