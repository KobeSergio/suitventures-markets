# LongMarkets

**A stock-paired token market built on LONG and Pons, on Robinhood Chain. By SuitVentures.**

This file is the source of truth for the build. If the article, the proposal, chat, and this file disagree, this file wins. Every change to a decision is recorded in the Decision Log with a date and a source.

| | |
|---|---|
| Product name | LongMarkets |
| Brand | SuitVentures |
| Client | MAXLONG |
| Developer | Christian Kobe Sergio (Kobe) |
| Kickoff | Thursday, September 10, 2026 |
| v1 target | Friday, October 9, 2026 (day 30) |
| Sources | Build Proposal, agreed terms (Sep 9, 2026). Article "SuitVentures introduces LongMarkets" v1.0. Client answers in chat and Discord, Sep 10, 2026. |

---

## 1. What LongMarkets is

A mobile-first social trading feed for community tokens ("memestocks") quoted in Robinhood Stock Tokens and the other quote assets LONG supports. It is an aggregator plus a launch layer. It does three things:

1. **Indexes LONG and Pons.** Every pool launched through LONG's launcher appears in the feed. Every Pons v2 token that has graduated from its bonding curve into a Uniswap v4 pool appears too, provided it is quoted in an allowlisted asset. Pons tokens still on the curve, and Pons v1 tokens in v3 pools, do not appear.
2. **Routes swaps** through those pools with a fixed 0.5% protocol fee, via a thin wrapper that adds the fee and nothing else.
3. **Launches new tokens** through its own factory, with a Community / LP / Team split the LONG launcher does not support, and a presale that refunds in full if it misses target.

Token creation is gated to holders of The Suits NFT. Trading and contributing are open to any wallet.

### What LongMarkets is not

- **Not a fork of LONG or Pons.** It does not redeploy their contracts and does not fragment their liquidity. It reads their events and routes into their pools.
- **Not visible on LONG or Pons.** Tokens launched through the SuitVentures factory appear only on LongMarkets. The client accepted this trade to keep the presale and the token standard. The aggregator is the distribution strategy: traffic comes for LONG and Pons tokens and finds SV launches in the same feed.
- **Not a second quote list.** The quote assets are exactly the ones LONG offers as pairing options. The factory keeps an admin allowlist only because LONG has no on-chain list to read.
- **Not a custodian.** No admin key can pause a launched token, alter its supply, add a tax, or withdraw its liquidity.
- **Not audited.** See Risks.

---

## 2. Chain and external contracts

| Item | Value |
|---|---|
| Chain | Robinhood Chain, chain id **4663** (Arbitrum Orbit L2, sub-cent gas paid in ETH, ~0.1s blocks) |
| RPC (public, dev only) | `https://rpc.mainnet.chain.robinhood.com` |
| Explorer | `https://robinhoodchain.blockscout.com` |
| Testnet | LONG is not deployed on the Robinhood testnet. Integration is tested on a mainnet fork, then live with small amounts. |

### Known addresses

| Contract | Address | Status |
|---|---|---|
| SUIT token (`Suits`, 18 decimals, ~997.26M supply) | `0x87A78590D563F0EFC62192239af46707FFF74386` | Verified via RPC Sep 10, 2026 |
| SUIT / PLTR Uniswap v4 pool | TBD | Need pool key from MAXLONG. Confirm no hook. |
| The Suits NFT | TBD | Week one |
| LongLauncher | TBD | Week one |
| Doppler shared v4 hook (LONG pools) | `0x4e3468951D49f2EEa976eD0D6e75fFCb44a9a544` | Read from the AI / NVDA pool's PoolManager `Initialize` log, Sep 12, 2026. Dynamic-fee flag, tick spacing 8. |
| Pons v2 LaunchFactory | `0x7eD598BcEf8bd9Edd8C97A195C6d13f40801EC7e` | Verified Sep 10 (docs + Blockscout source) |
| Pons v2 MemeHook (graduated pools) | `0xE5e702641Ea86F4ae6cC3cDaeD2B886f976Be044` | Verified Sep 10. afterSwap only, no router gating. |
| Pons v2 LaunchLocker (holds every graduated LP NFT) | `0x267444D099b10fB5Ed7c3Cc7B7c767AdcA574952` | Verified Sep 10 |
| Pons v2 GraduationExecutor | `0xC7819B64A1dAECD7eC19856d026cb14EfBd89046` | Verified Sep 10 |
| Uniswap v4 PoolManager | `0x8366a39CC670B4001A1121B8F6A443A643e40951` | Uniswap deployments page, matches Pons factory read |
| Uniswap v4 PositionManager | `0x58daec3116aae6D93017bAAea7749052E8a04fA7` | Same |
| Uniswap v4 Universal Router | `0x8876789976decbfcbbbe364623c63652db8c0904` | Uniswap deployments page |
| Uniswap v4 Quoter | `0x8dc178efb8111bb0973dd9d722ebeff267c98f94` | Uniswap deployments page |
| Uniswap v4 StateView | `0xf3334192d15450cdd385c8b70e03f9a6bd9e673b` | Uniswap deployments page |
| Permit2 | `0x000000000022D473030F116dDEE9F6B43aC78BA3` | Canonical |
| USDG (6 decimals) | `0x5fc5360d0400a0fd4f2af552add042d716f1d168` | From Pons approved pair list |
| NVDA (18 decimals) | `0xd0601ce157db5bdc3162bbac2a2c8af5320d9eec` | From Pons approved pair list |
| PLTR (18 decimals) | `0x894e1ec2d74ffe5aef8dc8a9e84686accb964f2a` | From Pons approved pair list |
| TSLA (18 decimals) | `0x322f0929c4625ed5bad873c95208d54e1c003b2d` | From Pons approved pair list |
| AAPL (18 decimals) | `0xaf3d76f1834a1d425780943c99ea8a608f8a93f9` | From Pons approved pair list |
| Other stock tokens | 56 approved on Pons, all 18 decimals except USDG (6) and cbBTC (8) | Full list in `research/pons/` snapshot. Cross-check against LONG's list in week one. |
| Safe (fee wallet + factory admin, production) | TBD | Kobe's EOA during build, see Environments |

---

## 3. Product scope, v1

### Screens

**Wallet.** Normal web3 experience. Users connect MetaMask, Rabby, Coinbase Wallet, or a mobile wallet through WalletConnect (Reown). Every transaction shows the wallet's own confirmation. No embedded wallets, no email login in v1.

**Home feed.** Vertical feed, one card per token: image, ticker, pair (for example `SUIT / PLTR`), FDV in USD, 1h change, sparkline. LONG tokens, graduated Pons tokens, SuitVentures launches, and curated tokens (SUIT) share one feed. Each card carries a small origin badge (LONG, Pons, SV).
- Stock chips at the top: `ALL` plus one chip per allowed quote asset. A chip filters by quote asset. The `AI` chip shows tokens *quoted in* AI, not the AI token itself (which trades against NVDA).
- Sort: `New` (default) and `Trending`.
- Presale cards appear in the same feed with raised-vs-target bar, presale FDV vs launch FDV, time remaining, and a Contribute button showing the 1% success fee.

**Token page.** Chart (1h and 24h). Buy and Sell in the pair's quote asset. Slippage setting. Trades list. Holders list. Every quote shows amount out, the 0.5% SuitVentures fee, and the pool fee separately, before the user signs.

**Create.** One screen, four blocks. Opens only when the connected wallet holds at least one Suits NFT.
1. Image, name, ticker.
2. Quote asset, from the allowlist.
3. Community / LP / Team split.
4. Presale FDV, launch FDV, deadline (24h, 48h, 7d). Raise target is auto-calculated and shown.

### Token standard (immutable, enforced by the factory)

| Parameter | Value |
|---|---|
| Supply | 1,000,000,000 (1e9), 18 decimals |
| Tax | None |
| Pause | None |
| Additional mint | None |
| Community allocation | 10% to 40% |
| LP allocation | at least 50% |
| Team allocation | at most 15% |
| Vesting | None. Team tokens transfer to the creator in the launch transaction. |
| Allocation sum | Exactly 100%, fixed at creation |
| LP position | Locked in the presale contract forever. Not withdrawable by anyone. |

All supply is minted to the presale contract at deploy. No token can move before launch.

### Presale rules

| Rule | Value |
|---|---|
| Presale FDV | at most 70% of launch FDV |
| Raise target | community % of supply, valued at presale FDV |
| Raise token | the pair's quote asset. No separate choice. |
| Minimum raise | USD 2,000 equivalent |
| Maximum raise | USD 50,000 equivalent |
| Per-wallet cap | 10% of raise target. No minimum. |
| Deadline | 24h, 48h, or 7d from creation |
| Target missed | Presale closes. **Every contributor is refunded 100%.** No fee of any kind. |
| Target met | 1% of the raise to the fee wallet. Community share claimable pro-rata at presale FDV. LP share plus the remaining raise seeded into a new Uniswap v4 pool at launch FDV. Team share sent to the creator. |
| Partial launch | Never. Target or refund. |

**Units.** Every value the contracts store or check is in **quote-token units**: presale FDV, launch FDV, contribution caps, and the per-numeraire min and max raise. The contracts never see dollars. The Create screen converts what the creator types in USD into quote-token units at the current price and shows both. The USD 2,000 / 50,000 bounds are admin-set token amounts per numeraire, updated when prices drift materially.

**LP shape.** The launch pool receives the LP share of tokens against roughly (community % × presale ratio) of launch FDV in quote. Example: 30% community at 70% ratio gives about 21% of launch FDV in quote depth. The v4 position is therefore asymmetric by construction, with limited quote depth below spot. It is not a full-range position.

**Price gap.** Presale FDV is at least 30% below launch FDV and there is no vesting, so contributors can sell into the pool from the first block. This is the product as specified.

### Fees

LongMarkets charges two fees and no others.

| Fee | Rate | Paid in | Applies to |
|---|---|---|---|
| Swap fee | 0.5% | the quote asset | every swap routed through the LongMarkets app, on LONG, Pons, and SV pools alike |
| Launch fee | 1% of the raise | the quote asset | successful presales only |

- Pool-level fees set by LONG, Pons, or Uniswap are unchanged. Pons's hook charges 1% per swap (70% creator, 30% protocol). The quote line shows the pool fee and the SV fee separately, so a Pons trade reads as 1% plus 0.5%.
- The swap fee is taken on the quote side by the router. It applies only to trades made through this app. Trades on the same pool from another frontend pay nothing. A pool-level hook that enforces the fee everywhere is v2.
- The router never holds the token being bought. LONG tokens can carry an early per-wallet balance cap that would trip on an intermediary, so output goes straight to the user.
- Quotes go through the Uniswap V4 Quoter, because the Doppler hook can take a fee from the output amount.

**Fee allocation** (80% buyback of $SUIT and Suits NFT floor sweeps, 20% PLTR Community Vault, per the article) is **not automated and not in scope**. Both fees land in the SuitVentures fee wallet. MAXLONG executes and publishes allocation manually. No fee dashboard in v1.

### Quote assets

- Every pairing option LONG offers: the Robinhood Stock Tokens (AAPL, NVDA, TSLA, PLTR, and the rest) plus AI, ETH, and USDG.
- There is no on-chain list on LONG. LongLauncher forwards any numeraire address to Airlock unchecked. The stock dropdown exists only in LONG's frontend.
- The factory keeps an **admin allowlist**, seeded from LONG's frontend list. Adding one is a single transaction and is covered by the retainer. Chip labels come from each token's on-chain symbol.
- The same allowlist governs which Pons tokens are shown. Pons also pairs against cbBTC and other assets; those tokens are excluded unless the quote asset is added to the allowlist.
- A self-updating list derived from LONG launch events is v2.

### Access

- **Create:** the calling wallet must hold at least one Suits NFT. On-chain balance check in the factory. Image uploads also require a signed message from a Suits-holding wallet.
- **Buy, sell, contribute:** any wallet.

---

## 4. Architecture

### Contracts written for this project (Solidity, Foundry)

| Contract | Role |
|---|---|
| `SVToken` | Fixed-supply ERC20. All supply minted to its presale at deploy. |
| `SVFactory` | Create entry point. Checks Suits balance, numeraire allowlist, split, FDV ratio, raise bounds. Deploys token and presale. Admin: allowlist, per-numeraire bounds. |
| `SVPresale` | One per launch. Holds contributions, enforces caps and deadline, refunds in full on failure. On success: takes 1% fee, initializes the v4 pool at launch FDV, deposits and locks liquidity, opens community claims, sends team tokens. |
| `SVRouter` | Fee wrapper in front of the Uniswap Universal Router. Takes 0.5% on the quote side, forwards the swap. Same v4 path for LONG, graduated Pons, and SV tokens. No bonding-curve path. Uses Permit2 so a first-time buy is one signature plus one transaction. |

One presale contract per launch, so a bug in one presale cannot touch another. Every deployed contract is verified with source on Blockscout.

### Existing contracts used as-is

- Uniswap v4: PoolManager, PositionManager, Universal Router, V4 Quoter, Permit2.
- LONG, read only: LongLauncher launch events, Doppler hook swap events, pool state.
- Pons, read only: graduation events from the v2 launcher, Pons hook swap events on graduated v4 pools, pool state.
- Robinhood Stock Tokens, USDG, The Suits NFT, Safe.
- OpenZeppelin and Uniswap v4 libraries.

### Infrastructure

| Layer | Choice | Notes |
|---|---|---|
| Indexer | Envio HyperIndex on Envio Cloud (Production Small, likely Medium with Pons) | Indexes LONG launches, Pons `PoolRegistered`, PoolManager `Swap` for every tracked poolId (LONG, Pons, SV, SUIT), all SV factory / presale events, and token transfers for holders. Produces tokens, trades, 1h and 24h candles, holder balances, presale state. Pons is around 60% of launchpad volume on the chain, so stored events will exceed the Small plan sooner. |
| API layer | Next.js route handlers on Vercel | The app never calls Envio directly. Fixed queries, short-lived caching, per-IP rate limiting (Upstash). |
| Web app | Next.js, wagmi, viem, mobile-first, on Vercel | |
| Wallets | wagmi + WalletConnect (Reown) | External wallets only. |
| RPC | Alchemy | App reads, quotes, transaction sends. Not used for indexing. |
| Storage | IPFS via Filebase | Token images and metadata from Create. |
| Explorer | Blockscout | Source verification and address links. |

### Pons integration spec (verified Sep 10, 2026)

Source: Pons v2 docs, verified factory and hook source on Blockscout, and live reads over RPC. Copies of the ABIs, the two contracts, and a snapshot of every graduated pool are in `research/pons/`.

**Lifecycle.** A Pons v2 token launches onto a constant-product bonding curve (one curve contract per token, 1% curve fee, 1B supply). When the curve sells out, the factory calls `createGraduatedPool`, initializes a Uniswap v4 pool, mints a full-range position, and sends the LP NFT to the LaunchLocker where it is permanently locked. Graduation is permissionless to retry if it fails mid-transaction.

**Graduation events, in one transaction.**
- Factory `PoolGraduated(address indexed token, uint256 positionId, uint256 tokenAmount, uint256 pairTokenAmount)`.
- Hook `PoolRegistered(bytes32 indexed poolId, address memecoin, address quoteToken, address creator)`. This is the one to index: it gives the poolId, the token, and the quote asset directly.
- Factory `TokenLaunched(...)` fires at launch, not graduation. Not needed for graduated-only scope, and the full history is too large for a single log query.

**Pool key.** `currency0`/`currency1` sorted by address, `fee = 0`, `tickSpacing = 200`, `hooks = MemeHook`. Only one launch config exists (`launchConfigCount = 1`), so every graduated pool uses the same fee and tick spacing. The poolId in `PoolRegistered` is the keccak of this key.

**Hook behavior.** Permissions are `beforeInitialize`, `afterSwap`, and `afterSwapReturnDelta` only. `afterSwap` checks the pool is registered, then takes `hookFeeBps` (currently 100, so 1%) plus any creator tax (max 1000 bps, fixed at launch) from the unspecified side of the swap. For an exact-input buy that is the output. There is no `msg.sender` check, so the Universal Router and therefore SVRouter can trade these pools. Protocol share is 30% of the hook fee, the rest goes to the creator and buyback.

**Indexing plan.** Envio handlers:
1. `MemeHook.PoolRegistered` creates the token row (origin = Pons), the pool row, and adds the poolId to the tracked set. Filter on `quoteToken` in the allowlist.
2. `PoolManager.Swap(bytes32 indexed id, ...)` filtered to tracked poolIds gives trades, price, candles. This is the same handler LONG pools use, since Doppler pools also live in the same PoolManager.
3. `MemeHook.HookFeeCollected(bytes32 indexed poolId, address currency, uint256 feeAmount, uint256 taxAmount)` gives the exact fee per swap if we want to display net amounts historically.
4. Holder balances from the token's `Transfer` events, registered dynamically at graduation.

**Quoting and routing.** Because the hook takes its fee from the output after the core swap, quotes must come from the V4 Quoter, not from pool math. Same rule as Doppler. SVRouter wraps the Universal Router with a v4 exact-input path and takes 0.5% on the quote side before forwarding.

**Volume snapshot, Sep 10, 2026.**

| Quote asset | Graduated pools |
|---|---|
| ETH (native) | 2,770 |
| USDG | 1,247 |
| Stock and ETF tokens (NVDA 341, SPY 231, GME 184, DJT 157, RDDT 116, QQQ 103, AAPL 80, TSLA 75, PLTR 42, and 40 others) | about 2,400 |
| cbBTC, TAO, WYFI, SKHY, BULL | 73 |
| Total | 6,490 |

Curve-phase tokens are a separate contract per token with `CurveBuy` / `CurveSell` / `CurveCompleted` events and no quote function. Out of scope for v1 as decided in D18.

### USD price source

Cards show FDV in USD, so each quote asset needs a USD price. USDG is one dollar. For stock tokens, ETH, and AI: on-chain pools against USDG if they exist on Robinhood Chain (indexer-derived), otherwise an external price API with a short cache. Resolved in week one. On-chain first, API fallback.

---

## 5. Build order and milestones

Build order follows the article: feed and swap ship before any presale contract accepts funds. Dates assume kickoff Sep 10, 2026 and shift with it. Payment on acceptance of each milestone.

| # | Milestone | Days | Target | Payment | Definition of done |
|---|---|---|---|---|---|
| 0 | Signing and kickoff | 1 | Sep 10 | USD 1,000 | Kickoff call, decisions confirmed, shared channel. |
| M1 | Feed and token page | 2 to 6 | Sep 15 | USD 600 | Loads on a phone, shows LONG tokens and graduated Pons tokens from live chain data, with origin badges. Cards, chips, sort. Token page with 1h/24h chart, trades, holders. No wallet yet. |
| M2 | Wallet connection and swap | 6 to 9 | Sep 18 | USD 600 | Connected wallet buys and sells a LONG token on a live pair (test pair AI / NVDA) and a graduated Pons token on a stock pair. Quote shows amount out, 0.5% SV fee, pool fee. 0.5% arrives in fee wallet. SVRouter verified on Blockscout. |
| M3 | Presale contracts | 9 to 16 | Sep 25 | USD 600 | SVToken, SVFactory, SVPresale deployed and verified. Foundry suite passes: bad splits, bad FDV ratios, non-Suits wallets rejected; per-wallet caps enforced; missed deadline refunds every contributor in full; met target runs the launch path end to end on a mainnet fork against real Uniswap v4. |
| M4 | Create, presale UI, Suits gate | 15 to 20 | Sep 29 | USD 600 | Suits holder completes Create, presale card appears. Non-holder is blocked. Any wallet contributes, sees the 1% fee, bar updates on confirmation. |
| M5 | Launch | 19 to 24 | Oct 3 | USD 300 | Test presale reaches target and launches: 1% in fee wallet, v4 pool at launch FDV, liquidity locked, community claims work, team tokens delivered. Token appears live in feed and trades at launch FDV. |
| M6 | Hardening, UI pass, handover | 24 to 30 | Oct 9 | USD 300 | Mobile layout reviewed. Error and empty states. One real launch with small amounts. Repo access, contract ownership transferred, address document with Blockscout links. |
| | **Total** | | | **USD 4,000** | |

Retainer after M6: USD 300 per month. Covers indexer and RPC monitoring, uptime, bug fixes, adding numeraires to the allowlist, up to 5 hours per month.

---

## 6. Environments

| | Build phase (now) | Production (at handover) |
|---|---|---|
| Fee wallet | Kobe's EOA, address TBD | MAXLONG's Safe or wallet |
| Factory admin | Kobe's EOA | MAXLONG's Safe or wallet, ownership transferred at M6 |
| Service accounts (Envio, Alchemy, Vercel, Filebase, Reown, Upstash, domain) | Kobe's accounts | Moved to MAXLONG's email when he is ready |
| Test funds | ~USD 100 in ETH and stock tokens, most recovered | |

---

## 7. Decision log

Resolved decisions, newest last. Source key: **P** = proposal, **A** = article, **C** = client chat (Sep 10, 2026), **K** = Kobe.

| # | Decision | Resolution | Source |
|---|---|---|---|
| D1 | Quote assets | All pairing options LONG offers: stock tokens plus AI, ETH, USDG. Admin allowlist in the factory mirrors LONG. | C |
| D2 | Failed presale | 100% refund, no fee of any kind. Fee only on success. Client initially answered "take 1% even if failed"; reversed to match the article and industry practice (PinkSale, DxSale, PartyDAO, Virtuals all refund in full and earn only on success). | A, C, K |
| D3 | Per-wallet caps | Max 10% of raise target per wallet, no minimum. A limit, not a fairness guarantee. | C |
| D4 | LP after launch | Locked in the presale contract. Nobody can withdraw. | C |
| D5 | Fee enforcement | Router-only, applies to trades through this app. Pool-level hook is v2. | A, C |
| D6 | Wallet experience | Normal web3 (Option B). Embedded wallets / one-tap (Privy) deferred until product-market fit. M2 is day 9. | C |
| D7 | Fee wallet and admin | Kobe's EOA during build, swapped to MAXLONG's address at handover. | C |
| D8 | Service accounts | Kobe's accounts during build, moved to MAXLONG's email later. | C |
| D9 | Fee allocation (buyback / vault) | Not automated, not in scope. Manual by MAXLONG. No dashboard. | C |
| D10 | Product name | LongMarkets, by SuitVentures. Name-collision risk with LONG raised; client chose to proceed. | C |
| D11 | SUIT token | Exists at `0x87A7…4386`. Not a factory launch and does not follow the token standard (60/30/10). Indexed independently as a curated pool. | C, K |
| D12 | On-chain units | All contract values in quote-token units. USD only in the UI. Min/max raise are per-numeraire admin constants. | K |
| D13 | Launch LP shape | Asymmetric v4 position by construction. Documented, not full range. | K |
| D14 | Approvals | Permit2 signature-based approvals in the router. | K |
| D15 | USD price source | On-chain USDG pools first, external API fallback. | K |
| D16 | AI chip semantics | Filters tokens quoted in AI. The AI token itself lives under NVDA. | K |
| D17 | LONG visibility | Stay on the SuitVentures factory. SV launches will not appear on LONG or Pons. Client accepted this to keep presales and the token standard. | C |
| D18 | Pons aggregation | Index and trade Pons v2 tokens that have graduated to Uniswap v4, quoted in allowlisted assets only. Curve-phase tokens and Pons v1 (v3 pool) tokens are out of scope. Feed and swap share the LONG path. | C, K |
| D19 | Origin badge | Each card shows LONG, Pons, or SV so users know which launcher the token came from. | K |
| D20 | Pons integration verified | Hook is afterSwap-only with no router gating, so SVRouter can trade graduated pools. Pool key is fee 0, tick spacing 200, MemeHook. Index on `PoolRegistered` plus PoolManager `Swap`. Addresses recorded in section 2. | K |
| D21 | Pool discovery | On PoolManager `Initialize` with a quote-asset filter: exactly one side of the pool is on the quote allowlist, the other is not. The only discovery path; launchpad events are not consumed. The hook address only labels origin (Doppler → LONG, Pons MemeHook → Pons, no hook → Uniswap, anything else → Other); a curated poolId list overrides the label for SUIT. | K |
| D22 | M1 quote allowlist | Robinhood stock tokens plus USDG. No ETH or WETH yet; the filter reads only `Quote` rows, so adding one later is a config row plus a backfill. | C, K |
| D23 | Token identity | Tokens are keyed by contract address, not ticker (thousands of memecoins share symbols). Routes and the data contract use the address. A token can have several pools; the table row uses `primaryPool`, chosen by USD TVL, then swap count, then earliest pool. | K |
| D24 | USD price source | Chainlink tokenized-equity feeds on Robinhood Chain, ingested as `AnswerUpdated` events into a `QuotePrice` entity; USDG pinned at 1. Proxies are indexed so aggregator rotations register the new aggregator automatically; a quote older than 3 days is flagged stale. On-chain stock/USDG pools were rejected: the deep ones are Uniswap v3, outside the v4 handler. | K |
| D25 | Storage units | Indexer stores prices, volumes and TVL in quote units; USD conversion happens only in the API route handlers (extends D12). | K |
| D26 | Candles | Buckets at 1m, 5m, 15m, 1h, 1d, created only on trades; a bucket after a gap opens at the previous close. 1m candles from about 7 days before first deployment, 5m from 30 days, longer intervals over the full range. No pruning in M1; rolling deletes are M6. | K |
| D27 | Liquidity (TVL) | Token-flow accounting from Swap, ModifyLiquidity and Donate, exact for full-range and multi-position pools. Hook fee deltas are invisible to the Swap event, so drift is accepted for v1; a StateView sanity check is M6. Named M1 schedule risk: if it slips, liquidity ships blank and lands in M3. | K |
| D28 | Tradeability | Never derived from origin. A V4 Quoter simulation at the API edge, cached per pool, decides whether the Buy/Sell panel is live; lands with the router in M2. | K |
| D29 | Indexer hosting | Envio Cloud free Development plan for the M1 demo (10-day life after its first sync once Transfer indexing passes the 100k-event soft limit), Production Small from the M2 demo. Alchemy free tier. | C, K |
| D30 | Holders and supply | Holder balances and count from ERC-20 Transfer events, registered at discovery; Envio indexes a dynamic contract from its creation block, so history is complete. Total supply is the running sum of mints minus burns, never read over RPC. | K |

---

## 8. Open items

Nothing here blocks starting M1. Each has an owner and a deadline.

| Item | Owner | Needed by |
|---|---|---|
| Pons aggregation was added Sep 10 after the proposal was agreed. Decide whether it is absorbed into the fixed price or billed as a change order at the day rate, and record it here. | Kobe / MAXLONG | Day 2 |
| SUIT / PLTR pool key, confirm no hook | MAXLONG | M1 |
| Kobe's EOA address recorded in Environments | Kobe | Day 2 |
| Verify Robinhood Stock Token contracts carry no transfer allowlist or blocklist (router and presale hold them) | Kobe | M2 |
| Confirm Safe is deployed on chain 4663, else propose fallback | Kobe | Day 2 |
| Confirm USDG pools exist for each quote asset (price source) | Kobe | M1 |
| Collect and record all TBD addresses in section 2 | Kobe | M1 |
| Article wording: "SuitVentures indexes LONG" is now LONG and Pons; "immediately visible on both LONG and SuitVentures" is false and should be cut; "one-tap swap" is tap plus wallet confirm under Option B; "stock tokens are the only quote assets" is now stocks plus AI, ETH, USDG. The name LongMarkets also reads oddly for a product that aggregates Pons. | MAXLONG | Before public launch |

---

## 9. Out of scope for v1

From the brief: vesting, LongLauncher as minter, LongX, native mobile app, gasless, Apple Pay, multi-chain, Community Mode hooks, docs page, terminal UI, vesting UI.

Added: third-party security audit, automated fee allocation (buyback, NFT sweeps, PLTR vault), fee dashboard, embedded wallets (Privy), pool-level fee hook, self-updating numeraire list, Pons bonding-curve (pre-graduation) trading, Pons v1 tokens in Uniswap v3 pools, Pons tokens quoted in non-allowlisted assets, launching through LongLauncher or Pons for visibility on their dapps.

Separate engagements, quoted separately: The Suits token deployment, airdrop claim page (30% of SUIT to Suits NFT holders).

Work outside this scope: USD 350 per day, agreed in writing first.

---

## 10. Risks

- **No audit.** Presale contracts hold up to USD 50,000 per launch. Fork tests against live Uniswap v4 are the mitigation. Shipping unaudited is the client's recorded decision.
- **No testnet for LONG.** Mainnet fork, then live with small amounts.
- **LONG or Pons changes.** If either platform changes its hook, launcher, or token behavior in a way that blocks routing or indexing, the schedule pauses until a path is agreed.
- **Pons hook parameters are owner-settable.** `hookFeeBps` and the creator tax cap can change by owner transaction. The quote line reads the pool fee live rather than hardcoding 1%.
- **Fee leakage.** Router-only fee. Other frontends trade SV pools for free.
- **Presale price gap.** Contributors buy at a discount of 30% or more with no vesting and can sell from block one.
- **Thin pools.** A small swap can move price materially.
- **Stock token characteristics.** Issuance, redemption, and market hours affect quote-side pricing independently of the community token.
- **Non-custodial.** LongMarkets cannot pause tokens it launches and cannot reverse transactions.

Nothing in this file is investment advice.

---

## 11. Working agreements

- Written progress update Monday and Thursday: shipped, next, waiting on.
- Each milestone ends with a short demo. Acceptance or specific change requests within 48 hours; silence counts as acceptance.
- Replies to open questions within 24 hours keep the schedule; the 30-day clock pauses for longer waits and for LONG-side blockers.
- Kobe deploys and holds admin during the build, then transfers contract ownership and repository access at handover. General tooling and non-client-specific code may be reused.

## Repositories

This folder is the workspace repo (`KobeSergio/suitventures-markets`): it holds this README (the Decision Log is the source of truth), `research/`, and the app folders as git submodules. Each app keeps its own repo and its own CI/CD (Vercel for the frontend, Envio Cloud for the indexer).

| Folder | Repo |
|---|---|
| `SuitVenturesMarket-frontend/` | https://github.com/KobeSergio/suitventures-markets-frontend |
| `SuitVenturesMarket-indexer/` | https://github.com/KobeSergio/suitventures-markets-indexer |

Clone with `git clone --recurse-submodules`. After pushing new commits in an app repo, run `git add <folder> && git commit` here to move the submodule pointer.
