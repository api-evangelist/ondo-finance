---
name: Quote and mint an Ondo Stock (Global Markets)
description: Get a live price, request a non-binding soft quote, then request a binding mint/redeem attestation for a tokenized stock on the Ondo GM Backend API.
api: openapi/ondo-finance-gm-openapi-original.json
operations: [getMarketStatus, getPrice, createSoftAttestationQuote, createAttestation, getTradingLimits]
---

# Quote and mint an Ondo Stock

Authenticate every request with the `x-api-key` header. Keys are onboarding-gated
(onboarding@ondo.finance); a read-only key can call the read steps but will return
`READ_ONLY_API_KEY` on `createAttestation`.

## Steps
1. **Check the market is open** — call `getMarketStatus` (`GET /v1/status/market`).
   If closed you will get `MARKET_CLOSED`/`MARKET_PAUSED`; stop here.
2. **Get the latest price** — call `getPrice` (`GET /v1/assets/{symbol}/prices/latest`)
   for the symbol (e.g. `TSLAon`). `primaryMarket` is the on-chain token, `underlyingMarket`
   is the off-chain stock.
3. **Confirm capacity** — call `getTradingLimits` (`GET /v1/limits/trading`) to check the
   real-time per-user limit for the symbol before committing.
4. **Soft quote (non-binding)** — call `createSoftAttestationQuote`
   (`POST /v1/attestations/soft`) with `{symbol, side, notionalValue, duration}`.
   This does not impact trading limits; use it to preview pricing.
5. **Binding attestation** — when ready, call `createAttestation`
   (`POST /v1/attestations`). The response includes `signature` and `expiration`;
   submit the signature on-chain to the Ondo Stocks contract with USDC to mint.

## Rules
- Honor `duration` (`short` = tighter spread, `long` = longer validity); attestations expire.
- Handle error codes from errors/ondo-finance-error-codes.yml (MARKET_CLOSED, ASSET_PAUSED,
  ASSET_REDEEM_ONLY, SESSION_LIMIT_REACHED, RATE_LIMITED …).
- On `429`/`RATE_LIMITED`, back off and retry.
