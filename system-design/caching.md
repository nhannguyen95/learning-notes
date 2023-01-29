TABLE OF CONTENS:
- [Caching Flavors](#caching-flavors)
- [Application Caching](#application-caching)
  * [Cache Patterns](#cache-patterns)
    + [Cache-aside](#cache-aside)
    + [Cache-as-SoR](#cache-as-sor)
  * [Eviction Algorithms](#eviction-algorithms)
    + [Most Recently Used](#most-recently-used)
    + [Least Recently Used](#least-recently-used)
    + [Least Frequently Used](#least-frequently-used)
  * [Invalidation](#invalidation)
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

Best practices on cache design:
- Monitor production cache usage to make sure hit and miss rate are in line with expectation.
- For each combination of read/write strategy pair, assess where reads and writes go to (cache or db).
- There must not be any consequences on a cache miss.

### Eviction Algorithms

Eviction policies are needed to prevent caches from using up all available memory.

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

If the data in cache can become stale, a good strategy is assigning a Time To Live (TTL) to the data to specify "how long to cache this data". **Most caches often don't consider the TTL when looking for data to evict, this means data can be evicted before it has expired**.

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

### Write strategy: write-through.

When data is written, both db and cache get updated. 

Notes:
- This strategy is suitable for use cases where the data get udpated and accessed frequently.
- Under this strategy, reads will predonimately come to cache and writes go to db, this is good because many databases can do writes faster when writers aren't contending with readers for locks.

Advantages:
- Cache is always up-to-date with db, no need TTL for cache data.

Disadvantages:
- Infrequently-read data might also get stored in the cache.

### Write strategy: write-behind

Same with write-through, but the update to db happens asynchronously. This is also known as a write-back cache, and internally is the strategy used by most db engines.

Advantages:
- Increases request responsiveness.

Disadvantages:
- Possible lost updates if the cache server crashes before a db update is completed.

### Write strategy: invalidate-on-write

When data is written, update the db but invalidate the (data in) cache.

Notes:
- This strategy is useful when invalidating out-speed updating the (data in) cache.
- Thus, this stragety is common if data in cache is linked (a write to 1 single data can affect multiple cache entries) or aggregated data.

## Web Caching

---

## References
- [Designing Data-Intensive Application](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321).

https://docs.google.com/document/d/1WhtjVDqdVONBVjD2TdZmUBXg37iE1TK-D1_lsmXZXVs/edit#heading=h.x61xfd9oscc8

https://aws.amazon.com/caching/best-practices/

https://calpaterson.com/ttl-hell.html

https://www.ehcache.org/documentation/3.3/caching-patterns.html#cache-aside

https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html

https://www.alachisoft.com/resources/articles/readthru-writethru-writebehind.html
