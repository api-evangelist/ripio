---
name: Withdraw cryptocurrency from Ripio Trade
description: Resolve a destination wallet, estimate the network fee, and submit a cryptocurrency withdrawal on the Ripio Trade v4 API, handling duplicate-prevention and limit errors.
api: openapi/ripio-trade-openapi.yml
operations: [GetUserWallets, GetUserWalletAndCreateIfNotExists, GetWithdrawalFee, CreateWithdrawal, GetUserWithdrawals]
---

# Withdraw cryptocurrency from Ripio Trade

Base URL: `https://api.ripiotrade.co/v4/`. The API credential must carry the
`Cryptocurrency withdrawals` access type in addition to `Read`. Same
Authorization/Timestamp/Signature headers as all private routes.

## Steps

1. **Find or create the wallet** — call `GetUserWallets` (`GET /wallets`) to list wallets,
   or `GetUserWalletAndCreateIfNotExists` (`GET /wallets/{currency_code}/{network}`) to
   ensure a wallet exists for the currency + network.
2. **Estimate the fee** — call `GetWithdrawalFee`
   (`GET /withdrawals/estimate-fee/{currency_code}`) to preview the miner/network fee.
3. **Create the withdrawal** — call `CreateWithdrawal` (`POST /withdrawals`) with the
   currency, amount, network and destination. Below-minimum returns `error_code 40031`;
   over-limit returns `40030`; an unauthorized destination returns `40033`.
4. **Confirm** — call `GetUserWithdrawals` (`GET /withdrawals`) to see the withdrawal and
   its status.

## Rules
- **Duplicate prevention is server-side:** an identical withdrawal returns `error_code
  40032` ("Prevention against duplicated withdrawal. Try again after ten minutes."). Do
  not blindly retry — poll `GetUserWithdrawals` first.
- Withdrawals may be blocked (`40040`) or blocked for TFA recovery (`40041`); TFA must be
  enabled (`40050`).
- Errors use the `{ error_code, message, data }` envelope (`errors/ripio-error-codes.yml`).
