# RIP-04: Evaluations & Quality Control

- [ ] Implement independent evaluation (eval) services for LLM providers
- [ ] Scan Nostr for all advertised providers
- [ ] Run anonymized, randomized, proxied evaluation jobs
- [ ] Publish benchmark reports for each provider (Kind 31555)
- [ ] Support free and paid report tiers
- [ ] Measure and report quality, latency, and cost metrics
- [ ] Publish standard ratings to Nostr
- [ ] Limit eval jobs to ≤5% of overall requests
- [ ] Make public evals modular component of the protocol

Specifies how independent evaluation (eval) services scan Nostr for all advertised LLM providers, run anonymized and randomized evals, and publish benchmark reports for each provider.

## Goals

- Prevent provider bias by making evals indistinguishable from real inference calls.
- Collect unbiased metrics on quality, latency, and cost.
- Establish eval services as the primary source of trust and ranking in the marketplace.

## Eval Service Model

- Evals are separate, continuously running services that monitor Nostr for all LLM providers advertising their capabilities.
- For each discovered provider, the eval service runs anonymized, randomized, and proxied evaluation jobs to ensure providers cannot distinguish eval requests from normal user traffic.
- Evals are designed to prevent providers from treating evals differently than real users, using strategies such as randomization, proxying, and ephemeral keys.
- After running evals, the service publishes a report for each provider, containing benchmark scores and detailed metrics.

## Report Tiers & Monetization

- There are multiple tiers of reports:
  - **Free Report:** All providers receive a basic report at no cost, summarizing essential metrics.
  - **Paid Reports:** Providers can pay the eval service (in sats) to receive more advanced, detailed reports. These paid reports offer deeper insights, improve user trust, and help the provider rank higher in the marketplace.
- Eval services are a monetizable component of the ecosystem, earning fees for advanced reporting and acting as a trusted third party.

## Eval Flow

1. Eval service scans Nostr for all advertised providers.
2. For each provider:
   - Generate randomized inference parameters.
   - Send anonymized, proxied inference requests.
   - Measure latency, output quality against ground truth.
3. Assemble eval result and prepare Nostr event:
   - Use new random ephemeral Nostr key per eval.
   - Publish Kind 31555 with standard rating tags:
     - `d`: provider node-id
     - `rating` categories: `quality`, `latency`, `value`
   - No direct link to eval service's main pubkey.
4. Publish report (free or paid tier) with benchmark scores.

## Tags

- `d`: provider node-id
- `rating`:
  - `quality`: 0–1 match to ground truth
  - `latency`: normalized inverse latency score
  - `value`: cost vs performance
- `content`: optional detailed notes

## Anonymity & Randomization

- Ephemeral keys rotated per session.
- Randomized send times & intervals.
- All requests are proxied and randomized to prevent provider detection.

## Aggregation

- Eval services can batch multiple evaluations in one event by repeating `rating` tags.

## Frequency

- Limit eval jobs to ≤5% of overall requests to avoid load spikes.

## Trust & Benchmarking

- Eval services are the source of trust in the marketplace, providing objective, third-party benchmarks.
- Each report results in a set of benchmark scores, enabling transparent provider comparison and ranking.
