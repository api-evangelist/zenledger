# ZenLedger

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

ZenLedger is a crypto tax and digital-asset accounting company. Beyond the consumer tax product at
app.zenledger.io, it operates two documented B2B REST APIs on `https://api.zenledger.io`:

| API | Version | Operations | Reference |
|---|---|---|---|
| **Compliance Suite API** — digital-asset trade monitoring, tax compliance and sanctions screening for financial institutions | v3 | 27 | https://docs.zenledger.io/compliance/v3/ |
| **Aggregator Suite API** — partner surface that aggregates accounts into a portfolio and returns its tax calculation | v1 | 5 | https://docs.zenledger.io/aggregators/rest-api/v1/ |

Both authenticate with an OAuth 2.0 `client_credentials` JWT from `POST /oauth/token` (30-minute lifetime).
Credentials are issued by ZenLedger; there is no self-serve API signup.

## Where the contract came from

**ZenLedger publishes no OpenAPI.** Its machine-readable contract is a versioned **Postman collection** per API,
served from its own documentation hub. Both collections are saved verbatim in `postman/`, and the OpenAPI documents
in `openapi/` are mechanically derived from them by API Evangelist — every path, method, header, documented
parameter, example request and example response is carried over; nothing was invented. Each derived document says
so in `info.description` and records the original request template per operation in `x-postman-request`.

- `postman/zenledger-compliance-v3.postman_collection.json` — from https://docs.zenledger.io/compliance/v3/compliance_api.postman_collection.json
- `postman/zenledger-aggregators-v1.postman_collection.json` — from https://docs.zenledger.io/aggregators/rest-api/v1/aggregators_api.postman_collection.json

Confirmed live and first-party on 2026-09-05: an anonymous `POST https://api.zenledger.io/oauth/token` returns a
conformant RFC 6749 `invalid_client` error (HTTP 401), and `GET /compliance/api/v3/chains` returns 401.

## What this profile found

- **A rich, current contract.** 32 operations, 45 documented error codes, a 46-subtype transaction taxonomy with
  stated tax treatment per subtype, and three webhooks with a documented retry schedule.
- **No agent surface.** No MCP server, no A2A agent card, no GraphQL, and no `/.well-known/` document on any host.
- **No client SDK in any language.** npm, PyPI, RubyGems, crates.io, Maven and NuGet all return nothing
  first-party. The `github.com/zenledger-io` organization holds forks of exchange clients ZenLedger consumes, not
  clients for its own API.
- **No rate limits, no idempotency, no sandbox.** Neither reference documents a rate limit, a retry-safety
  mechanism, or a test mode.
- **Unsigned webhooks.** No HMAC header, shared secret or IP allowlist is documented on inbound notifications —
  including the one that carries a sanctions-screening verdict.
- **A bounty program with no `security.txt`.** ZenLedger runs a real bug bounty at https://zenledger.io/security/
  but serves no RFC 9116 document on any host, so no scanner or agent can find it.

## Artifacts

`openapi/` `postman/` `overlays/` `authentication/` `conventions/` `errors/` `lifecycle/` `conformance/`
`data-model/` `asyncapi/` `security/` `packages/` `plans/` `rate-limits/` `mcp/` `skills/` `llms/` `well-known/`

- https://zenledger.io/
- https://docs.zenledger.io/
