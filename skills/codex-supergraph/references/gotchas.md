# Codex Supergraph Gotchas

Common failure points. Check here first when a query returns unexpected results.

## `getBars` / `getTokenBars` symbol format

The `symbol` parameter is `pairAddress:networkId`, not a ticker or token address.

- Correct: `"0xabc123def456:1"`
- Wrong: `"ETH"`, `"0xabc123def456"`

## `getBars` missing timestamps

Always include `t` in the selection set. Without it the bars are unplottable. The fields are `t o h l c volume`.

## `getBars` max datapoints

Max 1500 datapoints per request. If `from`/`to` spans more than 1500 bars at the given resolution, you'll get a truncated response. Narrow the window or increase the resolution.

## `filterTokens` pagination

Max 200 results per call. Use `offset` to paginate — not a cursor. First page: `offset: 0`, second: `offset: 200`, etc. The response includes `count` for the total.

## `filterTokens` trending queries need `statsType`

When ranking by `trendingScore24`, set `statsType: "FILTERED"` or you'll get zero/null scores. Also use `trendingIgnored: false` and `potentialScam: false` in filters to exclude noise.

## `pairMetadata` ID format

The `pairId` parameter is `pairAddress:networkId`, same format as `getBars` symbol.

## `networkId` validation

Always validate `networkId` against `getNetworks` before using it. A wrong ID returns empty results, not an error. Common IDs: Ethereum=`1`, Solana=`1399811149`, Base=`8453`.

## Event pagination uses cursors, not offsets

`getTokenEvents` and `holders` use `cursor` from the previous response. Don't use `offset` — it's not supported on these endpoints.

## Subscription fan-out

Use `onPricesUpdated` (batch) instead of opening many `onPriceUpdated` (single) subscriptions. Batch input supports ~25 tokens per subscription.

## Rate limits return 429, not GraphQL errors

Rate limit responses are HTTP 429, not wrapped in GraphQL `errors`. Your error handling needs to check the HTTP status, not just the response body.

## Short-lived tokens can't manage themselves

`apiTokens`, `apiToken`, and `deleteApiToken` are not available when authenticated with a short-lived token. Use the long-lived API key for token management.
