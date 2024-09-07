## Interview Question

How do you handle primary keys (IDs) after sharding and partitioning a database?

## Interviewer Psychological Analysis

This is a critical issue you will inevitably face after implementing sharding and partitioning: How do you generate IDs? When data is distributed across multiple tables, simply using auto-incrementing IDs in each table is inadequate because you need a **globally unique** ID. This is a fundamental consideration in a production environment.

## Interview Question Analysis

### Database-Based Solutions

#### Auto-Incrementing IDs in Databases

One approach is to generate IDs by inserting a dummy record into a table in a central database to get an auto-incrementing ID. Once the ID is generated, it can be used in the sharded and partitioned tables.

- **Advantages**: Simple and straightforward to implement; commonly used.
- **Disadvantages**: **Single-database bottleneck**. High concurrency can create a bottleneck since IDs are generated from a single database. You can mitigate this by creating a service that retrieves the current maximum ID, increments it, and returns a batch of IDs, but it still relies on a single database.

**Suitable for**: Situations where sharding is primarily driven by large data volumes rather than high concurrency.

#### Database Sequences or Table Auto-Increment Field Step Sizes

You can configure sequences or auto-increment fields in databases to support horizontal scaling. For instance, with 8 service nodes, each node uses a different sequence with a starting ID and increment step size of 8.

![database-id-sequence-step](./images/database-id-sequence-step.png)

**Suitable for**: Cases where you want to avoid ID duplication and maintain performance, but this approach can be problematic if you need to add more service nodes.

### UUID

UUIDs (Universally Unique Identifiers) are generated locally and do not depend on a database.

- **Advantages**: No need for a centralized ID generator.
- **Disadvantages**: UUIDs are long and take up more storage space, leading to poorer performance as primary keys. They are not sequential, which results in random writes in B+ tree indexes and can degrade performance due to frequent disk writes.

**Suitable for**: Generating unique filenames or identifiers where high performance is not a primary concern, but not recommended as primary keys due to performance issues.

```java
UUID.randomUUID().toString().replace("-", "") -> sfsdf23423rr234sfdaf
Current System Time
Generating IDs based on the current time, combined with other business fields, can be used, but high concurrency can lead to ID duplication.
```

Suitable for: When combining current time with other unique business fields to generate an ID, as long as the ID generation pattern is acceptable for the business.

### Snowflake Algorithm
The Snowflake algorithm, developed by Twitter, generates unique IDs using a 64-bit long value. It includes:

1 bit unused (always 0).
41 bits for a timestamp in milliseconds (representing 69 years).
10 bits for worker ID (up to 1024 machines).
12 bits for sequence number (4096 IDs per millisecond).

``` java
Copy code
public class IdWorker {
    // Snowflake algorithm implementation
    // Includes timestamp, datacenter ID, worker ID, and sequence number
    // Detailed code omitted for brevity
}
```
Suitable for: High-concurrency systems where performance is critical. The Snowflake algorithm is effective for distributed ID generation and can handle high throughput.

Summary
Database Auto-Increment IDs: Simple but may create bottlenecks.
Sequences or Step Sizes: Good for moderate scaling but less flexible for dynamic scaling.
UUIDs: Suitable for non-primary key scenarios.
Current Time: Practical for lower concurrency scenarios.
Snowflake Algorithm: Optimal for high-concurrency and distributed systems.
