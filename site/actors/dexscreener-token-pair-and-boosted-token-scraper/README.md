# DexScreener Token Pair & Boosted Token Scraper

DexScreener is the market screen for on-chain trading: live pairs across DEXs and dozens of chains, plus the paid boosts, profiles, takeovers, ads and narratives that drive attention. This actor reads DexScreener's official public API and returns clean, analysis-ready records for token research, pair monitoring and promotion tracking. No wallet, API key, cookie or proxy is needed.

## What this actor does

- **Search** tokens and pairs by name, symbol, pair text such as `SOL/USDC`, contract address, or a pasted DexScreener URL.
- **Inspect a token**: best pair per token on one chain, or every pool of the token.
- **Monitor pairs** by address or URL, many at a time.
- **Pull six promotion feeds**: most-boosted tokens, newest boosts, newest profiles, recently updated profiles, community takeovers and paid ads.
- **Follow narratives**: trending metas, then the pairs inside each meta.
- **Check paid orders**: profile, ad and takeover orders plus boost purchases for a token.
- **Filter and sort** by liquidity, 24h volume, market cap, FDV, transactions, 24h price change, pair age, DEX and quote symbol.
- **Token info and risk flags**: logo, header, website and social links when available; heuristic flags such as low liquidity, new pair, sell pressure or no socials.

## Output per record type

Empty fields are omitted. Every record has `recordType`, `siteName` and `scrapedAt`.

### Pair (`pair`)

- `chainId`, `dexId`, `pairAddress`, `pairUrl`
- `id` - `chain:pairAddress`
- `tokenAddress`, `tokenName`, `tokenSymbol`, `quoteTokenAddress`, `quoteTokenSymbol`
- `baseToken`, `quoteToken` - `{address, name, symbol}`
- `priceNative`, `priceUsd`
- `liquidity` - `{usd, base, quote}`
- `fdv`, `marketCap`
- `volume` - `{h24, h6, h1, m5}`
- `priceChange` - `{h24, h6, h1, m5}`
- `txns` - buy/sell counts for the same windows
- `pairCreatedAt`, `ageHours`
- `labels`, `boosts`
- `buySellRatio24h`, `volumeToLiquidity24h`
- `info` - `{imageUrl, header, openGraph, websites[], socials[]}`
- `flags[]` - `low_liquidity`, `no_liquidity_data`, `suspicious_liquidity`, `new_pair_<N>h`, `sell_pressure`, `no_sells_24h`, `extreme_price_move_24h`, `volume_over_10x_liquidity`, `no_profile`, `no_socials`
- `query`, `rank`, `metaSlug`, `metaName` when relevant
- `querySide`, `tokenPriceUsd` for token lookups where the token is the quote side

### Boosted token (`boostedToken`) and token profile (`tokenProfile`)

- `chainId`, `tokenAddress`, `tokenUrl`, `pairUrl`
- `icon`, `header`, `openGraph`, `description`, `links[]`
- `boostAmount`, `boostTotalAmount` (boosts)
- `cto` (profiles), `updatedAt` (recently updated profiles)
- `rank`, `listType`
- Market data when enabled: `symbol`, `name`, `priceUsd`, `quoteSymbol`, `dexId`, `liquidityUsd`, `fdv`, `marketCap`, `volume24h`, `priceChange1h`, `priceChange24h`, `pairCreatedAt`, `ageHours`, `pairsFound`, `topPairAddress`, `topPairUrl`

### Community takeover (`communityTakeover`) and ad (`ad`)

- Takeover: `chainId`, `tokenAddress`, `tokenUrl`, `pairUrl`, `icon`, `header`, `openGraph`, `description`, `links[]`, market data, plus `claimDate`.
- Ad: `tokenAddress`, `tokenUrl`, `adDate`, `adType`, `adImpressions`, optional `adDurationHours`, plus market data.

### Narrative (`meta`)

- `slug`, `name`, `description`, `icon`, `iconType`
- `marketCap`, `liquidity`, `volume`, `tokenCount`
- `marketCapChange`, `marketCapDelta` for `m5`, `h1`, `h6`, `h24`
- `rank`

### Paid orders (`orders`)

- `chainId`, `tokenAddress`
- `ordersCount`, `approvedCount`, `hasApprovedProfile`, `hasPaidAds`, `lastPaymentAt`
- `orders[]` - `{type, status, paymentTimestamp}`
- `boosts[]` - `{id, amount, paymentTimestamp}`
- `totalBoostAmount`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byToken`, `byPair`, `discovery`, `metas`, `metaPairs`, `orders` |
| `queries` | array | – | Names, symbols, `SOL/USDC`, addresses or DexScreener URLs (mode=search) |
| `tokenAddresses` | string | - | Token addresses or URLs, comma/newline separated (mode=byToken, orders) |
| `pairAddresses` | array | - | Pair URLs, `chain:pairAddress`, or bare pair addresses (mode=byPair) |
| `chainId` | string | `all` | Chain scope; required for byToken (unless `all`), bare pairs, and orders |
| `customChainId` | string | - | Override for a chain added after this release |
| `feed` | string | `boostedTop` | `boostedTop`, `boostedLatest`, `profilesLatest`, `profilesUpdated`, `takeovers`, `ads` |
| `metaSlugs` | array | - | Narrative slugs such as `ai`, `dog`, `cat` (mode=metaPairs) |
| `allPools` | boolean | `false` | byToken: every pool of the token instead of the best pair |
| `minLiquidityUsd` / `maxLiquidityUsd` | number | - | Keep pairs inside this liquidity range |
| `minVolume24hUsd` / `maxVolume24hUsd` | number | - | Keep pairs inside this 24h volume range |
| `minMarketCapUsd` / `maxMarketCapUsd` | number | - | Uses market cap, or FDV when market cap is missing |
| `minFdvUsd` / `maxFdvUsd` | number | - | Keep pairs inside this FDV range |
| `minTxns24h` / `maxTxns24h` | integer | - | Buys plus sells in the last 24 hours |
| `minPriceChange24h` / `maxPriceChange24h` | number | - | 24h price change in percent; negatives allowed |
| `minAgeHours` / `maxAgeHours` | number | - | Pair age in hours |
| `dexIds` | array | - | Keep only these DEX protocols |
| `quoteSymbols` | array | - | Keep only these quote assets |
| `sortBy` | string | `none` | Sort by liquidity, volume, market cap, FDV, change, transactions, age or boosts |
| `sortOrder` | string | `desc` | `desc` or `asc`; age ascending means newest first |
| `onePairPerToken` | boolean | `false` | Keep the most liquid pair per token |
| `includeInfo` | boolean | `true` | Add logo, header, websites and socials |
| `includeMarketData` | boolean | `true` | Attach best-pair market data to promotion feeds |
| `expandChainSearch` | boolean | `true` | Widen a chain-filtered search past the 30-pair cap |
| `chainSearchTokens` | integer | `10` | Matching tokens whose pools are fetched when widening |
| `includeFlags` | boolean | `true` | Add heuristic risk flags |
| `newPairHours` | integer | `24` | Threshold for the new-pair flag |
| `maxItemsPerQuery` | integer | `30` | Cap per query, token, feed or meta |
| `maxItems` | integer | `100` | Cap for the whole run |

### Example: search across all chains

```json
{
  "mode": "search",
  "queries": ["PEPE", "WIF", "AERO"],
  "sortBy": "volume24h",
  "maxItems": 50
}
```

Older saved tasks that use `query`, `pairAddress`, `minVolumeH24`, or the previous mode names (`byTokenAddress`, `byPairAddress`, `trending`, `byProfile`) keep working.

### Example: every pool of a token on Solana

```json
{
  "mode": "byToken",
  "chainId": "solana",
  "tokenAddresses": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "allPools": true,
  "minLiquidityUsd": 10000,
  "maxItems": 30
}
```

### Example: newest boosted tokens with market data

```json
{
  "mode": "discovery",
  "feed": "boostedLatest",
  "includeMarketData": true,
  "minLiquidityUsd": 5000,
  "maxItems": 30
}
```

### Example: paid orders for a token

```json
{
  "mode": "orders",
  "chainId": "solana",
  "tokenAddresses": "So11111111111111111111111111111111111111112",
  "maxItems": 1
}
```

## Use cases

- **Token screening** - rank pairs by liquidity, volume, age or price change before deeper research.
- **Liquidity mapping** - pull every pool of a token to see where it actually trades.
- **Launch monitoring** - watch newest profiles, boosts, takeovers and ads for emerging projects.
- **Narrative research** - follow trending metas and compare the pairs inside them.
- **Promotion due diligence** - check whether a token paid for profiles, ads or boosts.
- **Scheduled snapshots** - keep a time series of prices, liquidity and volume for a watchlist.

## FAQ

### Is this affiliated with DexScreener?

No. This is an independent actor that reads DexScreener's public API. Data belongs to DexScreener.

### Do I need a wallet, API key or proxy?

No. Every mode works without an account, key or proxy.

### Why did I get fewer records than `maxItems`?

DexScreener caps a search at 30 pairs per query and has no pagination. Pools per token are capped at 30. The promotion feeds return at most 30 rows for boosts, profiles and ads, 11 for takeovers, 18 trending narratives and 49 pairs per narrative feed. Repeated boosts or ads for the same token are collapsed into one record. Use multiple queries, `byToken` with `allPools`, or several chains to widen coverage.

### How do I get every pool of a token?

Use `byToken` with a specific chain and `allPools: true`. A search is capped at 30 pairs across all chains; pool lookup is scoped to the token and chain.

### Are the filters applied by DexScreener?

No. The public API has no filter parameters, so filters and sorting run in the actor after the fetch. That also means a filter cannot recover rows beyond the source cap.

### Why are `info` or some flags missing from a pair?

Token images, websites and socials are optional on DexScreener, and some market fields do not exist for brand-new pairs. Missing values are omitted, not sent as null. Flags are heuristics for triage, not verdicts.

### What is the difference between the boost and profile feeds?

`boostedTop` and `boostedLatest` return tokens with paid boosts (ranked or newest). `profilesLatest` and `profilesUpdated` return token profile submissions, and `takeovers` returns community takeover events.

### Does this actor include OHLC candles, holders or top traders?

No. Those are not part of DexScreener's official public API. This actor covers pairs, pools, promotion feeds, narratives and paid orders.

### How fresh is the data?

Every run reads DexScreener live, so prices, liquidity, volume and promotions reflect the market at run time.

### Is this financial advice?

No. It is market data for research and monitoring; verify token contracts independently.
