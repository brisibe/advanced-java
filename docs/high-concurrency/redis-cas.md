## Interview Questions

What are Redis concurrency issues? How can these issues be resolved? Are you familiar with Redis transactions and the CAS (Compare-And-Swap) approach?

## Interviewer Psychological Analysis

This is a common issue in online systems: **multiple clients concurrently writing** to the same key can result in data being overwritten in the wrong order, leading to incorrect data versions. Similarly, if multiple clients retrieve a key, modify its value, and then write it back, incorrect data can result from ordering issues.

Redis provides a built-in CAS-like optimistic locking mechanism to address these issues.

## Interview Question Analysis

When multiple system instances attempt to update the same key simultaneously, distributed locking can be used to ensure that only one instance performs the update at a time. Zookeeper is a popular tool for implementing distributed locks:

- **Zookeeper Distributed Lock**: Each system instance obtains a distributed lock from Zookeeper to ensure that only one instance is allowed to operate on a key at any given time. Other instances are prevented from reading or writing until the lock is released.

![Zookeeper Distributed Lock](./images/zookeeper-distributed-lock.png)

To handle cache consistency:

- **Write Process**: Data written to the cache must come from MySQL. When writing to MySQL, save a timestamp along with the data. When retrieving data from MySQL, also retrieve the timestamp.

- **Before Writing**: **Check** if the timestamp of the data to be written is newer than the timestamp of the existing cached data. If it is, proceed with the write; otherwise, do not overwrite the existing data with outdated information.

The CAS (Compare-And-Swap) approach is often used in Redis transactions to handle concurrency issues, ensuring that data is only updated if it has not been changed by another process in the meantime. This optimistic locking approach helps in maintaining data consistency in high-concurrency scenarios.
