# RIP-04: Evaluations & Quality Control

Specifies how clients anonymize and randomize evaluation submissions to mimic normal inference requests.

## Goals

- Prevent provider bias by making evals indistinguishable from real inference calls.
- Collect unbiased metrics on quality, latency, and cost.

## Eval Flow

1. Client selects a subset of providers based on discovery.
2. For each eval job:
   - Generate randomized inference parameters.
   - Send inference request randomized with proxy.
   - Measure latency, output quality against ground truth.
3. Assemble eval result and prepare Nostr event:
   - Use new random ephemeral Nostr key per eval.
   - Publish Kind 31555 with standard rating tags:
     - `d`: provider node-id
     - `rating` categories: `quality`, `latency`, `value`
   - No direct link to client's main pubkey.

## Tags

- `d`: provider node-id
- `rating`:
  - `quality`: 0–1 match to ground truth
  - `latency`: normalized inverse latency score
  - `value`: cost vs performance
- `content`: optional detailed notes

## Anonymity

- Ephemeral keys rotated per session.
- Randomized send times & intervals.

## Aggregation

- Clients can batch multiple evaluations in one event by repeating `rating` tags.

## Frequency

- Limit eval jobs to ≤5% of overall requests to avoid load spikes.
