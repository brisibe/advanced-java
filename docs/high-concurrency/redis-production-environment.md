## Interview Question

How is Redis deployed in a production environment?

## Interviewer Psychological Analysis

This question assesses your understanding of the deployment architecture of your company's Redis production cluster. If you are not familiar with it, it indicates a lack of diligence on your part. Consider the following:

- Is your Redis setup a master-slave architecture or a cluster architecture?
- What clustering solution is used?
- Is high availability guaranteed?
- Is persistence enabled to ensure data recovery?
- How much memory is allocated for Redis in production?
- What parameters are configured?
- After load testing, what QPS (queries per second) can your Redis cluster handle?

You should be well-versed in these aspects; otherwise, it suggests you haven't given the matter adequate thought.

## Interview Question Analysis

### Redis Cluster Setup

In a Redis cluster setup with 10 machines:
- **5 machines** are deployed with Redis master instances.
- **5 machines** are deployed with Redis slave instances.
- Each master instance has a corresponding slave instance.
- **5 nodes** provide read and write services externally.
- During peak read and write times, each node might handle up to 50,000 QPS, totaling up to 250,000 requests per second across the 5 machines.

### Machine Configuration

- **Machine specs**: 32GB memory, 8 CPU cores, 1TB disk.
- **Memory allocated to Redis**: Typically 10GB. In a production environment, Redis memory should generally not exceed 10GB to avoid potential issues.

### Memory and Load

- **Total memory available**: 50GB across the 5 machines providing read and write services.
- **High availability**: Each master instance has a corresponding slave. If a master instance fails, automatic failover ensures that a Redis slave instance will promote to master and continue to provide services.

### Data and Performance

- **Data type**: Product data, with each entry approximately 10KB in size.
- **Data volume**: 
  - 100 entries = 1MB
  - 100,000 entries = 1GB
  - **In-memory data**: 2 million product entries, occupying 20GB of memory, which is less than 50% of the total memory.
- **Current peak load**: Approximately 3,500 requests per second.

### Infrastructure and Operations

In large companies, a dedicated infrastructure team typically manages the operations of the caching cluster.
