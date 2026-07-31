---
name: Record and reconcile account transactions
description: Post a transaction record against an account, then list and reconcile transactions and pull statements.
api: openapi/modernfi-openapi-original.json
operations:
  - create  # POST /digital-banking/v1/accounts/{account_id}/transactions
  - list    # GET  /digital-banking/v1/accounts/{account_id}/transactions
  - get     # GET  /digital-banking/v1/accounts/{account_id}/transactions/{transaction_id}
  - get_statements_digital_banking_v1_files_statements_get  # GET /digital-banking/v1/files/statements
---

# Record and reconcile account transactions

Base `https://api.modernfi.com`, prefix `/digital-banking/v1`. Needs
**read+write** access for the create; read-only is enough for the rest.

## Auth
OAuth 2.0 client_credentials — mint a token at
`https://auth.modernfi.com/oauth/token`, then send `Authorization: Bearer <token>`.

## Steps
1. **Record a transaction** — `POST /digital-banking/v1/accounts/{account_id}/transactions`
   (operationId `create`). Send an `Idempotency-Key` header (UUID v4) — required —
   so a network retry never double-posts (e.g. a duplicate $10,000 record).
2. **List transactions** — `GET /digital-banking/v1/accounts/{account_id}/transactions`
   (operationId `list`). Page with `page` + `page_size`; filter with `start_date`/
   `end_date` (or `start_datetime`/`end_datetime` + `timezone`), `transaction_type`,
   and `transaction_status`.
3. **Inspect one** — `GET /digital-banking/v1/accounts/{account_id}/transactions/{transaction_id}`
   (operationId `get`).
4. **Pull statements** — `GET /digital-banking/v1/files/statements` (operationId
   `get_statements_digital_banking_v1_files_statements_get`) to reconcile against
   monthly statements.

## Rules
- **Idempotency (9-pt convention):** stable `Idempotency-Key` UUID v4 on the
  create; 24h retention; stored only on 2XX.
- **Pagination:** transaction listing is page-number based (`page`, `page_size`).
- **Errors:** `{ "detail": ... }` envelope; 422 is a validation array.
