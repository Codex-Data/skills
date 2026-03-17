# Codex Supergraph Query Templates

## 1) API-key query over HTTPS

```bash
curl -sS https://graph.codex.io/graphql \
  -H 'Content-Type: application/json' \
  -H "Authorization: $CODEX_API_KEY" \
  --data-binary @- <<'JSON'
{
  "query": "query GetNetworks { getNetworks { id name } }"
}
JSON
```

## 2) API-key query with variables

```bash
curl -sS https://graph.codex.io/graphql \
  -H 'Content-Type: application/json' \
  -H "Authorization: $CODEX_API_KEY" \
  --data-binary @- <<'JSON'
{
  "query": "query GetTokenPrices($inputs: [GetPriceInput!]!) { getTokenPrices(inputs: $inputs) { address networkId priceUsd timestamp } }",
  "variables": {
    "inputs": [
      { "address": "So11111111111111111111111111111111111111112", "networkId": 1399811149 }
    ]
  }
}
JSON
```

## 3) Token discovery (`filterTokens`)

```graphql
query FilterTokens(
  $filters: TokenFilters
  $statsType: TokenPairStatisticsType
  $rankings: [TokenRanking]
  $limit: Int
  $offset: Int
) {
  filterTokens(
    filters: $filters
    statsType: $statsType
    rankings: $rankings
    limit: $limit
    offset: $offset
  ) {
    count
    offset
    results {
      buyVolume24
      sellVolume24
      circulatingMarketCap
      liquidity
      txnCount24
      trendingScore24
      token {
        info {
          address
          name
          symbol
          networkId
        }
      }
    }
  }
}
```

Example variables — top tokens on Solana by volume:

```json
{
  "filters": {
    "network": [1399811149],
    "liquidity": { "gte": 10000 }
  },
  "rankings": [{ "attribute": "volume24", "direction": "DESC" }],
  "limit": 25,
  "offset": 0
}
```

Example variables — trending tokens:

```json
{
  "filters": {
    "volume24": { "lte": 100000000000 },
    "liquidity": { "lte": 1000000000 },
    "marketCap": { "gte": 500000, "lte": 1000000000000 },
    "trendingIgnored": false,
    "creatorAddress": null,
    "potentialScam": false
  },
  "statsType": "FILTERED",
  "rankings": [{ "attribute": "trendingScore24", "direction": "DESC" }],
  "limit": 50,
  "offset": 0
}
```

## 4) Pair metadata (`pairMetadata`)

```graphql
query PairMetadata($pairId: String!) {
  pairMetadata(pairId: $pairId) {
    id
    pairAddress
    networkId
    liquidity
    price
    priceChange24
    volume24
    token0 {
      address
      symbol
      name
    }
    token1 {
      address
      symbol
      name
    }
  }
}
```

## 5) Pair bars (`getBars`)

`symbol` format is `pairAddress:networkId` (e.g., `"0xabc123:1"`). `from`/`to` are Unix timestamps in seconds. `resolution` examples: `"1"` (1 min), `"5"`, `"15"`, `"60"`, `"240"`, `"1D"`, `"1W"`. Max 1500 datapoints per request.

```graphql
query GetBars(
  $symbol: String!
  $from: Int!
  $to: Int!
  $resolution: String!
  $countback: Int
  $removeEmptyBars: Boolean
) {
  getBars(
    symbol: $symbol
    from: $from
    to: $to
    resolution: $resolution
    countback: $countback
    removeEmptyBars: $removeEmptyBars
  ) {
    t
    o
    h
    l
    c
    volume
  }
}
```

## 6) Single-token realtime (`onPriceUpdated`)

```graphql
subscription OnPriceUpdated($address: String!, $networkId: Int!) {
  onPriceUpdated(address: $address, networkId: $networkId) {
    address
    networkId
    priceUsd
    timestamp
    blockNumber
  }
}
```

## 7) Multi-token realtime (`onPricesUpdated`)

```graphql
subscription OnPricesUpdated($input: [OnPricesUpdatedInput!]!) {
  onPricesUpdated(input: $input) {
    address
    networkId
    priceUsd
    timestamp
    blockNumber
  }
}
```

Example variables:

```json
{
  "input": [
    { "address": "So11111111111111111111111111111111111111112", "networkId": 1399811149 },
    { "address": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2", "networkId": 1 }
  ]
}
```

## 8) Token bars (`getTokenBars`)

Aggregates OHLCV across top liquidity pairs for a token. Uses same `resolution` values as `getBars`.

```graphql
query GetTokenBars(
  $symbol: String!
  $from: Int!
  $to: Int!
  $resolution: String!
  $countback: Int
  $removeEmptyBars: Boolean
) {
  getTokenBars(
    symbol: $symbol
    from: $from
    to: $to
    resolution: $resolution
    countback: $countback
    removeEmptyBars: $removeEmptyBars
  ) {
    t
    o
    h
    l
    c
    volume
  }
}
```

## 9) List pairs for a token (`listPairsWithMetadataForToken`)

```graphql
query ListPairs($tokenAddress: String!, $networkId: Int!) {
  listPairsWithMetadataForToken(tokenAddress: $tokenAddress, networkId: $networkId) {
    results {
      pair {
        address
        networkId
        token0
        token1
      }
      volume24
      liquidity
      price
    }
  }
}
```

## 10) Token events (`getTokenEvents`)

```graphql
query GetTokenEvents(
  $address: String!
  $networkId: Int!
  $cursor: String
  $limit: Int
) {
  getTokenEvents(
    address: $address
    networkId: $networkId
    cursor: $cursor
    limit: $limit
  ) {
    cursor
    items {
      timestamp
      eventType
      priceUsd
      token0ValueBase
      token1ValueBase
      maker
      transactionHash
    }
  }
}
```

## 11) Maker events (`getTokenEventsForMaker`)

```graphql
query GetTokenEventsForMaker(
  $address: String!
  $networkId: Int!
  $maker: String!
  $cursor: String
  $limit: Int
) {
  getTokenEventsForMaker(
    address: $address
    networkId: $networkId
    maker: $maker
    cursor: $cursor
    limit: $limit
  ) {
    cursor
    items {
      timestamp
      eventType
      priceUsd
      token0ValueBase
      token1ValueBase
      transactionHash
    }
  }
}
```

## 12) Holders (`holders`)

```graphql
query Holders($input: HoldersInput!) {
  holders(input: $input) {
    cursor
    count
    holders {
      address
      balance
      sharedPct
    }
  }
}
```

## 13) Top-10 holder concentration (`top10HoldersPercent`)

```graphql
query Top10HoldersPercent($address: String!, $networkId: Int!) {
  top10HoldersPercent(address: $address, networkId: $networkId)
}
```

## 14) Wallet leaders (`filterTokenWallets`)

```graphql
query FilterTokenWallets(
  $address: String!
  $networkId: Int!
  $limit: Int
) {
  filterTokenWallets(
    address: $address
    networkId: $networkId
    limit: $limit
  ) {
    results {
      walletAddress
      balance
      balanceUsd
      pnlUsd
    }
  }
}
```

## 15) Wallet chart (`walletChart`)

```graphql
query WalletChart(
  $walletAddress: String!
  $networkId: Int!
  $from: Int!
  $to: Int!
  $resolution: String!
) {
  walletChart(
    walletAddress: $walletAddress
    networkId: $networkId
    from: $from
    to: $to
    resolution: $resolution
  ) {
    t
    balanceUsd
    pnlUsd
  }
}
```

## 16) WebSocket client (`graphql-ws`)

```typescript
import { createClient } from "graphql-ws";

const client = createClient({
  url: "wss://graph.codex.io/graphql",
  connectionParams: {
    Authorization: process.env.CODEX_API_KEY!,
  },
});

const unsubscribe = client.subscribe(
  {
    query: `
      subscription OnPriceUpdated($address: String!, $networkId: Int!) {
        onPriceUpdated(address: $address, networkId: $networkId) {
          address
          networkId
          priceUsd
          timestamp
          blockNumber
        }
      }
    `,
    variables: {
      address: "So11111111111111111111111111111111111111112",
      networkId: 1399811149,
    },
  },
  {
    next: (msg) => console.log(msg),
    error: (err) => console.error(err),
    complete: () => console.log("done"),
  }
);
```
