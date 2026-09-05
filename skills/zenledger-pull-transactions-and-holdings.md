---
name: zenledger-pull-transactions-and-holdings
description: Page through normalized transactions and per-source holdings for a user or a whole company in the ZenLedger Compliance Suite, and classify each transaction correctly.
api: ZenLedger Compliance Suite API
openapi: openapi/zenledger-compliance-api-openapi.yml
operations:
  - jwtRequest
  - getTransactionsForAllUserOfACompany
  - getTransactionsForUser
  - getHoldingsForAllUsersOfACompany
  - getHoldingsForUser
  - getResyncSource
  - getCurrencies
generated: '2026-09-05'
method: generated
source: openapi/zenledger-compliance-api-openapi.yml, conventions/zenledger-conventions.yml, data-model/zenledger-data-model.yml
---

# Pull transactions and holdings

Use this to extract a company's or a user's normalized crypto activity for reporting.

## Steps

### 1. Get a token — `jwtRequest`

30-minute lifetime. A long extraction **will** outlive it: refresh proactively rather than waiting for
`ZENCS-AUTHGET-AA4` mid-page.

### 2. Pull transactions

Company-wide: `GET /compliance/api/v3/companies/{company_reference}/transactions`
One user: `GET /compliance/api/v3/companies/{company_reference}/users/{user_id}/transactions`

Documented query parameters:

| Parameter | Notes |
|---|---|
| `currency_code` | Unsupported code returns `ZENCS-PARAMGET-AA1`. Validate against `getCurrencies`. |
| `date_from` / `date_to` | Cut-off by **ZenLedger import date**. |
| `transaction_date_from` / `transaction_date_to` | Cut-off by **when the transaction happened**. |
| `source_id` | Restrict to one wallet/exchange. |
| `sorting_method` | `transaction_timestamps` or `import_timestamp`. Defaults to transaction date. |
| `page` | Page number. |

Dates must be UTC ISO-8601 — anything else returns `ZENCS-PARAMGET-AA2`.

The two date pairs are not interchangeable. For a tax period you want `transaction_date_*`; for "what changed since
my last sync" you want `date_*`. Picking the wrong pair silently returns the wrong set.

### 3. Page

**100 elements per page**, fixed. There is no `per_page` or `limit` — do not try to raise it. Increment `page` and
read the `pagination` object in the envelope until it is exhausted.

**There are no published rate limits**, and no `Retry-After`, `X-RateLimit-*` or documented 429 to back off
against. Pace yourself conservatively anyway: absence of a documented limit is not a promise there is no limit,
and you have no runtime signal to tell you when you have found it.

### 4. Classify each transaction

Every transaction falls into one of three classes with its own tax treatment:

- **Deposits** (18 subtypes) — a crypto asset enters without being paired as a trade. Includes `airdrop`, `buy`,
  `mined`, `staking_reward`, `interest_received`, `gift_received`, `self_transfer`, `fiat_deposit`.
- **Withdrawals** (12 subtypes) — a crypto asset leaves without one received in return. Includes `sell`, `Send`,
  `gift_sent`, `donation_501c3`, `lost`, `stolen`, `fee`, `fiat_withdrawal`.
- **Trades** (16 subtypes) — crypto for crypto. Includes `trade`, `swap`, `nft_trade`, the `margin_trading_*`
  family and the `liquidity_pool*` family.

**Do not infer the class from the subtype.** `self_transfer`, `staking_lockup`, `staking_return`, `nft_mint` and
`fiat_withdrawal` each appear under more than one class, and the tax treatment differs. Read the class field.

Do not compute or assert a tax outcome yourself. The subtype descriptions state treatment (income vs cost basis vs
non-taxable) — surface them; leave the determination to the tax professional.

### 5. Pull holdings

Company-wide: `GET /compliance/api/v3/companies/{company_reference}/holdings`
One user: `GET /compliance/api/v3/companies/{company_reference}/users/{user_id}/holdings`
One source: append `/{source_id}`.

**Holdings page at 20 per page, not 100.** Different endpoint, different fixed size — hardcode both.

Each source carries a `status` field describing its import state. Check it before trusting the balances: a source
mid-import or failed is not a complete picture. `ZENCS-CMPGET-AA4` on a company-wide call means the company has no
active users at all.

### 6. Refresh stale data — `getResyncSource`

`GET /compliance/api/v3/companies/{company_reference}/holdings/{source_id}/resync`

This is a **GET with side effects** — it re-pulls the source. It is not safe to retry in a loop and it is not
cacheable. Call it deliberately, once, and wait for the `IMPORT_STATUS_UPDATE` webhook or poll the source `status`.

`ZENCS-HLDRSC-AA1` means the resync could not be started; `ZENCS-HLDRSC-AA2` means the source id is wrong.
