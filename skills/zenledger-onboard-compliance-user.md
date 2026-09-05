---
name: zenledger-onboard-compliance-user
description: Register a company and an end user in the ZenLedger Compliance Suite, connect a wallet or exchange account, and confirm the import completed.
api: ZenLedger Compliance Suite API
openapi: openapi/zenledger-compliance-api-openapi.yml
operations:
  - jwtRequest
  - createCompany
  - createCompanyUser
  - postWallet
  - postExchange
  - getHoldingsForUser
generated: '2026-09-05'
method: generated
source: openapi/zenledger-compliance-api-openapi.yml, conventions/zenledger-conventions.yml, asyncapi/zenledger-compliance-webhooks.yml
---

# Onboard a Compliance Suite user and connect their first account

Use this when a compliance team needs a new end user tracked in ZenLedger with at least one exchange account or
wallet feeding it.

## Before you start

You need a `client_id` and `client_secret` issued by ZenLedger. There is no self-serve key — if you do not have
them, this flow cannot start and the answer is to contact ZenLedger, not to retry.

You also need the shared **API secret** used to sign and encrypt the import request. It is distinct from the OAuth
client secret and is exchanged out of band.

## Steps

### 1. Get a token — `jwtRequest`

`POST https://api.zenledger.io/oauth/token`, body `application/json`:

    {"client_id": "...", "client_secret": "...", "grant_type": "client_credentials"}

Keep `access_token`. It expires in **1800 seconds**. Re-request at the same endpoint when it does; there is no
refresh grant. Send `Authorization: Bearer {access_token}` on every step below.

If you get `ZENCS-AUTHGET-AA3` the key is invalid; `ZENCS-AUTHGET-AA4` means the token expired — get a new one and
retry once.

### 2. Create the company — `createCompany`

`POST /compliance/api/v3/companies`

Set the webhook endpoints here, at creation. They are company attributes, not a subscription API, and there is no
operation that lists or tests them later:

- `import_notification_url` — receives `IMPORT_STATUS_UPDATE`.
- `wallet_screening_notification_url` — receives `ADDRESS_SCREENING_REPORT`, but only if Wallet Screening is
  enabled for your Enterprise.

`company_reference` is **yours to choose** and must be unique. A collision returns `ZENCS-CMPPST-AA2`
("Duplicated Reference for company"). **There is no idempotency mechanism on this API** — a retried POST does not
replay the original response, it fails as a duplicate. So treat `ZENCS-CMPPST-AA2` as "this already exists, read it
back with `getCompany`", not as a failure.

`ZENCS-CMPPST-AA3` means you are at your account's company cap. That cap is not published; ask ZenLedger.

### 3. Create the user — `createCompanyUser`

`POST /compliance/api/v3/companies/{company_reference}/users`

Email must be unique within the company — a duplicate returns `ZENCS-USRPST-AA2`. Same reasoning as above: on a
duplicate, read the user back rather than retrying the write. `ZENCS-USRPST-AA8` is the user cap.

Keep the returned `user_id` (a UUID).

### 4. Connect the account — `postWallet` or `postExchange`

This is **not a plain JSON POST**. The body must be AES-256-CBC encrypted and the request signed:

- Derive the key: SHA-256 digest of your shared API secret.
- Generate a random 16-byte IV.
- Encrypt the JSON payload; base64 the ciphertext.
- `X-Signature` = HMAC-SHA256 of the base64 ciphertext, keyed with the secret, hex-encoded.
- Send `{"data": <ciphertext>, "iv": <base64 iv>, "signature": <hex>}` with `Content-Type: application/json` and
  the `X-Signature` header.

Payload fields for an exchange: `type`, `exchange_reference`, `access_token`, `refresh_token`, `password`,
`private_key`, `key_name`. Get `exchange_reference` from `getSources` — an unknown one returns
`ZENCS-PARAMGET-AA3`.

Note the path: the wallet variant is on `/compliance/api/v3/...`, the exchange variant is documented on
`/compliance/api/v1/...`. That is what ZenLedger publishes; do not "correct" it to v3.

Error handling that matters here:
- `ZENCS-USRPST-AA5` — wallet already imported. Not a failure; call `getHoldingsForUser` and use the existing source.
- `ZENCS-USRPST-AA7` — the exchange API credentials expired. The user must create a new key pair at their exchange.
  No amount of retrying fixes this.
- `ZENCS-USRPST-AA6` — review the account data before resending.

### 5. Confirm the import — webhook, or `getHoldingsForUser`

If your `import_notification_url` is reachable, you will receive:

    {"notification_type": "IMPORT_STATUS_UPDATE", "import_status": "complete", "transaction_count": 7235, ...}

**Switch on `import_status`, not on `notification_type`.** A stopped import arrives with the same
`notification_type` and `import_status: "limit-reached"` plus a `transaction_limit` field. Treating that as
"complete" means reporting on a partial ledger. If you see it, call `getResumeSource` —
`GET /companies/{ref}/holdings/{source_id}/resume` — which lifts the cap and re-triggers the import;
already-imported transactions are de-duplicated, and a real "complete" follows.

Respond **HTTP 200** to the webhook. Anything else triggers retries at 1m, 3m, 10m, 30m, 1h and a final 24h attempt.

**ZenLedger documents no webhook signature**, so you cannot verify the sender from the request alone. If that is
unacceptable — and for a screening verdict it usually is — skip the webhook and poll `getHoldingsForUser`, reading
the `status` field on each source instead.

## Undoing this

- Remove the connected account: `deleteUserSource` (`DELETE .../holdings/{source_id}`).
- Remove the user: `deleteCompanyUser`. Remove the company: `deleteCompany`.
- **No restore operation and no retention window is documented for any of these.** Treat every delete as permanent
  and confirm with a human before calling one.
