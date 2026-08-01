---
name: Place and manage a Ripio Trade order
description: Authenticate, size an order against live market data, submit it, and track/cancel it by client external_id on the Ripio Trade v4 API.
api: openapi/ripio-trade-openapi.yml
operations: [GetPairs, EstimatePrice, ListBalances, CreateOrder, GetUserOpenOrders, GetOrderByExternalId, CancelOrderByExternalId]
---

# Place and manage a Ripio Trade order

Base URL: `https://api.ripiotrade.co/v4/`. Private routes require the `Authorization`
(API Token), `Timestamp` (ms), and `Signature` (Base64 SHA-256 HMAC over the payload
with your Secret Key) headers. See `authentication/ripio-authentication.yml`.

## Steps

1. **List tradable pairs** — call `GetPairs` (`GET /pairs`) to confirm the pair symbol
   (e.g. `BTC_BRL`), its `min_amount` and price tick.
2. **Check balance** — call `ListBalances` (`GET /user/balances`) to confirm you hold
   enough of the quote currency (buy) or base currency (sell).
3. **Estimate price** — call `EstimatePrice` (`GET /orders/estimate-price/{pair}`) to
   preview execution price before committing.
4. **Create the order** — call `CreateOrder` (`POST /orders`). Set a client-supplied
   `external_id` so the order has a stable idempotent reference. Respect the pair's
   `min_amount` or you get `error_code 40010` (order amount below minimum); insufficient
   funds returns `40011`.
5. **Track it** — call `GetUserOpenOrders` (`GET /orders/open`), or fetch by your own id
   with `GetOrderByExternalId` (`GET /orders/by-external-id/{external_id}`).
6. **Cancel if needed** — call `CancelOrderByExternalId` (`DELETE /orders/by-external-id`)
   using the same `external_id`. `40020` = already canceled, `40021` = already executed.

## Rules
- Use Market/authenticated endpoints for trading; Public endpoints are cached up to 30s.
- Honor rate limits (`rate-limits/ripio-rate-limits.yml`); a 429 returns `error_code 42900`.
- All errors use `{ error_code, message, data }` — branch on `error_code`
  (`errors/ripio-error-codes.yml`).
