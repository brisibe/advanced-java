## Interview Questions

Can you explain how Redis cluster mode works? How is key addressing handled in cluster mode? What distributed addressing algorithms are there? Are you familiar with the Consistent Hashing algorithm?

## Interviewer’s Psychological Analysis

A few years ago, if you wanted to use multiple Redis nodes, you had to rely on middleware like `codis` or `twemproxy` to handle data distribution across multiple Redis instances.

Recently, Redis has evolved, and the native Redis cluster mode allows deploying multiple Redis instances across different machines, with each instance storing a portion of the data. Each Redis master instance can have Redis slave instances to ensure high availability through automatic failover.

Now, with Redis's built-in cluster support, interviewers will likely focus on Redis cluster-related questions. If you haven't used Redis cluster, you should at least familiarize yourself with its concepts.

For scenarios with minimal data but high concurrency, a single Redis instance with replication (one master and multiple slaves) along with a sentinel cluster might be sufficient. However, Redis cluster is designed for **large data volumes, high concurrency, and high availability**. Redis cluster supports multiple master nodes, each with multiple slave nodes, allowing horizontal scaling.

## Breakdown of Interview Questions

### Redis Cluster Overview

- **Automatic Sharding**: Data is automatically partitioned across master nodes.
- **Built-in High Availability**: The cluster continues to operate even if some master nodes are unavailable.

In Redis cluster mode, each Redis node opens two ports: the primary port (e.g., 6379) and an additional port (e.g., 16379). The latter is used for inter-node communication and cluster bus operations, such as fault detection, configuration updates, and failover authorization. This communication uses the `gossip` protocol, which is efficient and reduces network bandwidth usage.

### Inter-node Communication Mechanism

#### Basic Communication Principles

Cluster metadata can be managed using two approaches: centralized storage or the Gossip protocol. Redis cluster uses the Gossip protocol for inter-node communication.

**Centralized Storage**: Metadata (node information, failures, etc.) is stored in a single node. For example, `storm`, a distributed real-time computation engine, uses ZooKeeper for centralized metadata storage.

![zookeeper-centralized-storage](./images/zookeeper-centralized-storage.png)

**Gossip Protocol**: Every node maintains a copy of the metadata. When changes occur, nodes propagate these updates to other nodes, ensuring all nodes are updated.

![Redis-gossip](./images/redis-gossip.png)

**Centralized Storage Advantages**:
- Immediate updates to metadata are reflected quickly.
- Efficient for metadata reads and writes.

**Gossip Protocol Advantages**:
- Distributed metadata updates reduce pressure on any single node.
- More resilient to failures but can have update delays.

- **10000 Port**: Each node has a dedicated port for inter-node communication, e.g., if the service port is 7001, the communication port is 17001. Nodes periodically send `ping` messages to other nodes, and these nodes respond with `pong`.

- **Exchanged Information**: Includes failure notifications, node additions/removals, and hash slot information.

#### Gossip Protocol Messages

- **meet**: A node sends a `meet` message to a new node, allowing it to join the cluster.
  
  ```bash
  Redis-trib.rb add-node
ping: Nodes frequently send ping messages containing their status and metadata.
pong: Responses to ping and meet messages, used for broadcasting and updating information.
fail: A node sends a fail message to notify other nodes of a failure.
Ping Message Details
Frequent ping messages can add network load. Nodes send ping messages 10 times per second to the five oldest nodes. If a node’s communication delay reaches cluster_node_timeout / 2, it sends a ping immediately to avoid long data exchange delays. This helps maintain consistency in the cluster.

Distributed Addressing Algorithms
Hash Algorithm: Maps keys to nodes using hash values and modulo operation. When a master node fails, requests may fail to retrieve data, causing a cache avalanche and overloading the database.



Consistent Hashing Algorithm: Maps hash values onto a virtual ring. Each master node is placed on this ring based on its hash. A key is placed at the nearest master node in a clockwise direction. This reduces the impact of node failures and additions.



Consistent hashing introduces virtual nodes to handle uneven node distributions and cache hotspots.

Redis Cluster Hash Slot Algorithm: Redis cluster uses 16,384 hash slots. Each key is hashed and mapped to a slot. Masters handle a portion of these slots, making scaling and slot migration efficient. Clients can use hash tags to ensure related keys are stored in the same slot.



Redis Cluster High Availability and Failover
Redis cluster’s high availability is similar to Redis Sentinel.

Node Failure Detection
If a node suspects another node is down (pfail or subjective failure), it broadcasts this to other nodes. If a majority of nodes agree, the node is marked as fail or objective failure. This is similar to Sentinel's sdown and odown.

If a node doesn’t respond within cluster-node-timeout, it is considered pfail. If a majority of nodes agree a node is pfail, it becomes fail.

Slave Node Promotion
For a failed master node, one of its slave nodes is promoted to master to maintain cluster operation.

#### Checking Slave Node Disconnection

Each slave node's disconnection from its master is monitored. If the disconnection time exceeds `cluster-node-timeout * cluster-slave-validity-factor`, the slave node is **not eligible** to be promoted to master.

#### Slave Node Election

Each slave node sets an election time based on its offset of replicated data from the master. The higher the offset (the more data replicated), the earlier the election time, giving it a higher priority for promotion.

Master nodes begin the slave election voting process. If the majority of master nodes (`N/2 + 1`) vote for a particular slave node, the election passes, and the slave node is promoted to master.

#### Comparison with Sentinel

The entire process is very similar to Sentinel. Thus, Redis Cluster integrates the functionalities of replication and Sentinel directly, making it a powerful solution.
