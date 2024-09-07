## Interview Questions

Do you know what Redis cache avalanche, penetration, and breakdown are? What happens if Redis crashes? How should the system handle such situations? How do you handle Redis cache penetration?

## Interviewer Psychological Analysis

These questions are essential when discussing caching, as cache avalanche and penetration are two major issues with caching. If they occur, they can be fatal. Therefore, the interviewer is likely to ask you about these.

## Interview Question Analysis

### Cache Avalanche

For System A, assume it experiences 5000 requests per second during peak hours, and the cache can handle 4000 requests per second. If the cache crashes unexpectedly, all 5000 requests will hit the database. The database will likely fail due to the overload, potentially causing a complete system crash. 

This is known as a cache avalanche.

![Cache Avalanche](./images/redis-caching-avalanche.png)

About three years ago, a well-known internet company in China experienced a cache avalanche, causing a complete system crash and significant financial losses.

Solutions for handling cache avalanche:

- **Before the Incident**: Use Redis high availability solutions like master-slave replication with Sentinel or Redis Cluster to prevent a total cache failure.
- **During the Incident**: Implement local caching (e.g., ehcache) and rate limiting or fallback mechanisms to prevent the database from being overwhelmed.
- **After the Incident**: Use Redis persistence to automatically reload data from disk and quickly restore the cache.

![Cache Avalanche Solution](./images/redis-caching-avalanche-solution.png)

The user sends a request, and System A first checks the local ehcache. If not found, it queries Redis. If Redis also doesn’t have it, it checks the database and writes the result to both ehcache and Redis.

Rate limiting components ensure only a limited number of requests pass through, while the rest are handled by fallback mechanisms, such as returning default values or error messages.

Benefits:

- The database will not be overwhelmed as rate limiting controls the request rate.
- As long as the database is operational, a portion of requests (e.g., 2/5) can still be handled.
- Users may experience delays but will eventually get responses after multiple attempts.

### Cache Penetration

For System A, suppose 5000 requests occur per second, but 4000 of these are malicious attacks. These attacks query the database directly since the cache doesn't store the data.

For example, if database IDs start from 1 and the attacks use negative IDs, the cache won’t have these IDs, leading to repeated database queries and potential overload.

![Cache Penetration](./images/redis-caching-penetration.png)

Solution:

- When the system queries the database and finds no result, store an empty value in the cache with an expiration time, e.g., `set -999 UNKNOWN`. This prevents future requests with the same key from hitting the database.

For more sophisticated attacks, use a Bloom filter before the cache to map possible database data hashes. Requests are checked against the Bloom filter:

- If the key does not exist in the Bloom filter, the data is guaranteed not to be in the database, and a non-existence response is returned.
- If the key is in the Bloom filter, proceed to query the cache.

Using a Bloom filter helps pre-screen requests and reduces the load on the database.

![Avoid Cache Penetration](./images/redis-caching-avoid-penetration.png)

### Cache Breakdown (Hotspot Invalid)

Cache breakdown refers to a scenario where a highly accessed key becomes a bottleneck. When this key expires, a large number of requests bypass the cache and hit the database, causing overload.

Solutions for different scenarios:

- **For static data**: Set the hotspot data to never expire.
- **For infrequently updated data with quick cache refreshes**: Use distributed or local mutex locks (e.g., Redis, Zookeeper) to ensure that only a few requests rebuild the cache while others wait for the new cache.
- **For frequently updated data or long cache refresh times**: Use scheduled threads to proactively rebuild the cache before expiration or extend the cache expiration time to ensure continuous access to the data.
