---
name: zenledger-aggregate-portfolio-taxes
description: Create an aggregated portfolio from a set of exchange and wallet accounts with the ZenLedger Aggregator Suite and retrieve its tax calculation.
api: ZenLedger Aggregator Suite API
openapi: openapi/zenledger-aggregator-api-openapi.yml
operations:
  - jwtRequest
  - getSources
  - getCurrencies
  - createPortfolio
  - taxes
generated: '2026-09-05'
method: generated
source: openapi/zenledger-aggregator-api-openapi.yml, conventions/zenledger-conventions.yml
---

# Aggregate a portfolio and get its tax calculation

The Aggregator Suite is the partner-facing surface: hand it a set of accounts, get back a portfolio and its tax
calculation. It is a two-step flow with a much smaller surface than the Compliance Suite — five operations total.

## Steps

### 1. Get a token — `jwtRequest`

`POST https://api.zenledger.io/oauth/token` — the **same** endpoint and the same credentials as the Compliance
Suite. 30-minute lifetime, `Authorization: Bearer {access_token}` thereafter.

The Aggregator reference describes the token request as a `GET` in its prose, while its own JavaScript sample and
its Postman collection both use `POST`. **Use POST.**

### 2. Resolve the accounts — `getSources`, `getCurrencies`

`GET /aggregators/api/v1/sources` — supported exchanges and wallets.
`GET /aggregators/api/v1/currencies` — supported currencies.

Resolve every account against `getSources` before building the portfolio payload. An unrecognised source is
rejected at creation, and creation is the expensive step.

### 3. Create the portfolio — `createPortfolio`

`POST /aggregators/api/v1/portfolios`

Returns an **aggregation code** (`aggcode`, observed form `AGGREF7bfa64c5d52e`).

**Store the `aggcode` immediately.** It is the only handle on the portfolio. The Aggregator Suite publishes no
read, list, update or delete operation for portfolios — if you lose the code, you cannot enumerate your way back
to it, and you cannot delete the portfolio you just made. There is also no idempotency mechanism, so a retried
POST creates a second portfolio rather than returning the first.

`ZENAGG-PFLPST-AA1` means creation failed. Re-check the sources before resending, and log the attempt so a
duplicate does not go unnoticed.

### 4. Retrieve the tax calculation — `taxes`

`GET /aggregators/api/v1/taxes?aggcode={aggcode}`

Requires the code from step 3. `ZENAGG-RSRCGET-AA1` means the code did not resolve.

## Cautions

- Present the calculation as ZenLedger's output, attributed. Do not restate it as tax advice and do not adjust it.
- There are no published rate limits and no backoff headers on this API either. Pace conservatively.
- Portfolio creation is not reversible through the API. Confirm the account set with a human before calling it.
