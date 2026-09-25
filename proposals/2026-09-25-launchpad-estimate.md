# Token launchpad: scope and running costs

**For MAXLONG. Prepared by SuitVentures, 25 September 2026.**

## The short version

LongMarkets can add a launchpad: a creator raises in an allowlisted stock token or USDG, and if the
raise succeeds the token goes live in a Uniswap v4 pool in the same transaction, with the liquidity
locked forever. You already settled the rules for this in early September, before the launch layer was
set aside: a failed raise refunds every contributor in full with no fee of any kind, one wallet may
take at most 10% of the raise target, the liquidity position is locked and nobody can withdraw it, and
the fee is earned only on success.

SuitVentures is building this at no charge. What MAXLONG carries is the running cost of the site,
which the launchpad does not change, plus one decision about an audit.

## What it adds

Today LongMarkets lists tokens other people launched. A launchpad makes it the place tokens begin.
The creator gets a raise, a pool and a live market page in one flow. You get a launch fee on every
successful raise, on top of the trading fee the site already earns on every swap. The Suits NFT gets
its original job back: it can gate who is allowed to launch, which is a reason to hold one beyond the
trading perk.

## What already exists, and therefore is not in this price

Everything after the raise closes. The site notices a new pool within a minute of it being created
and gives the token a price, a candle chart at five intervals, its recent trades with wallet
addresses, an exact holder list with each holder's share of supply, a dollar price, and working Buy
and Sell buttons through the existing router. None of that has to be built again. A launchpad here is
only the part before the pool exists, which is why this estimate is as short as it is.

## What gets built

**Three contracts.** A token contract with a fixed supply and no ability to mint more. A factory that
creates a token and its raise together and holds the settings: which assets may be raised in, the
smallest and largest raise allowed in each, the per-wallet cap and the fee. A raise contract, one per
launch, which takes contributions, refunds them all if the target is missed, and on success creates
the Uniswap v4 pool, places the liquidity, locks it, and lets contributors claim their tokens.

**Two screens.** A Create screen where a creator sets the token, the target, the deadline and the
asset to raise in. A raise page where contributors see progress, contribute, and later claim or take
a refund. A third small page lists the live and finished raises.

**Raise data in the site's own indexer.** So the raise pages load instantly and correctly rather than
asking the chain on every visit, the same way the markets table works today.

## How it reaches the screen

The site already asks the chain for new activity once a minute and writes what it finds into its own
database, which every screen reads. A raise rides on that same request. The contracts announce each
contribution, refund, claim and launch, and those announcements travel inside the query the site is
already sending, so there is no second indexer, no second schedule and no extra request.

When a raise succeeds it creates its pool in the same transaction. The site therefore sees the raise
close and the market open together: within a minute the token has a price, a chart and a tradeable
page, and the raise page simply points at it.

## What it costs you

| Service | What it does | Per month |
|---|---|---|
| Envio HyperSync | Reads the chain | USD 70 |
| Supabase | The database behind every screen | USD 25 |
| Vercel | Hosts the site, runs the scheduled jobs | USD 20 |
| Alchemy | Backup chain access | free tier |
| Domain | The address people type | about USD 1 |
| **Total** | **Unchanged by the launchpad** | **about USD 116** |

The launchpad adds nothing to that, for the reason above. The database is the only line that moves
with growth: it holds 7.8 GB today and settles near 10 GB, because old trades and candles are deleted
nightly on a fixed schedule. Storage past the included allowance is billed by the gigabyte and is a
rounding error at this size.

**One-time costs.** Putting the three contracts on chain costs a few dollars of gas. Rehearsing a real
launch end to end needs roughly USD 50 to 100 of test funds, most of which comes back.

**The one real expense: an independent audit.** The raise contract holds contributors' money between
the contribution and the launch. That is the one place in this system where a mistake costs users
directly, and the one cost worth spending real money on. An independent audit of a contract this size
typically runs USD 3,000 to 15,000 and takes one to three weeks. You can launch without one and carry
that risk, or budget for one. Either way the call belongs before the build starts, not after.

## What we need from you before starting

1. **Who may launch.** Anyone with a wallet, or Suits NFT holders only.
2. **The launch fee.** A percentage of a successful raise, a flat fee, or none. The old 1% went away
   when the launch layer did, so this is a fresh decision. It is earned only on success.
3. **Supply and split.** The original proposal described a 60/30/10 token standard. Confirm the split
   and who receives each part.
4. **Which assets a raise may be denominated in.** All 54 allowlisted quote assets, or a short list so
   new pools land with real depth.
5. **Raise limits.** The smallest and largest raise allowed, per asset.
6. **Geography and identity.** Whether contributors must be screened in any way, and which
   jurisdictions are excluded. This is a question for your counsel, not for us.

## Risks, stated plainly

**Issuing is not listing.** Today the site observes pools other people made. A launchpad makes
MAXLONG the party that creates and sells new tokens, and the raise is denominated in tokenised
equities. That is a different legal position and it deserves your counsel's opinion before the build,
not after it.

**The money sits somewhere.** Between a contribution and a launch, the raise contract holds
contributors' funds. That is what the audit question above is about.

**A raise can fail in public.** Some launches will miss their target and refund. The product has to
make that look like a normal outcome rather than a fault, which is a design point, not an afterthought.

**Our own launches get no favours.** Tokens launched here rank in the table by the same rules as
every other token. The site's credibility as an aggregator depends on that, and we will not change it.
