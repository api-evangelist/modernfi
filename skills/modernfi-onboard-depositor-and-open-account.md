---
name: Onboard a depositor and open a deposit account
description: Create a depositor, open a deposit account for them in the ModernFi network, then confirm the network allocation.
api: openapi/modernfi-openapi-original.json
operations:
  - create   # POST /digital-banking/v1/depositors
  - create   # POST /digital-banking/v1/accounts
  - get      # GET  /digital-banking/v1/accounts/{account_id}
  - getAllocation  # GET /digital-banking/v1/accounts/{account_id}/allocation
---

# Onboard a depositor and open a deposit account

Use the ModernFi Digital Banking API (base `https://api.modernfi.com`, prefix
`/digital-banking/v1`). This flow needs **read+write** access.

## Auth
1. POST `https://auth.modernfi.com/oauth/token` with JSON body
   `{client_id, client_secret, audience: "https://api.modernfi.com", grant_type: "client_credentials"}`.
2. Send `Authorization: Bearer <access_token>` on every call. Tokens last 86400s.

## Steps
1. **Create the depositor** — `POST /digital-banking/v1/depositors` (operationId
   `create`). Supply identity fields (first/last or entity name, `tin`,
   `depositor_type`, `ownership_category`). Send an `Idempotency-Key` header (UUID
   v4) — it is required on this create. Capture the returned depositor `id`.
2. **Open the account** — `POST /digital-banking/v1/accounts` (operationId
   `create`). Reference the depositor id in `depositors`, set `account_type` and
   any rate/pricing config. Send an `Idempotency-Key` header. Capture the account
   `id`.
3. **Confirm the account** — `GET /digital-banking/v1/accounts/{account_id}`
   (operationId `get`) to verify status and balances.
4. **Check network allocation** — `GET /digital-banking/v1/accounts/{account_id}/allocation`
   (operationId `getAllocation`) to see how the balance is allocated across
   network institutions (populated by the daily allocation job).

## Rules
- **Idempotency:** always send a stable `Idempotency-Key` (UUID v4) on the two
  creates; retries with the same key return the original response (24h window).
  Keys are only stored on a 2XX, so failed calls are safe to retry.
- **Errors:** non-2XX responses return `{ "detail": "<message>" }`; 422 returns a
  structured `{ "detail": [ { loc, msg, type } ] }`. A 403 means your token is
  read-only.
