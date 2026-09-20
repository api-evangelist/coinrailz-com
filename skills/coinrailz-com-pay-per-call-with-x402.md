---
name: Pay for a single call with x402 USDC
description: Answer a Coin Railz HTTP 402 challenge with a signed on-chain USDC payment and retry the request.
api: coinrailz-com:x402-services
operations:
- ping
- first-call
- tokenPrice
generated: '2026-09-19'
method: generated
source: Grounded in openapi/ operationIds, conventions/coinrailz-com-conventions.yml, errors/, rate-limits/ and
  the provider's agent-instructions.json; no operation named here is invented.
---

# Pay for a single call with x402 USDC

Answer a Coin Railz HTTP 402 challenge with a signed on-chain USDC payment and retry the request.

1. POST the service with no payment, e.g. `POST https://coinrailz.com/x402/ping` (`ping`). Expect **402** with a `PAYMENT-REQUIRED` header (base64 JSON, `x402Version: 2`) and a JSON body that repeats the price and lists alternative paths (trial key, hosted checkout).
2. Decode `PAYMENT-REQUIRED` and pick ONE `accepts[]` entry - never hardcode payment details (llms.txt: "Agents must use the exact accepts entry from a live HTTP 402 challenge"). Entries observed: Base `eip155:8453` USDC `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` (EIP-712 domain `USD Coin` v2), Arc `eip155:5042` via Circle Gateway, and Solana mainnet. `maxTimeoutSeconds` (60 on Base) bounds how long the signed payload is valid.
3. Sign the `exact` scheme payload for the chosen network (EIP-712 `transferWithAuthorization` on EVM) with an x402 client library (`npm install x402` / `pip install x402`, per mcp-integration.json) and retry the same POST with the payload in `X-PAYMENT` (also accepted as `PAYMENT-SIGNATURE`).
4. On success the body is the service result; the facilitator is `https://api.cdp.coinbase.com/platform/v2/x402`. A processing fee of 1.5% + $0.01 applies per transaction.
5. **There is no refund or reversal for a settled x402 payment** (conventions/ reversibility) and no idempotency key on /x402/* - a retried request with a fresh signature is a second charge. Retry only on a transport failure where no response was received, and prefer `first-call` ($0.05, `first-call`) as the cheapest end-to-end rehearsal.
