# RIP-05: Pricing Algorithm

This document defines the standard pricing model, cost calculation logic, and verification procedures for the Routstr protocol. It ensures transparency and predictability in the decentralized marketplace.

## 1. Unit of Account

- **Base Unit**: All internal calculations and balances are tracked in **Millisats (msats)** to ensure precision for micro-transactions.
  - 1 SAT = 1,000 msats.
- **Public API**: The `/v1/models` endpoint exposes pricing in **SATS** (floating point) or **USD** (for reference), but the actual accounting happens in msats.

## 2. Pricing Components

Providers define pricing via the `Pricing` object in the `/v1/models` endpoint. The total cost of a request is the sum of applicable components:

| Component | Description | Unit |
| :--- | :--- | :--- |
| **Prompt** | Cost for processing input text/tokens. | sats / 1M tokens |
| **Completion** | Cost for generating output text/tokens. | sats / 1M tokens |
| **Request** | Fixed fee per API call (optional). | sats / request |
| **Image** | Cost per generated image (optional). | sats / image |
| **Web Search** | Cost for active web search/browsing (optional). | sats / query |
| **Reasoning** | Cost for internal reasoning tokens (e.g., DeepSeek R1). | sats / 1M tokens |

*Note: Prices are typically pegged to USD by the provider and converted to SATS in real-time based on the current BTC/USD exchange rate.*

## 3. Cost Calculation Logic

The provider calculates the final cost *after* the request is completed based on actual usage.

```python
Total_Cost_msats = (Input_Tokens * Price_Input_per_token_msats) +
                   (Output_Tokens * Price_Output_per_token_msats) +
                   (Image_Count * Price_Image_msats) +
                   (Web_Search_Count * Price_Search_msats) +
                   Request_Fee_msats
```

- **Rounding**: Calculations are performed in floating point and rounded **UP** (ceil) to the nearest millisat to ensure the provider is covered.

## 4. Max Cost (Minimum Required Balance)

Since the output length is unknown before the request, the Client must prepay the **Max Cost**.

### 4.1. Formula

The provider calculates `max_cost` for each model and exposes it in `/v1/models`. The standard formula is:

```python
Max_Cost = (Context_Window_Size * Price_Input) +
           (Max_Output_Limit * Price_Output) +
           Request_Fee
```

*If `Max_Output_Limit` is not defined by the model architecture, a safe default (e.g., 4096 tokens) or the full context remaining is used.*

### 4.2. Client usage

The client **MUST** ensure the Cashu token used for the request has a value `>= max_cost`.

## 5. Client-Side Verification

To prevent "hidden fees" or overcharging, clients should verify the cost charged by the provider.

### 5.1. Verification Steps

1. **Fetch Rates**: Before the request, the client caches the pricing rates from `/v1/models`.
2. **Track Usage**: The client records the `usage` (prompt_tokens, completion_tokens) returned in the API response.
3. **Calculate Expected Cost**:

    ```
    Expected_Cost = (usage.prompt_tokens * cached_rates.prompt) +
                    (usage.completion_tokens * cached_rates.completion) +
                    ...
    ```

4. **Calculate Actual Cost**:

    ```
    Actual_Cost = Original_Token_Value - Refunded_Token_Value
    ```

5. **Compare**:
    - `abs(Expected_Cost - Actual_Cost) < Tolerance`
    - A small tolerance (e.g., 1-5 sats) is recommended to account for minor exchange rate fluctuations during the request window or rounding differences.

### 5.2. Dispute Resolution

- If `Actual_Cost` consistently exceeds `Expected_Cost` beyond the tolerance, the client should:
  - Log a "Price Mismatch" warning.
  - Downgrade the provider's trust score.
  - Publish a negative audit event (RIP-04) if the discrepancy is egregious.
