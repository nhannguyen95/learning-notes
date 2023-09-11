## Table of Contents
- [Caching Flavors](#caching-flavors)
- [Application Caching](#application-caching)
  - [Eviction Algorithms](#eviction-algorithms)
    - [Most Recently Used (MRU)](#most-recently-used-mru)
    - [Least Recently Used (LRU)](#least-recently-used-lru)
    - [Least Frequently Used (LFU)](#least-frequently-used-lfu)
  - [Invalidation](#invalidation)
  - [Cache Patterns](#cache-patterns)
    - [Cache-aside](#cache-aside)
    - [Read-through/Write-through Cache (Cache-as-SoR)](#read-throughwrite-through-cache-cache-as-sor)
  - [Read/Write Strategies](#readwrite-strategies)
    - [Read strategy: lazy-loading](#read-strategy-lazy-loading)
    - [Write strategy: write-through](#write-strategy-write-through)
    - [Write strategy: write-behind](#write-strategy-write-behind)
    - [Write strategy: invalidate-on-write](#write-strategy-invalidate-on-write)
  - [Cache Problems](#cache-problems)
    - [Cache Stampede](#cache-stampede)
  - [Best practices](#best-practices)
- [Web Caching](#web-caching)
- [References](#references)

---

## Caching Flavors

Two main caching flavors:
- Application caching: caches precomputed results from business logic.
- Web caching: caches internet resources by exploiting mechanisms built into the HTTP protocol.

When used effectively, both will protect services and databases from heavy read traffic loads.

## Application Caching

Design principles: we have a large set of files, we’d like an algorithm to determine which k files (cache size) to keep at any point so that number of cache hits is maximized and number of cache misses is minimized.

2 predominant technologies: memcached and Redis, both are in-memory hash tables arbitrary data representing the result of db queries or downstream service API calls.

Common cache use cases:
- Storing user session data.
- Dynamic web pages.
- Results of db queries.

The cache appears to application service as a single store, and objects are allocated to individual cache servers using a hash function on the object key.

### Eviction Algorithms

Eviction policies are needed to prevent caches from using up all available memory. A good strategy in selecting an appropriate eviction policy is to consider the data stored in your cluster and the outcome of keys being evicted.

#### Most Recently Used (MRU)

When we need discard file, we remove the one we just recently accessed. This algorithm prefers to keep around old data that is rarely used instead of data that is frequently accessed. Implementation requires each item in the cache is associated with a timestamp which indicates the access time.

This algorithm works well in scenario where accessing some file is an indicator that you’re unlikely to see the same file again, and it does not work well if we alternative between a small set of files.

#### Least Recently Used (LRU)

When we need to discard a file, we remove the one we haven’t used in the longest time. Same with MRU, implementation requires keeping the access order of the files in the cache. This algorithm tends to keep around files used more often, however if a user’s interest changes, the entire cache can quickly become tuned to the new interest.

This cache tends to work poorly if certain files are acessed every once in a while, consistently, while others are accessed very frequently for a short while and never again, or if we alternative between more files than our cache can hold.

Overall, this is the algorithm you want to use, since it is theoretically very good and in practice both simple and efficient.

#### Least Frequently Used (LFU)

When we need to discard a file, we remove the one that is least frequently used. Implementation requires keeping a counter on each file, stating how many times it’s been accessed. If a file is accessed a lot of a while, then is no longer useful/accessed, it will stick around, so this algorithm probably does poorly if access patterns change. On the other hand, if usage patterns stay stable, it’ll do well.

### Invalidation

Data in cache are removed/evicted by evicted algorithms, but should also be removed/invalidated if it becomes stale/expired after some time. For example:
- Data can become stale if the underlying data stored in the db is updated by another process without the cache's awareness.
- If cache store result of some calculation, the result can become stale if the calculation's input change.

If the data in cache can become stale, a good strategy is assigning a Time To Live (TTL) to the data to specify "how long to cache this data".

**Most caches often don't consider the TTL when looking for data to evict, this means data can be evicted before it has expired**. Some caches such as Amazon ElastiCache however support eviction policies that take TTL into consideration:
- allkeys-lfu/lru: evicts keys regardless of TTL set.
- allkeys-random: evicts random keys reglardless of TTL set.
- volatile-lfu/lru: evicts keys from those that have a TTL set.
- volatile-ttl: evicts keys with shortest TTL set.
- volatile-random: evicts random keys with a TTL set.
- no-eviction: do not evict keys, writes to cache are blocked until memory frees up.

### Cache Patterns

There are several common access patterns when using a cache. We will talk about cache-aside and cache-as-sor patterns.

#### Cache-aside

Cache-aside means application code interacts directly with the cache along side with the system-of-record (SoR, e.g. db), the cache does not interact with the db at all.

Advantages:
- Resilient to cache failure (cache failure, not cache miss): if the cache become unavailable, all requests are essentially handled as a cache miss. Performance will suffer but services will still be able to satisfy the requests.
- Scaling cache-aside platforms (Redis, memcached) is straightforward due to their simple, distributed hash table model.

#### Read-through/Write-through Cache (Cache-as-SoR)

Cache-as-SoR means application code only interacts directly with the cache and treats cache as if it were the primary SoR, and delegates the SoR reading and writing activities to the cache (hence the "through" suffix, meaning that reads/writes go through the cache first).

Advantages:
- Simpler application code (the cache is responsible for reading and writing to the db, relieving the application of this responsibility).

### Read/Write Strategies

A cache pattern can be implemented using a combination of the following read and write strategies.

#### Read strategy: lazy-loading

Cache don't have to be populated on db init, only on cache misses/expires.

Notes:
- This is the most read strategy available.

Advantages:
- Straightforward.
- Keep cache size cost-effective.

Disadvantages:
- Additional roundtrips to cache and db on cache misses/expires.

#### Write strategy: write-through

When data is written, both db and cache get updated. 

Notes:
- Is a good combination with lazy-loading: lazy-loading caches cache misses on reads, write-through populates data on writes, so the two complement each other.
- This strategy is suitable for use cases where the data get udpated and accessed frequently.
- Under this strategy, reads will predonimately come to cache and writes go to db, this is good because many databases can do writes faster when writers aren't contending with readers for locks.

Advantages:
- Cache is always up-to-date with db, no need TTL for cache data.

Disadvantages:
- Infrequently-read data might also get stored in the cache.

#### Write strategy: write-behind

Same with write-through, but the update to db happens asynchronously. This is also known as a write-back cache, and internally is the strategy used by most db engines.

Advantages:
- Increases request responsiveness.

Disadvantages:
- Possible lost updates if the cache server crashes before a db update is completed.

#### Write strategy: invalidate-on-write

When data is written, update the db but invalidate the (data in) cache.

Notes:
- This strategy is useful when invalidating out-speed updating the (data in) cache.
- Thus, this stragety is common if data in cache is linked (a write to 1 single data can affect multiple cache entries) or aggregated data.

### Cache Problems

#### Cache Stampede

Cache Stampede is a varant of thundering herd effect that happens when many different application processes simultaneously request a cache key, get a cache miss/expire, and then each hits the same database query in parallel.

Some solutions:
- Blocking all threads that are simultaneously requesting a particular value and let only one thread through to the db. Once that thread has populated the cache, the other threads will be allowed to read the cached value.
- [Read more](https://distributed-computing-musings.com/2021/12/thundering-herd-cache-stampede/).

### Best practices

Cache usage:
- Cache almost everything.
- Use lazy-loading strategy when you can, apply where you have data that is read often, but written infrequently.
- Always apply a TTL to all of your cache keys, except those you are updating by write-through caching. This helps cache application bugs where you forget to update/delete a given cache key when updating the underlying record because eventually, the cache key will auto-expire and get refreshed.
- For rapid changing data (such as leaderboards), it makes more sense to set a short TTL (of a few seconds) rather than adding write-through caching or complex expiration logic.
- [Russian doll caching](https://signalvnoise.com/posts/3690-the-performance-impact-of-russian-doll-caching).

Cache design:
- Monitor production cache usage to make sure hit and miss rate are in line with expectation.
- For each combination of read/write strategy pair, assess where reads and writes go to (cache or db).
- There must not be any consequences on a cache miss.

## Web Caching

"HTTP has a caching system built-in but it's a TTL-only system"

---

## References

- [Foundations of Scalable Systems [Book]](https://www.oreilly.com/library/view/foundations-of-scalable/9781098106058/)
- [Caching Best Practices](https://aws.amazon.com/caching/best-practices/)
- [Caching patterns](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html)
- [Staying out of TTL hell](https://calpaterson.com/ttl-hell.html)
- [Using Read-through & Write-through in Distributed Cache](https://www.alachisoft.com/resources/articles/readthru-writethru-writebehind.html)
