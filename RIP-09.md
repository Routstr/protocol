# RIP-09: Evaluations & Quality Control

This document defines the protocol for advanced, third-party verification of provider quality and honesty. While **RIP-04 (Auditing)** covers basic uptime and feature checks, RIP-09 specifies the methodology for running costly, anonymized, and qualitative benchmarks to detect sophisticated fraud (e.g., model substitution) and measure performance under load.

## 1. Goal

To establish a "Web of Trust" where independent evaluators verify that providers are delivering the intelligence they advertise.

- **Fraud Detection**: Detecting if a provider advertises `gpt-4` (expensive) but routes to `gpt-3.5-turbo` or `llama-3-70b` (cheaper) to pocket the difference.
- **Performance Benchmarking**: Measuring real-world tokens-per-second (TPS) and time-to-first-token (TTFT).

## 2. Methodology: Anonymized "Mystery Shopping"

To prevent providers from gaming the metrics (e.g., prioritizing traffic from known audit bots), evaluations MUST be indistinguishable from normal user traffic.

1. **Ephemeral Identity**: The evaluator creates a fresh, one-time-use wallet and keypair for each benchmark run.
2. **Paid Traffic**: The evaluator pays full market price for the request.
3. **Real-World Prompts**: The benchmark uses diverse, realistic prompt sets (e.g., coding problems, reasoning tasks) rather than static "ping" strings that are easily fingerprinted.

## 3. Metrics

### 3.1. Model Authenticity (Fingerprinting)

Verifying the underlying model identity.

- **Deterministic Questions**: Asking questions where models have known, distinct biases or refusal patterns.
- **Logprobs Analysis**: If supported, comparing token probabilities against a reference implementation (ground truth).
- **Hard Reasoning**: Submitting complex reasoning tasks that smaller models typically fail.

### 3.2. Performance

- **Time to First Token (TTFT)**: Latency from request to first byte.
- **Tokens Per Second (TPS)**: Generation speed.
- **Variance**: Consistency of performance over multiple samples.

### 3.3. Feature Verification

- **Function Calling**: Verifying the model correctly triggers and formats tool calls.
- **JSON Mode**: Testing strict schema adherence.
- **Context Window**: Sending long-context inputs to verify the provider supports the full advertised window size (and doesn't silently truncate).

### 3.4. Security & Safety Verification

Evaluators probe for vulnerabilities that could expose downstream users or agents to risk.

- **Prompt Injection**: Testing if the model can be coerced into ignoring system instructions or executing arbitrary commands (especially relevant for models with tool-use/agent capabilities).
- **Tool Sandbox Escape**: Attempting to manipulate tool arguments to access unauthorized resources.
- **Data Leakage**: Checking if the model reveals sensitive training data or system prompts from other users in the same context (for cached contexts).

## 4. Reporting

Evaluators publish their findings as **Kind 31555 (Evaluation Report)** events on Nostr.

- **Content**:
  - `provider_pubkey`: The audited provider.
  - `model`: The specific model tested.
  - `authenticity_score`: 0.0 - 1.0 (confidence that the model is genuine).
  - `performance`: `{ "tps": 45.2, "ttft_ms": 120 }`
  - `timestamp`: Time of audit.
  - `proof`: Optional cryptographic proof or trace ID of the interaction.

## 5. Monetization & Incentives

Running these benchmarks is costly (computational + API fees).

- **Subscription Model**: Clients or Aggregators subscribe to the Evaluator's paid feed (encrypted Nostr events) to get real-time "whitelist/blacklist" intelligence.
- **Bounties**: The network (or a DAO) funds a bounty pool for consistent high-quality audit reports.
