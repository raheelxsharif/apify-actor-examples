# DexScreener Token Pair & Boosted Token Scraper

DexScreener aggregates live trading pairs from decentralized exchanges across 20 blockchains. This actor pulls that data by keyword search, by token or pair address, or as the site's current boosted-token and token-profile feeds — all without a wallet or account.

## What this actor does

- Searches tokens and pairs by name, symbol, or address and returns matching pairs.
- Looks up every pair for one or more token contract addresses (up to 30 at once).
- Fetches a single pair by chain and pair address.
- Pulls the current boosted (promoted) tokens and the latest submitted token profiles.
- Filters pair results by minimum liquidity and minimum 24-hour volume in USD.
- Fields DexScreener doesn't report for a given pair or token (no fully diluted valuation, no labels) are left out of the record instead of being sent empty.

## Output per mode

### `search`, `byTokenAddress`, `byPairAddress` — trading pair

- `chainId` — blockchain the pair is on (e.g. `ethereum`, `solana`)
- `dexId` — DEX protocol name (e.g. `uniswap`, `raydium`)
- `pairAddress`, `pairUrl` — pair contract address and its DexScreener page
- `baseToken`, `quoteToken` — `{address, name, symbol}` for each side of the pair
- `priceNative`, `priceUsd` — price in native quote-token units and in USD
- `liquidity` — `{usd, base, quote}` liquidity amounts
- `fdv` — fully diluted valuation in USD
- `marketCap` — market cap in USD
- `volume` — trading volume by window: `h24`, `h6`, `h1`, `m5`
- `priceChange` — percentage price change by the same windows
- `txns` — buy/sell counts by the same windows
- `pairCreatedAt` — ISO 8601 pair creation date
- `labels` — tags such as `v2`, `v3`, or `pump.fun`
- `boosts` — `{active}` count of active boosts on the pair
- `recordType` — always `pair`
- `siteName` — always `DexScreener`
- `scrapedAt` — ISO 8601 timestamp of the fetch

### `trending` — boosted token

- `chainId`, `tokenAddress` — blockchain and token contract address
- `icon`, `header` — token image URLs
- `description` — token description text
- `pairUrl` — link to the token's DexScreener page
- `boostAmount`, `boostTotalAmount` — boost amount on this submission and the token's running total
- `links` — array of `{type, label, url}` social/website links
- `recordType` — always `boostedToken`
- `siteName` — always `DexScreener`
- `scrapedAt` — ISO 8601 timestamp of the fetch

### `byProfile` — token profile

- `chainId`, `tokenAddress` — blockchain and token contract address
- `icon`, `header` — token image URLs
- `description` — token description text
- `pairUrl` — link to the token's DexScreener page
- `links` — array of `{type, label, url}` social/website links
- `recordType` — always `tokenProfile`
- `siteName` — always `DexScreener`
- `scrapedAt` — ISO 8601 timestamp of the fetch

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byTokenAddress`, `byPairAddress`, `trending`, or `byProfile` |
| `query` | string | – | Token name, symbol, or address to search for; used by `search` |
| `tokenAddresses` | string | – | Comma-separated token contract addresses, up to 30; used by `byTokenAddress` |
| `chainId` | string | `all` | Blockchain network, or `all` for every chain (search mode only) |
| `pairAddress` | string | – | Pair contract address; used by `byPairAddress`, requires `chainId` set to a specific chain |
| `minLiquidityUsd` | number | – | Only keep pairs with total liquidity at or above this USD value |
| `minVolumeH24` | number | – | Only keep pairs with 24-hour volume at or above this USD value |
| `maxItems` | integer | `5` | Maximum number of records to return (1-1000) |

### Example: search across all chains

```json
{
  "mode": "search",
  "query": "USDC",
  "maxItems": 20
}
```

### Example: every pair for a token on Ethereum

```json
{
  "mode": "byTokenAddress",
  "tokenAddresses": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "chainId": "ethereum"
}
```

### Example: liquid, actively traded pairs only

```json
{
  "mode": "search",
  "query": "ETH",
  "minLiquidityUsd": 100000,
  "minVolumeH24": 50000,
  "maxItems": 30
}
```

### Example: currently boosted tokens

```json
{
  "mode": "trending",
  "maxItems": 50
}
```

## Use cases

- Screen for pairs above a liquidity and volume threshold before researching a token further.
- Pull every pair tied to a token contract to see where it's traded.
- Track which tokens are currently paying for DexScreener boosts.
- Monitor newly submitted token profiles for emerging projects.
- Compare price, volume, and transaction activity for a specific pair over repeated runs.

## FAQ

**Is this affiliated with DexScreener?**
No. This actor reads DexScreener's public pages and data; it is not operated by or connected to DexScreener.

**Do I need a wallet or API key?**
No. Every mode works without a wallet connection, account, or key.

**Do `minLiquidityUsd` and `minVolumeH24` apply to `byTokenAddress` and `byPairAddress` too?**
Yes — they apply to every mode that returns pair records (`search`, `byTokenAddress`, `byPairAddress`), not just `search`. A pair without liquidity or volume data is excluded once either filter is set. They have no effect on `trending` or `byProfile`, which don't return pair records at all.

**Can I search across every blockchain at once?**
Yes — leave `chainId` at its default of `all` to search across every supported chain.

**Why does a pair record have no `fdv` or `marketCap`?**
DexScreener doesn't publish a fully diluted valuation or market cap for every token, so those fields are left out when there's no value to report.

**What's the difference between `trending` and `byProfile`?**
`trending` returns tokens that currently have an active paid boost on DexScreener; `byProfile` returns the latest token profiles submitted to the site, regardless of whether they're boosted.

**How many records can one run return?**
Up to `maxItems`, capped at 1000 per run.

**How current is the data?**
Every run reads DexScreener live, so prices, liquidity, and volume reflect the current market at run time.
