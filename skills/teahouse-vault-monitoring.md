---
name: Monitor Teahouse vault performance and flows
description: Fetch the Teahouse vault catalog, read a vault's performance over time, and pull share transaction logs for a vault or wallet.
api: openapi/teahouse-vault-openapi.yml
operations: [listVaults, listPermissionlessVaults, getPermissionlessVaultPerformance, getVaultShareTransactions, getAccountShareTransactions]
---

# Monitor Teahouse vault performance and flows

The Teahouse Vault API is a read-only HTTP/JSON API. Base URL: `https://vault-content-api.teahouse.finance`. All responses are JSON. Errors return `{ "error": string, "details"?: array }` with status `400` (bad request), `401` (API key verification failed), or `500` (server error). This API has no write operations and no idempotency contract.

## Steps

1. **Discover vaults.** Call `listVaults` (`GET /vaults`) for the full catalog, or `listPermissionlessVaults` (`GET /vaults/type/permissionless`) for permissionless vaults with latest TVL, fee APR, and share-token APR. Note each vault's `chainID` and contract `address` — you need both to query performance and logs.

2. **Read performance over time.** Call `getPermissionlessVaultPerformance` (`GET /vaults/permissionless/performance/{chainID}/{contractAddress}/{start}/{end}`) with the vault's `chainID` (e.g. `137` for Polygon), `contractAddress`, and a `start`/`end` Unix-timestamp window. Returns TVL, APR, and share-price time series.

3. **Pull share transaction logs.** For a vault, call `getVaultShareTransactions` (`GET /vaults/permissionless/log/shares/{chainID}/{contractAddress}/{startTimestamp}/{endTimestamp}`). For a specific wallet, call `getAccountShareTransactions` (`GET /vaults/permissionless/log/{address}/{startTimestamp}/{endTimestamp}`). Log entries carry `action` (deposit / withdraw / transfer_from / transfer_to), `amount`, `blockNumber`, `timestamp`, and `transactionID`.

## Rules

- Timestamps are Unix seconds; keep `start` <= `end`.
- If an endpoint requires an API key, supply it in the request header (the exact header name is not published by Teahouse — confirm with the provider). Public catalog reads generally succeed without a key.
- On `500`, retry with backoff; on `400`, re-validate `chainID`, addresses, and the timestamp range.
