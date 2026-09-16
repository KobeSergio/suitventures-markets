# Draft question to Envio support (not sent)

Purpose: our deployment has processed 859 M events on a Production Small plan whose stated figure is
about 1 M, and the dashboard shows "This has exceeded the plan limits". Envio documents over-limit
behaviour only for the free Development plan. We need to know what actually happens on a paid plan
before we build on top of this deployment.

---

Subject: Production Small: what happens past the event figure, and how are events counted?

Hi,

We run the indexer `suitventures-indexer` (org `kobesergio`) on Production Small, indexing Robinhood
Chain (4663) from genesis. The backfill finished at about 859 million events processed, and the
deployment page shows "This has exceeded the plan limits". It is serving queries normally and we have
been invoiced only the USD 70 base price.

Three questions:

1. What happens to a paid-plan deployment that is over its event figure? Is it throttled, paused,
   deleted, or billed for overage, and after how long? The docs describe the grace period and deletion
   stages for the Development plan only.

2. The pricing page says "Event counts, chains, and contracts are per deployment". If we create a
   second deployment of the same indexer, does its event count start at zero, independent of the first?
   We would like to redeploy regularly, and each deployment re-indexes from the start block.

3. The same line mentions contracts per deployment. Ours has registered about 130,000 token contracts
   dynamically against the plan's 1,000. Is that figure enforced, and if so, how?

We are happy to move to a plan that fits, but we would like to understand the real limits before
choosing one.

Thanks,
Kobe
