## Caching Flavors

Two main caching flavors:
- Application caching: caches precomputed results from business logic.
- Web caching: caches internet resources by exploiting mechanisms built into the HTTP protocol.

When used effectively, both will protect services and databases from heavy read traffic loads.

---

## Application Caching

**Design principle**
We have a large set of files, we’d like an algorithm to determine which k files (cache size) to keep at any point so that number of cache hits is maximized and number of cache misses is minimized.



## Eviction Algorithms

### Most Recently Used

When we need discard file, we remove the one we just recently accessed. This algorithm prefers to keep around old data that is rarely used instead of data that is frequently accessed. Implementation requires each item in the cache is associated with a timestamp which indicates the access time.

This algorithm works well in scenario where accessing some file is an indicator that you’re unlikely to see the same file again, and it does not work well if we alternative between a small set of files.

### Least Recently Used

When we need to discard a file, we remove the one we haven’t used in the longest time. Same with MRU, implementation requires keeping the access order of the files in the cache. This algorithm tends to keep around files used more often, however if a user’s interest changes, the entire cache can quickly become tuned to the new interest.

This cache tends to work poorly if certain files are acessed every once in a while, consistently, while others are accessed very frequently for a short while and never again, or if we alternative between more files than our cache can hold.

Overall, this is the algorithm you want to use, since it is theoretically very good and in practice both simple and efficient.

### Least Frequently Used

When we need to discard a file, we remove the one that is least frequently used. Implementation requires keeping a counter on each file, stating how many times it’s been accessed. If a file is accessed a lot of a while, then is no longer useful/accessed, it will stick around, so this algorithm probably does poorly if access patterns change. On the other hand, if usage patterns stay stable, it’ll do well.

---

## Web Caching

---

## References
- [Designing Data-Intensive Application](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321).
- [Caching in theory and practice](https://docs.google.com/document/d/1WhtjVDqdVONBVjD2TdZmUBXg37iE1TK-D1_lsmXZXVs/edit#)