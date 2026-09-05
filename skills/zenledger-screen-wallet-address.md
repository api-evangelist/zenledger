---
name: zenledger-screen-wallet-address
description: Screen a blockchain address against sanctions and risk lists with the ZenLedger Compliance Suite, synchronously or via the pre-import webhook.
api: ZenLedger Compliance Suite API
openapi: openapi/zenledger-compliance-api-openapi.yml
operations:
  - jwtRequest
  - getChains
  - getWalletScreeningReport
generated: '2026-09-05'
method: generated
source: openapi/zenledger-compliance-api-openapi.yml, asyncapi/zenledger-compliance-webhooks.yml
---

# Screen a blockchain address

Use this to check whether an address appears on a sanctions or risk list before onboarding or transacting with it.

## Steps

### 1. Get a token — `jwtRequest`

As in the onboarding skill. `Authorization: Bearer {access_token}`, 30-minute lifetime.

### 2. Confirm the chain is supported — `getChains`

`GET /compliance/api/v3/chains`

Screening only works on supported chains. Passing an unsupported one returns `ZENCS-PARAMGET-AA6`; omitting it
returns `ZENCS-PARAMGET-AA5`. Check first rather than discovering it in the error.

### 3. Screen the address — `getWalletScreeningReport`

`GET /compliance/api/v3/screening?chain={chain}&address={address}`

Both parameters are mandatory — a missing address returns `ZENCS-PARAMGET-AA4`.

### 4. Read the verdict

    {
      "screening_status": "filed",
      "screening_report": [
        {
          "blockchain": "ETH",
          "address": "0x...",
          "report_data": [{"description": "The address listed on the US Treasury Department's Office of Foreign Assets Control sanction list."}],
          "owner_info": [{"owner_info": [{"name": "...", "url": "...", "legal_name": "..."}]}]
        }
      ]
    }

- `screening_status: "clean"` — no risk indicator triggered. `owner_info` is `null`.
- `screening_status: "filed"` — the address matched. `report_data[].description` says which list.

`owner_info` is doubly nested — an array of objects each carrying their own `owner_info` array. That is how
ZenLedger publishes it; parse accordingly.

## What this does and does not tell you

Report the `description` verbatim. Do not paraphrase a sanctions finding, do not summarise "filed" as "risky", and
do not act on the verdict on your own — an OFAC match is a legal determination with reporting duties attached. Hand
the report to a human on the compliance team.

A `clean` result is a point-in-time answer from ZenLedger's lists. It is not a guarantee and it does not expire in
any documented way, so re-screen rather than caching indefinitely.

## The webhook alternative

If Wallet Screening is enabled for your Enterprise and `wallet_screening_notification_url` is set on the company,
ZenLedger pushes an `ADDRESS_SCREENING_REPORT` with the same shape **before** an import begins — which is the point
at which you can still stop it.

Note that **no signature is documented on this webhook**. For a payload that carries a sanctions verdict, verify
out of band with `getWalletScreeningReport` before acting on anything a webhook told you.
