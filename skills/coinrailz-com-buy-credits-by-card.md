---
name: Buy prepaid credits by card (hosted checkout)
description: Fund an agent with prepaid credits through Stripe hosted checkout and poll for the auto-provisioned
  API key.
api: coinrailz-com:agent-payment-api
operations:
- createCheckoutSession
- getCheckoutStatus
generated: '2026-09-19'
method: generated
source: Grounded in openapi/ operationIds, conventions/coinrailz-com-conventions.yml, errors/, rate-limits/ and
  the provider's agent-instructions.json; no operation named here is invented.
---

# Buy prepaid credits by card (hosted checkout)

Fund an agent with prepaid credits through Stripe hosted checkout and poll for the auto-provisioned API key.

1. `POST https://coinrailz.com/api/m2m/credits/checkout/session` (`createCheckoutSession`) with `{"amountUsd": 10}` (tiers published: $5 ~80-100 calls, $10 ~200, $25 ~500 [recommended], $100 ~2,000; card range $1-$2,500). No auth is required.
2. The response carries `checkoutUrl` and a `retrievalToken`. A human operator must open `checkoutUrl` in a browser and pay; **save the retrievalToken** - it is the only way to collect the key.
3. Poll `GET https://coinrailz.com/api/m2m/credits/checkout/status/{sessionId}?token=<retrievalToken>` (`getCheckoutStatus`) until it returns the `cr_live_` key; the provider states the key is auto-provisioned within ~60 s of payment.
4. Direct card charging exists as `POST /api/m2m/credits/purchase` with `{paymentMethodId, amountUsd, idempotencyKey}` (documented in agent-instructions.json, not in the OpenAPI). If you use it, generate a UUID v4 `idempotencyKey` per purchase and reuse the SAME key on any retry - it is the only replay protection on the payment surface.
5. No refund policy is published on a machine-readable surface; treat a credit purchase as final.
