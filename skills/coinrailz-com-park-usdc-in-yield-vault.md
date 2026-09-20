---
name: Park idle USDC in the yield vault
description: Deposit idle agent USDC into the ERC-4626 vault on Base between trades and redeem it on demand, non-custodially.
api: coinrailz-com:agent-payment-api
operations:
- getYieldManifest
- getYieldRates
- getYieldStats
- getDepositTx
- getYieldPosition
- getRedeemTx
- getYieldContract
generated: '2026-09-19'
method: generated
source: Grounded in openapi/ operationIds, conventions/coinrailz-com-conventions.yml, errors/, rate-limits/ and
  the provider's agent-instructions.json; no operation named here is invented.
---

# Park idle USDC in the yield vault

Deposit idle agent USDC into the ERC-4626 vault on Base between trades and redeem it on demand, non-custodially.

1. Read `GET /api/yield/manifest` (`getYieldManifest`) and `GET /api/yield/rates` (`getYieldRates`): current APY, the active protocol (Aave v3 / Compound v3 / Morpho Blue) and fees - entry 0.5%, performance 15% of yield, exit 0%, switch 0%. All yield endpoints are public (no key).
2. Fetch `GET /api/yield/contract` (`getYieldContract`) for the vault address (`0xb7697bf34f1566dd3d19792e12c366e396816736`, share token crUSDC, ERC-4626) and ABI if you want to verify calldata before signing.
3. Build the deposit with `GET /api/yield/deposit-tx?preset=<usd>&recipient=<0xwallet>` (`getDepositTx`). Substitute the real wallet - the manifest warns not to pass `{wallet}` literally. Standard mode returns two transactions (approve + deposit); permit mode returns one. **Coin Railz never holds funds**: the agent signs and submits the transactions itself.
4. Track the position with `GET /api/yield/position/{wallet}` (`getYieldPosition`): shares, USD value, yield earned, and a `redeem_hint` ready-to-sign withdraw tx.
5. To exit, `GET /api/yield/redeem-tx?wallet=<0xwallet>` (`getRedeemTx`) burns crUSDC for USDC in one transaction. The manifest states "No lockup. Near-instant redemption" with a 0% exit fee, but no bounded window is published - do not promise a settlement time to a principal.
