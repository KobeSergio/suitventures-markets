# Indexer capacity: what we measured, what Envio said, what we can do

21 Sep 2026. Follows `indexing-decision-2026-09-14.md`, where we chose to run the full indexer on Envio's
Production Small plan as a live limit test. The test is over: Envio paused the deployment on 21 Sep.
All times below are UTC (Manila is eight hours ahead).

## Verdict

Envio's hosted plans cannot hold this workload at our budget, whatever events we drop. The one event we
cannot drop (Swap) is by itself one to two and a half times the Small plan's entire intended size every day.
The site stays up on the paused endpoint's frozen data (about 20 Sep 10:00) until a replacement feeds the
store. Two replacements are real: self-hosting Envio's open-source indexer on a small server, or reading the
PoolManager's events ourselves inside the sync that already runs on Vercel, straight into Supabase.

## What Envio said (Discord ticket, 21 Sep, support engineer Denham)

> Yes, this indexer is far too big for the production small resources. This is currently close to 1 billion
> events, but the prod small plan is intended for 1 million event.

Asked whether cutting to a million events would let it sync: "It's a start, I suggest running it locally as
well and debugging there, try setting a more recent start block. Let me know if you want to explore a
dedicated instance setup, but what you have now will not be able to run on the amount of resources on prod
small." They paused the deployment the same day. The paused endpoint still answers queries with frozen data
(last processed block 67,841,653).

## How it failed

| When | Processed block | Behind the chain |
|---|---|---|
| 18 Sep 17:05 | 66,365,951 | 61,157 blocks |
| 19 Sep 16:03 | 67,050,340 | 145,834 |
| 21 Sep 03:33 | 67,740,779 | 737,605 (19 hours) |
| 21 Sep 04:41 | 67,782,699 | 736,025, flat for a full minute |

It was never slow. Sampled every 15 s for three minutes against the chain's real head, it moved once,
4,756 blocks in one tick (about 317 blocks a second), and sat idle for the other eleven ticks with a backlog
of 737,605 blocks it already knew about. The chain makes about 10.6 blocks a second. Logs showed
`Height subscription for HyperSync source went down … other side closed` every five minutes from
18 Sep 18:28. Lesson: `chain_metadata.block_height` on the endpoint is itself lumpy and stale, so measure lag
against the node's `eth_blockNumber`, never against Envio alone.

## What the indexer takes in, per day

Sampled in twelve windows across the 24 hours to 21 Sep 08:00.

| Event | Per day | Share | What it powers | If removed |
|---|---|---|---|---|
| Transfer, from our 132,004 registered tokens | ~6.26 M | 68 % | Holders column, top holders tab, the supply behind market cap and FDV | Replaceable: stored counts, `totalSupply()`, trades |
| Swap, chain-wide from the PoolManager | ~2.49 M | 27 % | Price, 1H and 24H, volume, candles, trades list, activity sort | No product |
| ModifyLiquidity | ~0.47 M | 5 % | Liquidity column | Replaceable by reading pool state |
| Initialize | ~8,700 | 0.1 % | New listings, launchpad badge, age | No new listings; negligible, keep |
| Chainlink AnswerUpdated | ~220 | ~0 % | Every dollar figure | Negligible, keep |

Total about 9.2 M that day; 16 Sep measured about 25 M. Over its life the deployment processed 859 M events
by 16 Sep and 891.8 M by 18 Sep, 93 % of them transfers.

Trading is concentrated but not enough to matter. On the newest stored day (915,843 swaps, 7,211 pools):

| Pools kept | Share of swaps | Swaps a day | Passes 1 M events after |
|---|---|---|---|
| Top 50 | 44.3 % | 406 k | 2.5 days |
| Top 100 | 57.6 % | 527 k | 2 days |
| Top 300 | 79.4 % | 727 k | 33 hours |
| Top 500 (what the table lists) | 87.2 % | 798 k | 30 hours |
| All | 100 % | 0.9 to 2.5 M | 10 to 26 hours |

A pool filter is also fixed at deploy time, so a token launched this morning would show no price until the
next redeploy. Envio 3.10 does support source-side filters, in the handler API (`eventFilters` /
`where: { params }`), not in the YAML config.

## Ways to keep Envio

1. A bigger hosted plan or a dedicated instance. Medium was USD 300 a month on 15 Sep; "dedicated" was the
   only thing support offered for this size. Ask them for the smallest plan that holds about 3 M events a day.
2. Self-host the open-source indexer in Docker on a small server. Keeps all handler code and tests, and the
   store's sync runs unchanged against our own endpoint. Costs a server (USD 10 to 20 a month), about a day
   of setup, a machine to keep alive and hand over, and a second database with no cleanup that grows about
   1 GB a day without transfers.
3. Redeploy on Small every day to reset the per-deployment counter. Rejected.

## Reading the chain ourselves: proven feasible

One minute of chain (600 blocks), 21 Sep 07:30:

| Source | Result |
|---|---|
| Public RPC `rpc.mainnet.chain.robinhood.com`, `eth_getLogs` filtered to the PoolManager | 1,769 logs, 1.66 MB, 385 ms; logs carry `blockTimestamp`; 835 distinct transactions; a JSON-RPC batch of 100 `eth_getTransactionByHash` returns the senders in 429 ms; `eth_getBlockReceipts` is supported |
| HyperSync data API `4663.hypersync.xyz/query` with our API token (client already written in the indexer repo, `src/lib/hypersync.ts`) | The same 1,769 logs plus 835 senders plus 431 block timestamps in one request, 2.4 s |

The public RPC answers HTTP 429 to a back-to-back burst of about 26 calls and answers normally at one call
every 1.2 s. A reader needs about twelve calls a minute. The indexer's whole brain is 718 lines of
TypeScript with tests: `handlers/poolManager.ts` 248, `lib/liquidity.ts` 161, `lib/quotes.ts` 72,
`handlers/shared.ts` 71, `lib/candles.ts` 50, `lib/origin.ts` 36, `lib/primaryPool.ts` 36, `lib/price.ts` 27,
`lib/denylist.ts` 17.

## What is already in our store (21 Sep)

132,004 tokens, of which 131,902 carry a holder count and 131,884 a total supply; 238,391 pools;
16,647,299 trades in a ten-day window ending 18 Sep 16:20. The database is 8.64 GB on a 12 GB disk and has not
grown since the copy was paused on 18 Sep 18:25; nothing is writing to it (the sync was never switched on).

| Table | Size | Note |
|---|---|---|
| swaps | 7.5 GB | 287 bytes a row, of which 220 are four hex strings stored as text |
| candles | 0.96 GB | a partial copy of 2.85 M rows; the full set is 20 to 25 M rows, about 7 to 8 GB |
| pools + tokens | 161 MB | 2 % of the database, so deleting inactive tokens saves nothing |

Candles have no retention anywhere: once fed, the store would gain about 870 k rows (290 MB) a day.

## Keep only what the screen can show

`src/lib/market/dbBackend.ts` caps every read: the newest 300 candles per interval, the newest 50 trades per
pool, and the traders board reads 24 hours. Nothing reads anything older.

| Data | Most the site can show | Keep |
|---|---|---|
| 1-minute candles | 5 hours | 1 day |
| 5-minute candles | 25 hours | 2 days |
| 15-minute candles | 3 days | 4 days |
| Hourly candles | 12.5 days | 14 days |
| Daily candles | 300 days | 300 days |
| Trades | 50 per pool, 24 hours for the board | 3 days |

Trades by age in the store: 1 day 0.41 GB, 2 days 0.91, 3 days 1.40, 5 days 2.60, 10 days 7.27. With these
windows the whole database settles near 2.5 to 3 GB and stays flat, inside the 8 GB the Pro plan includes.
Freed space is reused, not returned, so the disk stays provisioned at 12 GB.

## Direction and what is time-critical

Direction under design: read the PoolManager's events inside the existing Vercel sync, apply the indexer's
own price, candle and liquidity math (ported with its tests), write Supabase with the retention above, take
dollar prices from the Chainlink contracts and supply from each token contract, move holder counts with
trades from the stored baseline, then cancel hosted indexing (USD 70 a month).

Time-critical: the paused endpoint is a consistent snapshot of every pool's cumulative state at block
67,841,653, and the only copy of the hourly and daily candle history. Copy pools (with their square-root
price), tokens and those two candle histories before Envio deletes it. The copy needs the candle retention
filter first, or it would pull all 22 M candles and overrun the disk.
