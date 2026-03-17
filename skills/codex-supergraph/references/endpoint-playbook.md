# Codex Supergraph Endpoint Playbook

Use this map to pick the best operation quickly.

## Discovery and search

| Intent | Preferred query | Notes |
| --- | --- | --- |
| Discover tradable tokens with ranking/filtering | `filterTokens` | Supports phrase search and quality filters; response limit 200. |
| Trending tokens | `filterTokens` | Rank by `trendingScore24` DESC with `statsType: "FILTERED"`. Use `trendingIgnored: false` and `potentialScam: false` filters to exclude noise. |
| Find pairs for a token | `listPairsWithMetadataForToken` | Useful before charting or pair-specific metadata. |
| Find wallet leaders for a token | `filterTokenWallets` | Good for holder quality and top-wallet analysis. |

## Pricing and charts

| Intent | Preferred query | Preferred subscription | Notes |
| --- | --- | --- | --- |
| Get fast current or historical token price snapshots | `getTokenPrices` | `onPriceUpdated` (single token) | Uses aggregate weighted prices unless source pair is provided. |
| Stream prices for many tokens | N/A | `onPricesUpdated` | Batch token inputs in one subscription; start around 25 tokens. |
| Build pair chart candles | `getBars` | `onBarsUpdated` | `getBars` max datapoints is 1500 per request. |
| Build aggregate token chart candles | `getTokenBars` | `onTokenBarsUpdated` | Aggregates across top liquidity pairs. |
| Fetch pair metrics and stat windows | `pairMetadata` | `onPairMetadataUpdated` | Use query for initial load, subscription for live refresh. |

## Events and flows

| Intent | Preferred query | Preferred subscription | Notes |
| --- | --- | --- | --- |
| Fetch token transactions | `getTokenEvents` | `onTokenEventsCreated` | Choose subscription for live feeds. |
| Fetch maker-specific activity | `getTokenEventsForMaker` | `onEventsCreatedByMaker` | Use maker address filters. |
| Track unconfirmed Solana events | N/A | `onUnconfirmedEventsCreated` | Solana-focused unconfirmed flow. |

## Wallet analytics

| Intent | Preferred query | Notes |
| --- | --- | --- |
| Wallet-level chart and performance data | `walletChart`, `detailedWalletStats` | Use date windows and aggregation granularity intentionally. |
| Holder distribution | `holders`, `top10HoldersPercent` | Combine with token metadata for context. |

## Launchpads and high-throughput streams

| Intent | Preferred subscription | Notes |
| --- | --- | --- |
| Launchpad token event firehose | `onLaunchpadTokenEventBatch` | High-frequency stream; isolate connection and proxy through backend. |
| Per-event launchpad updates | `onLaunchpadTokenEvent` | Use when batch handling is not needed. |

## Auth and token management

| Intent | Preferred mutation/query | Notes |
| --- | --- | --- |
| Create short-lived API tokens | `createApiTokens` | Use long-lived key for creation. |
| List existing API tokens | `apiTokens` | Not available with short-lived keys. |
| Revoke API token | `deleteApiToken` | Use server-side secret key context. |

## Selection heuristics

- Prefer one-shot queries for initial page hydration.
- Add subscriptions only where visible realtime UX exists.
- Choose aggregated endpoints when user intent is token-level, not pool-level.
- Keep fields minimal for first pass; expand only after payload shape is verified.
