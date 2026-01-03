# RIP-04: Auditing

This document defines the process for auditing Routstr providers to ensure reliability, compliance, and honesty. Auditing is a lightweight, continuous verification process performed by independent entities or clients themselves to maintain network health.

## 1. Uptime & Reliability Monitoring

Auditors periodically check the availability and responsiveness of providers.

- **Ping Checks**: Auditors send lightweight requests (e.g., `GET /v1/info`) to verify the provider is online.
- **Latency Tracking**: Response times are measured and logged. High latency or timeouts negatively impact the provider's score.
- **Uptime Score**: A rolling average of successful checks over time (e.g., 99.9% uptime over 24h) is calculated and may be published via Nostr.

## 2. Feature Verification

Auditors verify that providers actually support the features and models they advertise in their Kind 38421 announcements.

- **Model Verification**: Auditors query the `/v1/models` endpoint to confirm the list of supported models matches the advertisement.
- **Capabilities Check**: Simple inference requests (with minimal tokens) are sent to specific models to ensure they are functional and not just placeholders.
- **API Compliance**: Checks are performed to ensure the provider adheres to the standard Routstr/OpenAI API schema (correct error codes, header formats, etc.).

## 3. Price Integrity Checks

Auditors ensure that the prices charged by the provider match their advertised rates.

- **Advertised vs. Actual**: The auditor compares the pricing information returned in `/v1/models` (or the advertisement event) with the actual cost deducted during a test transaction.
- **Refund Verification**: The auditor verifies that the `/v1/balance/refund` endpoint correctly returns the expected remaining balance after a transaction.
- **Overcharge Detection**: If a provider consistently charges more than advertised, they are flagged.

## 4. Reporting

Audit results are published to the Nostr network to inform clients and other network participants.

- **Event Kind**: (To be defined, e.g., Kind 3xxxx)
- **Content**:
  - `provider_pubkey`: The pubkey of the audited provider.
  - `timestamp`: Time of the audit.
  - `status`: `pass`, `fail`, `warning`.
  - `details`: Specifics of failures (e.g., "Model 'gpt-4' not found", "High latency > 2s").
- **Client Usage**: Clients ingest these audit events to filter out unreliable or dishonest providers from their local registry.

## 5. Advanced Evaluations

For comprehensive quality benchmarks, fraud detection (e.g., model substitution), and performance metrics (TPS/TTFT), refer to **[RIP-09: Evaluations](RIP-09.md)**. RIP-04 focuses on lightweight, continuous health checks, while RIP-09 defines the protocol for periodic, deep-dive "mystery shopper" investigations.
