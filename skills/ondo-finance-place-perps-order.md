---
name: Place an Ondo Perps order
description: Authenticate via Sign-In With Ethereum, inspect markets, place a perpetual futures order, then track positions and fills on the Ondo Perps API.
api: openapi/ondo-finance-perps-openapi-original.json
operations: [getSIWEChallenge, completeSIWEChallenge, getMarkets, createOrder, getOrders, getPerpsPositions, getPerpsBalance]
---

# Place an Ondo Perps order

## Steps
1. **Authenticate (SIWE / EIP-4361)** — call `getSIWEChallenge`
   (`POST /v1/auth/erc-4361/login/get_challenge`), sign the returned message with the
   wallet, then `completeSIWEChallenge` (`POST /v1/auth/erc-4361/login/complete_challenge`)
   to receive a JWT. Send it as `Authorization: Bearer <jwt>` on subsequent calls.
   (Programmatic clients may instead use an `X-API-KEY-ID` key created via `createApiKey`.)
2. **List markets** — call `getMarkets` (`GET /v1/markets`) to resolve the tradable symbol
   and contract parameters.
3. **Check balance** — call `getPerpsBalance` (`GET /v1/perps/balance`) to confirm margin.
4. **Place the order** — call `createOrder` (`POST /v1/perps/orders`) with the symbol, side,
   size and order type.
5. **Track it** — poll `getOrders` (`GET /v1/perps/orders`) and `getPerpsPositions`
   (`GET /v1/perps/positions`), or subscribe to the WebSocket channels
   `subscribe_ordersPerps` / `subscribe_positionsPerps` (openapi/ondo-finance-perps-ws-openapi-original.json).

## Rules
- The API has no idempotency-key contract; do not blindly retry `createOrder` on a timeout —
  reconcile with `getOrders` first.
- Respect pagination: pass `cursor`/`limit` and follow `nextCursor`.
- WebSocket is capped at 25 req/s (burst 50), 32 KB max message.
- On `429`, back off. JWTs can be revoked with `invalidateJWT`.
