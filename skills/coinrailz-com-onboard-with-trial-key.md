---
name: Onboard an agent with the free trial key
description: Get a free $5 Coin Railz API key with no wallet or card, then call any metered service with it.
api: coinrailz-com:agent-payment-api
operations:
- getTrialKey
- getAuthCapabilities
- ping
- gasPriceOracle
generated: '2026-09-19'
method: generated
source: Grounded in openapi/ operationIds, conventions/coinrailz-com-conventions.yml, errors/, rate-limits/ and
  the provider's agent-instructions.json; no operation named here is invented.
---

# Onboard an agent with the free trial key

Get a free $5 Coin Railz API key with no wallet or card, then call any metered service with it.

1. Read `GET https://coinrailz.com/api/auth/capabilities` (`getAuthCapabilities`) to confirm the current auth modes; no key is needed.
2. Call `GET https://coinrailz.com/api/m2m/credits/trial` (`getTrialKey`). The response carries `apiKey` (prefix `cr_live_`), `credits: 5` and `expiresIn: 7 days`. **Store the key immediately** - it is returned once and cannot be retrieved again. Limit: 1 key per IP per 7 days; a second call inside the window will not issue another key.
3. Call a metered service with the key as `X-API-KEY` (or `Authorization: Bearer <key>`), e.g. `POST https://coinrailz.com/x402/gas-price-oracle` (`gasPriceOracle`, $0.10) with `Content-Type: application/json` and a `{}` body. `gas-price-oracle` and `token-metadata` are first-call-free.
4. Read the billing headers on every paid response: `X-Credits-Used`, `X-Credits-Remaining`, `X-Recharge-Url`. When credits run out, switch to the buy-credits skill.
5. Errors: a `402` with `error: "X-PAYMENT header is required"` means the key was not sent or is exhausted; a `429` with code `RATE_LIMIT_EXCEEDED` carries `retryAfter` seconds in the body - honour it, and watch the `ratelimit-remaining` / `ratelimit-reset` headers (policy observed: 200 requests per 60 s).
