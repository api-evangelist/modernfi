---
name: Price accounts with pricing groups and custom benchmarks
description: Define a custom benchmark and a pricing group, set its rates, and assign accounts to it.
api: openapi/modernfi-openapi-original.json
operations:
  - create        # POST /digital-banking/v1/custom-benchmarks
  - create        # POST /digital-banking/v1/pricing-groups
  - updateRates   # PUT  /digital-banking/v1/pricing-groups/{pricing_group_id}/rates
  - assignAccounts  # POST /digital-banking/v1/pricing-groups/{pricing_group_id}/accounts
---

# Price accounts with pricing groups and custom benchmarks

Base `https://api.modernfi.com`, prefix `/digital-banking/v1`. Needs
**read+write** access.

## Auth
OAuth 2.0 client_credentials — token from `https://auth.modernfi.com/oauth/token`,
sent as `Authorization: Bearer <token>`.

## Steps
1. **(Optional) Create a custom benchmark** — `POST /digital-banking/v1/custom-benchmarks`
   (operationId `create`) to define a reference rate; capture `custom_benchmark_id`.
   Update its rate later with `PUT /digital-banking/v1/custom-benchmarks/{id}`
   (operationId `update_rate`).
2. **Create a pricing group** — `POST /digital-banking/v1/pricing-groups`
   (operationId `create`). Capture `pricing_group_id`.
3. **Set the group's rates** — `PUT /digital-banking/v1/pricing-groups/{pricing_group_id}/rates`
   (operationId `updateRates`). Rates can be fixed, a floating spread, or tied to a
   custom benchmark / index.
4. **Assign accounts** — `POST /digital-banking/v1/pricing-groups/{pricing_group_id}/accounts`
   (operationId `assignAccounts`); remove with `DELETE .../accounts` (operationId
   `unassignAccounts`). Assigned accounts inherit the group's `rate_configuration`.

## Rules
- **Idempotency:** the pricing-group/benchmark creates in the OpenAPI do not
  declare an `Idempotency-Key` parameter (only account/depositor/transaction
  creates do); still make assignment operations safe by checking the account's
  `pricing_group_id` before reassigning.
- **Errors:** `{ "detail": ... }`; a 403 indicates a read-only token.
