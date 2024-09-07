## Interview Questions

How do you ensure consistency between cache and database in a dual-write scenario?

## Interviewer Psychological Analysis

Whenever you use a cache, you might deal with dual storage and dual write issues. With dual writes, data consistency problems are inevitable. How do you address these consistency issues?

## Interview Question Analysis

Generally, if your system does not strictly require "cache + database" to be consistent, it is better to avoid complex solutions like **serializing read and write requests** into an **in-memory queue**.

Serialization can ensure consistency but significantly reduces system throughput, requiring multiple times the usual resources to handle online requests.

### Cache Aside Pattern

The classic pattern for cache and database read-write is the Cache Aside Pattern:

- **Read**: First check the cache. If not found, then query the database, update the cache, and return the response.
- **Update**: **Update the database first, then delete the cache**.

**Why delete the cache instead of updating it?**

In complex caching scenarios, cache values may be derived from computations involving multiple tables, not just direct values from the database. Updating the cache every time a database change occurs can be costly. If the cache involves multiple tables and is frequently updated, it might not be worth updating the cache with every database change. 

Deleting the cache (lazy computation) is akin to the concept of lazy loading in MyBatis or Hibernate, where data is only loaded when accessed, reducing unnecessary computations.

### Basic Cache Inconsistency Issues and Solutions

**Problem**: Update the database first, then delete the cache. If cache deletion fails, the database holds new data while the cache still has old data, causing inconsistency.

![Basic Cache Inconsistency](./images/redis-junior-inconsistent.png)

**Solution 1**: Delete the cache first, then update the database. If database update fails, the cache will have the old data, preventing inconsistency.

**Solution 2**: Implement delayed double deletion. After updating the database and deleting the cache, perform another cache deletion after a short delay (e.g., 5 seconds).

```java
public void set(key, value) {
    putToDb(key, value);
    deleteFromRedis(key);

    // ... a few seconds later
    deleteFromRedis(key);
}
```
Deletions can be handled in various ways, such as using DelayQueue, which may lose updates if the JVM process dies, or using a message queue, which adds coding complexity. Choose the solution based on your needs.

Complex Data Inconsistency Issue Analysis
When data changes, if the cache is deleted before the database update is complete, a read request might get old data from the database, leading to inconsistency.

Why does this occur under high traffic scenarios?

This issue arises primarily in high-concurrency scenarios. With low concurrency, inconsistencies are rare. However, with massive traffic (e.g., billions of requests), inconsistencies are more likely if data is being updated concurrently.

Solutions:

Route operations based on the unique identifier of the data to an internal JVM queue. When data is read, if it’s not in the cache, the system re-executes the "read and update cache" operation, ensuring consistency through the queue.

Use a work thread for each queue to handle operations sequentially. After the database update completes, the thread will handle the cache update.

Optimizations:

If multiple cache update requests are queued, filter out redundant requests to avoid unnecessary updates.
Test and simulate the system under load to ensure that update delays do not lead to excessive read timeouts or blockages.
Considerations:

Read Request Latency: Ensure that read requests return within the timeout range.
High Concurrency: Ensure the system can handle peak loads and distribute updates effectively.
Service Instances: Ensure requests are routed to the same service instance for consistency.
Hotspot Data: Handle potential load imbalances if certain data is highly accessed.
Practical Testing:

Perform stress tests to determine the impact of update frequency on queue accumulation and ensure that the system can handle high loads without significant delays.

For a detailed discussion of this interview question, see #54.
