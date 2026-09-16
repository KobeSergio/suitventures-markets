# LongMarkets Indexing Decision

Decision memo for MAXLONG. Prepared by SuitVentures (Kobe), 16 September 2026. The proposal stays on Production Small (USD 70 per month); any upgrade waits until the product has found its market.

## The problem

The agreed scope lists every Uniswap v4 pool that pairs a token against a stock token or USDG, from any source, and tracks holders for every listed token. The indexer does exactly that, and it is expensive on Envio Cloud for two reasons:

- Envio bills by events processed and by tracked contracts. Every swap and liquidity change on the chain's single PoolManager is delivered, including for pools we don't list, and every token tracked for holders counts as a contract.
- The chain is busier and spammier than the snapshot. The full sync lists 234,945 pools; 165,331 have no launchpad hook, many of them spam (WETH alone has 962 pools, and 136,943 pools have never traded). About 4,000 to 6,000 new pools appear per day chain-wide.

Production Small (about 1 M events, 1,000 contracts, USD 70/month) was exceeded within the first hour. Envio is warning, not cutting off, and nothing beyond USD 70 has been charged. The full sync ended on 16 September at 859 million events and 130,226 registered token contracts, and the chain adds about 25 million events a day at the current subscriptions.

## What we measured

| Measure | Value | Source, date |
|---|---|---|
| Chain size | 64.29 M blocks, about one per 0.1 s | Envio, 16 Sep |
| Full sync of the current build, events processed | 859.4 M in 38 h for the whole chain; about 295 events per second at the chain head, 25 M a day | Envio metrics and endpoint, 16 Sep 11:43 and 14:25 |
| Token contracts registered | 130,226 by the end of the sync (68,265 plain Uniswap, 45,547 LONG, 3,825 Pons, 12,589 other) | indexer database, 16 Sep |
| Pools listed, by origin | 234,945: 165,331 plain Uniswap, 47,122 LONG, 4,513 Pons, 17,979 other hooks; 136,943 never traded; 8,653 traded in the last 24 hours | indexer database at 100%, 16 Sep |
| Launchpad pools | 51,635 (47,122 LONG, 4,513 Pons); 31,001 never traded; 3,857 traded in the last 24 hours, 9,385 in the last 7 days | indexer database at 100%, 16 Sep |
| Concentration of launchpad swaps, lifetime | top 50 pools 20.2%, top 100 29.3%, top 300 47.1%, top 1,000 69.1% of 30.4 M lifetime swaps | indexer database at 100%, 16 Sep |
| Launchpad swaps on 15 September, full day | 694,496 in 3,793 pools (USD 60 M); the 100 busiest pools 461,112, the 150 busiest 533,834, the 300 busiest 631,048. All quote-paired pools: 1,106,322 swaps in 8,370 pools | indexer database, daily candles, 16 Sep |
| Swaps in the Pons pools, 16 Aug | 7,680 in the full day | HyperSync sample |
| Swaps in the Pons pools, 29 Aug | 138,396 in the first 8 h (about 17,000 per hour) | HyperSync sample, 33% of the day |
| Swaps in the Pons pools, 6 Sep | 212,258 in the first 3 to 4 h; 1,426 of 3,664 pools active | HyperSync sample, about 2% of the 7-day window |
| Busiest launchpad pools, lifetime swaps | GME on LONG 427,021; MEME on LONG 401,533; AI on LONG 371,177 | indexer database at 100%, 16 Sep |
| New pools created chain-wide, 16 Aug | 5,762 in the day | HyperSync sample |
| New pools created chain-wide, 24 hours to 16 Sep | 4,042 (2,397 plain Uniswap, 1,243 LONG, 340 other, 62 Pons) | indexer database |
| Events per block, by region | 2.8 up to block 5 M; 5.7 between blocks 7.5 M and 13 M; about 12 between 13 M and 51 M; about 30 at the chain head | deployment metrics and endpoint, 14 to 16 Sep |
| Total for the current build | 859.4 M events, growing by about 25 M a day; 52.5 M swaps and 17.7 M holder rows stored | sync completed 16 Sep 11:43 |

The trend is steep: Pons pools alone went from 7,680 swaps a day in mid-August to roughly 400,000 a day on 29 August and over a million a day on 6 September. Even limited to launchpad pools, a 30-day window is tens of millions of events per month at September's pace.

## Envio Cloud plans

| Plan | Price | Events (approx.) | Contracts | Notes |
|---|---|---|---|---|
| Development | free | 0.1 M | 100 | no custom environment variables; auto-deletes over the limit; unusable here |
| Production Small | USD 70/month | 1 M | 1,000 | current plan; 800 hours included, USD 0.10 per extra hour, 250 queries/minute |
| Production Medium | USD 300/month | 10 M | 10,000 | effect cache survives redeploys; USD 0.20 per extra hour, 1,000 queries/minute |
| Production Large | USD 800/month | 100 M | 50,000 | USD 0.50 per extra hour, 2,000 queries/minute |
| Dedicated | custom | unlimited | unlimited | quoted by Envio |

The pricing page counts events and contracts per deployment, calls its figures indicative, and bills only extra indexing hours beyond the base price. Envio documents what happens over the limit only for the free plan; for paid plans nothing is documented, and after a full sync at 860 times the figure we have seen a warning and an upgrade prompt, nothing else. A question is with Envio support.

## The plan, and the upgrade path

The plan and its later upgrade use the same redesigned indexer: only tokens launched on Pons or LONG that have left their curve or auction; a pool-id filter applied at the data source so unlisted pools are never fetched or billed; a live window with older charts built on demand; liquidity read from pool state; and a daily automatic redeploy that picks up new launches. They differ only in how much is live and on which plan; the upgrade is a configuration change plus the plan, taken after product-market fit.

| | What it is | What the app shows | Changes from the agreed scope | Risks | Cost |
|---|---|---|---|---|---|
| The plan: launchpad tokens, live set of about 100 pools, Production Small (proposed) | Every Pons and LONG token past its curve is listed; live trading data for the most active launchpad pools (about 100, roughly two thirds of a day's launchpad swaps) over a 24-hour window; holders as a snapshot per token; all other tokens show hourly volume and trades and a daily price and liquidity; a daily redeploy keeps each deployment's event count at about one day's worth | The markets table with USD price, 24h change and volume, FDV, liquidity, holder count, age and origin for every launchpad token; live charts, trades and top traders for the active pools; 7-day figures and older charts from the archive | Plain Uniswap pools leave the listing; live trading data covers the active pools only; 7-day figures come from the archive; holder counts are dated snapshots; candle windows and liquidity source change | Small's figures are approximate and its 1,000-contract cap is met only by not tracking transfers live; on 15 September the launchpad pools traded 694,000 times and the 100 busiest of them 461,000 times, so a day of the live set sits inside the 1 M figure with room for a day twice as busy; the 300 busiest (631,000) would leave little room; the live-set size and window are tuned from measured daily counts after the first filtered sync | USD 70/month (current plan); 3 to 5 dev days for the indexer plus about half a day for the switches; the upgrade later is a configuration change plus the plan |
| Upgrade path, after product-market fit: every active launchpad token live, 30-day window, Production Medium (later) | The same engine with the live set widened to every launchpad token that traded in the window (about 3,800 a day, 9,400 in a week), holders tracked live for them and snapshots for dormant tokens (over 49,000 launchpad tokens exist against Medium's 10,000 contracts); the last 30 days indexed live; 15-minute and longer candles for older periods from the archive | Every agreed feature for launchpad tokens: USD price, 1h and 24h change, 24h volume, FDV, liquidity, holders, age, origin, charts at all intervals, last 50 trades, top holders, top traders, watchlist and votes | Plain Uniswap pools stay out; candle windows and liquidity source stay changed; holders return to live for active tokens | At September's pace 30 days of launchpad swaps plus transfers can exceed Medium's approximate 10 M events; if the first sync at that scope shows that, the live window is trimmed to 7 days rather than moving to Large | USD 300/month when taken; configuration change plus the plan; tier confirmed on the first sync at that scope |

## What changes from the initial plan

The plan keeps every agreed feature on screen but changes how several are produced. After the upgrade to Medium, the live set becomes every recently traded launchpad token, so the rows with a live-set limit relax.

| Area | Initial plan, as agreed | Proposed for the USD 70 plan | Trade-off |
|---|---|---|---|
| Which tokens are listed | Every Uniswap v4 pool pairing a token with a stock token or USDG, from any source | Only tokens launched on Pons or LONG that have left their curve or auction, about 49,000 today | Plain Uniswap pools disappear: the spam pools and majors such as USDe, but also any memestock launched outside a launchpad |
| Live trading data (price ticks, trades, minute candles) | Every listed pool, live | The roughly 100 most active launchpad pools live; all others updated hourly (volume, trades) and nightly (price, liquidity) | A breakout token outside the live set goes live about 10 minutes after the hourly scan flags it, through an early rollover |
| History and charts | Full history in the indexer; 1m candles 7 days, 5m 30 days, longer intervals over the full range | Envio holds 24 hours; a Supabase archive holds the full range, seeded from the current sync; 1m and 5m candles for the live set, hourly and daily for every token | Minute-level charts only for the live set; other tokens chart at hourly resolution |
| Liquidity | Computed from every liquidity event | Read from pool state at seed time and nightly; live for the live set | Up to a day old for non-live tokens; the same figure DexScreener shows |
| Holders and supply | Live from transfer events for every token | Seeded for every token from the full sync; recomputed daily for the live set, nightly for tokens that traded, on demand for dormant ones | Counts are daily and dated rather than live; the six largest tokens need a raised cap or live tracking |
| USD prices | Chainlink feeds through the indexer | The same, with the last answer seeded at each rollover | None |
| Top traders board | From indexed swaps across all pools | From the hourly scan across all launchpad pools | Up to an hour behind |
| Freshness | Seconds behind the chain | 20 to 30 s for the live set; hourly or daily for the rest; swap quotes stay real-time on-chain | Near-real-time instead of real-time for market data |
| Where the app reads | Envio's GraphQL through the API | Supabase Postgres, filled by a 15-second sync job; the browser never touches Envio | One more system; the decision that Supabase holds only user data changes; Supabase Pro at USD 25/month |
| Hosting and cost | Envio Small, USD 70/month | Envio Small USD 70; Supabase USD 0 rising to 25; HyperSync USD 0 rising to 70 if the hourly scan outgrows fair use | USD 70 to 165/month versus USD 300 on Medium |
| Operations | Deploy once; redeploy on code changes | Automated daily rollover (regenerate list, deploy, promote, delete), early rollovers on breakouts, hourly and nightly jobs | More moving parts; a failed rollover keeps the old deployment serving but needs attention |
| Engineering | Indexer complete (M1) | 3 to 5 days indexer redesign, 2 to 3 days Supabase and sync, 1 to 2 days API changes, during M2 | Pressure on the M2 and M3 dates |
| Upgrade path | | Moving to Medium later is a config change (the live set becomes every active token) plus the plan | None; the work is not thrown away |
| Risks | | Envio does not document whether a redeploy resets its event figure; a busy day can still exceed it; six decision-log entries change: pool discovery, product scope, candle windows, liquidity, holders, and what Supabase stores | The plan depends on Envio's answer to the support question |

## Considered and not proposed

| Option | Why not | Cost |
|---|---|---|
| Keep the current build and upgrade the plan (ruled out) | Measured at 859 M events for the full chain, more than eight times Large's 100 M (Dedicated pricing), growing by 25 M a day; about 38 hours of re-sync per code change; pays to index spam | USD 800/month or more |
| Every quote-paired pool on the chain, redesigned | Keeps the spam pools and the majors (USDe, VIRTUAL) in the listing and the bill; Large at a 30-day window, Medium only at 7 days | USD 300 to 800/month |
| Self-host the indexer | Replaces a managed service with a server run under the retainer: no managed uptime, backups or monitoring, one machine, no failover; also needs its own HyperSync plan (Free with fair-use limits, Starter USD 70 at 100 rpm, Pro USD 480 at 1,000 rpm; the first sync ran at about 300 rpm) while Cloud deployments are exempt | USD 100 to 540/month plus operations |

## Recommendation

Stay on Production Small and build the plan above. It lists every launchpad token, keeps every agreed feature on screen, removes the spam pools and the majors by definition, and holds the hosting bill at the USD 70 already being paid, plus Supabase. Live, minute-level trading data covers the roughly 100 most active tokens, with everything else refreshed hourly and daily; that is the trade the plan makes to fit.

Upgrade only after the product has found its market. The upgrade to Medium is a configuration change plus the plan and widens live data to every active launchpad token; it is described above so the path is known now and decided later, on real usage.

Meanwhile the M1 demo is covered: the current deployment finished its backfill on 16 September, shows every M1 acceptance item, and is deleted once the replacement is live.

## What we need from MAXLONG

- Approval of the plan, ideally by 16 September so the indexer work lands during M2 without moving M3.
- Approval to narrow the listing to launchpad tokens, and to the changes in how candle history is kept (a live window, older intervals on demand), how liquidity is measured (from pool state), and holder counts as dated snapshots until the upgrade.
- Who holds and pays the hosting account. The Envio subscription is on SuitVentures' card (USD 70, paid 14 September).

Figures from Envio's dashboard and metrics, the indexer's database, and read-only HyperSync samples on 14 to 16 September 2026. Engineering estimates are SuitVentures' own. Prices in USD from Envio's public pricing pages.
