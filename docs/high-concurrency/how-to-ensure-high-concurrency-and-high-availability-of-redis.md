## Interview Questions

How do you ensure high concurrency and high availability for Redis? Can you explain the principles of Redis master-slave replication? Can you also explain the Sentinel mechanism in Redis?

## Interviewer’s Psychological Analysis

The purpose of this question is to assess a few things: How high of a concurrency Redis can handle on a single instance? If a single instance cannot handle the load, how do you scale to manage more concurrency? Since Redis might fail, how do you ensure its high availability?

These are critical considerations for any production system. If you haven’t thought about them, it indicates a lack of experience in handling real-world production issues.

## Breakdown of Interview Questions

If you are using Redis as a caching technology, you need to consider how to scale Redis across multiple machines to ensure high concurrency, as well as how to ensure Redis remains available even if some instances fail, i.e., achieving high availability.

Due to the extensive content, this will be divided into two sections:

- [Redis Master-Slave Architecture](/docs/high-concurrency/redis-master-slave.md)
- [Redis High Availability with Sentinel](/docs/high-concurrency/redis-sentinel.md)

Redis achieves **high concurrency** primarily through the **master-slave architecture**, with one master and multiple slaves. Generally, this setup is sufficient for many projects: the master handles writes (capable of handling tens of thousands of QPS), and multiple slave instances handle reads (providing up to 100,000 QPS per second).

If you need to handle high concurrency while accommodating a large amount of data, Redis clustering is required. Redis clustering can provide read and write concurrency of several hundred thousand QPS per second.

For high availability, if you use a master-slave deployment, adding Sentinel will enable automatic failover, allowing the system to switch between master and slave nodes in the event of a failure.
