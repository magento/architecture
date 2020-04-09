# Non blocking cache writing mechanism

### Terms

* Stale cache - the previous version of cache that customer will receive until a new version will be written in cache.
* Block cache - Magento cache type, typically consist of cached html output.
* Config cache - Magento cache type, usually consist of cached config data.
* Cache revalidation - process of cache invalidation and writing fresh one in cache storage.
* Lookup for lock, lookup timeout - time that takes to recheck if new version of cache already written.
* stale-while-revalidate - mechanism that send old cache while new one is on the generation phase.

### Overview

Currently, we have two cache types(Block and Config) that uses lock mechanism to avoid parallel cache generation and excessive resource utilization. 
So every time we want to generate and write a new cache, we acquire a lock and parallel process wait until lock released. 
When the lock is released, customer get fresh data from storage.

We usually think that trade-off with lock waiting is acceptable from the performance side. 
But the larger amount of Blocks or Cache merchant has, the more time he will wait in locks. 
In some scenarios, we could wait  **numbers of keys** * **lookup timeout**  amount of time in parallel process. 
We noticed that in some rare cases merchant can have hundreds keys in Block/Config cache, so even small lookup timeout for lock may cost seconds.

We have couple public issue that shows possible pointed limitations of this approach - https://github.com/magento/magento2/issues/22824 https://github.com/magento/magento2/issues/22504

[IvanChepurnyi](https://github.com/IvanChepurnyi) in his [PR](https://github.com/magento/magento2/pull/22829) - proposed to use non-locking way of cache generation a.k.a. *stale-while-revalidate*. 
The purpose of this PR is to wrap all things up, determine all A.C., discuss approach with community and deliver solid solution.

### Design

This approach is well known and already used in some popular libraries, i.e. [Varnish](https://info.varnish-software.com/blog/grace-varnish-4-stale-while-revalidate-semantics-varnish).
Basically, we will send stale cache while we generating a new one. That will free us from needs to wait until any locks will be released, except the case when we have completely empty cache storage.

To do that, we need to extend current Magento\Framework\Cache\LockGuardedCacheLoader that can revalidate the cache only in blocking way.
Also we should not rewrite business logic of Config and Block cache, but rather implement solid separate API/SPI.

To keep efficiency, we still should use locking mechanism when we have  empty cache storage to write a very first version of cache.

#### Acceptance Criteria Fulfillment

1. New code should be compatible with current DOD https://devdocs.magento.com/guides/v2.3/contributor-guide/contributing_dod.html
    1. New functionality should be separated from business logic.
    1. Functional Backward Compatibility.
    Feature should be optional and disabled by default. Since cache freshness of blocks and config data may be critical for our merchants, we should introduce a new config variable that will enable/disable the feature. It should be off by default.
1. Stale data should have TTL. 
Since we don't want to have a constant copy of the stale cache and it is not designed to exist for a long time, I propose to have up to 10 minutes of TTL.
TTL should be added when we save stale data.
1. Efficiency - means no parallel cache generating/writing process either with stale cache or a fresh one.
Basically, only one process should generate or write cache.
1. We keep only one(last) copy of the stale cache over time. 

### Prototype or Proof of Concept

PR with working functionality for config cache - https://github.com/magento/magento2/pull/22829
PR needs to be reworked to fulfill the acceptance criteria.

### Data size and Performance Requirements

1. At least the same efficiency when we don't have any cache.
1. Better efficiency with stale cache in comparison to lock waiting.
