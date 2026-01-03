# RIP-07: Admin API

This document specifies the internal Administrative API for managing a Routstr provider node. The Admin API is intended for the node operator and is secured via a persistent secret token or password-based session.

## 1. Authentication

Access to all Admin API endpoints requires authentication.

- **Method**: Bearer Token or Session Cookie.
- **Authorization**: `Authorization: Bearer <ADMIN_TOKEN>`
- **Security**: This API should typically be exposed only on `localhost` or over a secure VPN/tunnel.

## 2. Endpoints Overview

The API is divided into functional modules:

### 2.1. General Settings (`/admin/api/settings`)

Manages the core configuration of the provider node.

- **GET /api/settings**: Retrieve current configuration (redacting secrets).
- **PATCH /api/settings**: Update configuration (e.g., `cashu_mints`, `description`, `name`).
- **POST /api/settings/password**: Rotate the admin password.

### 2.2. Upstream Providers (`/admin/api/upstreams`)

Manages connections to backend LLM providers (e.g., OpenAI, OpenRouter, vLLM).

- **GET /api/upstreams**: List configured upstream providers.
- **POST /api/upstreams**: Add a new upstream provider.
- **PATCH /api/upstreams/{id}**: Update credentials or base URL.
- **DELETE /api/upstreams/{id}**: Remove a provider.

### 2.3. Models Management (`/admin/api/models`)

Controls the catalog of models offered to the public network.

- **GET /api/models**: List all models (active and inactive).
- **POST /api/models**: Manually add or import a model.
- **PATCH /api/models/{id}**:
  - Enable/Disable a model.
  - Set custom pricing (overriding default calculation).
  - Set aliases (e.g., map `gpt-4o` to `my-custom-gpt4`).
- **DELETE /api/models/{id}**: Remove a model from the catalog.

### 2.4. Wallet & Balances (`/admin/api/balances`, `/admin/api/temporary-balances`)

Monitors the financial state of the node.

- **GET /api/balances**: View total holdings in the internal Cashu wallet.
- **GET /api/temporary-balances**: View active ephemeral sessions and their current "held" funds.
- **POST /admin/withdraw**: Withdraw funds from the internal wallet to an external Cashu token.

### 2.5. Logs & Audits (`/admin/logs`)

Investigative tools for debugging and monitoring.

- **GET /logs/{request_id}**: Retrieve detailed trace logs for a specific request ID (useful for debugging client issues).
- **GET /logs/recent**: Fetch a stream of recent system events.
