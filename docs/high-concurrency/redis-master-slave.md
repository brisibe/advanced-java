# Redis Master-Slave Architecture

A single Redis instance can handle approximately tens of thousands to hundreds of thousands of QPS. For caching purposes, it is typically used to support **high read concurrency**. Therefore, the architecture is designed as a master-slave (master-slave) setup, with one master and multiple slaves. The master handles writes and replicates data to other slave nodes, while slave nodes handle reads. All **read requests go to slave nodes**. This also facilitates horizontal scaling and **supports high read concurrency**.

![Redis-master-slave](./images/redis-master-slave.png)

Redis replication -> Master-Slave Architecture -> Read-Write Separation -> Horizontal Scaling for High Read Concurrency

## Core Mechanism of Redis Replication

- Redis uses **asynchronous** replication to copy data to slave nodes. Starting from Redis 2.8, slave nodes periodically confirm the amount of data they have copied.
- A single master node can be configured with multiple slave nodes.
- Slave nodes can also connect to other slave nodes.
- Replication from the slave node does not block the master node’s normal operations.
- While replicating, a slave node does not block its own queries. It uses an old dataset to serve requests. However, once replication is complete, the old dataset is removed and the new dataset is loaded, which causes a temporary suspension of services.
- Slave nodes are mainly used for horizontal scaling and read-write separation. Adding more slave nodes can improve read throughput.

Note: If using a master-slave architecture, it is essential to **enable** persistence on the master node, as outlined in the [Redis Persistence Documentation](/docs/high-concurrency/redis-persistence.md). Using a slave node as a hot backup for the master node is not recommended, as data may be lost if the master’s persistence is turned off. In the event of a master failure and restart, the data could be empty, and replication might result in lost data on the slave nodes.

Additionally, various backup solutions for the master node should be implemented. If all local files are lost, recover the master from a backup RDB file to **ensure data availability** upon startup. Even if using [Redis Sentinel for High Availability](/docs/high-concurrency/redis-sentinel.md), the system might not detect the master failure in time, which could lead to clearing of data on all slave nodes if the master node restarts automatically.

## Core Principles of Redis Master-Slave Replication

When a slave node starts, it sends a `PSYNC` command to the master node.

If this is the first time the slave node connects to the master, a `full resynchronization` (full replication) is triggered. At this point, the master starts a background thread to generate an `RDB` snapshot file and caches all newly received write commands in memory. After the `RDB` file is generated, the master sends it to the slave, which writes it to local disk and then loads it into memory. The master then sends the cached write commands to the slave, which synchronizes this data. If there is a network failure between the slave and master, the slave will automatically reconnect, and the master will only replicate the missing data.

![Redis-master-slave-replication](./images/redis-master-slave-replication.png)

### Point-in-Time Resumption for Replication

Starting with Redis 2.8, point-in-time resumption is supported for master-slave replication. If the network connection is lost during replication, it resumes from the last replicated offset rather than starting from scratch.

The master node maintains a backlog in memory, and both the master and slave store a replica offset and master run ID. The offset is saved in the backlog. If the network connection is lost, the slave will request the master to continue replication from the last replica offset. If the offset is not found, a `resynchronization` is performed.

> Identifying the master node based on host+IP is unreliable. If the master node restarts or data changes, the slave should differentiate based on different run IDs.

### Diskless Replication

The master creates the `RDB` in memory and sends it to the slave without writing it to disk. Enable this feature in the configuration file with `repl-diskless-sync yes`.

```bash
repl-diskless-sync yes

```

# Wait 5 seconds before starting replication to allow more slaves to reconnect
repl-diskless-sync-delay 5
Expired Key Handling
Slaves do not handle key expiration; they wait for the master to handle expired keys. If the master expires a key or uses LRU to evict a key, it simulates a del command and sends it to the slaves.

### Complete Replication Process
When a slave node starts, it saves the master node’s information, including host and IP, but the replication process has not yet begun.

The slave node has a scheduled task that checks every second for new master nodes to connect and replicate. If found, it establishes a socket connection with the master node and sends a ping command. If the master requires authentication (via requirepass), the slave must send the masterauth password. The master node performs the initial full replication, sending all data to the slave node. Subsequent write commands are asynchronously replicated to the slave.



Full Replication
The master executes bgsave to generate an RDB snapshot file.
The master sends the RDB snapshot file to the slave. If the RDB transfer exceeds 60 seconds (repl-timeout), the slave considers replication failed. This parameter can be adjusted (e.g., for 1 Gbps network cards, 100MB per second transfer, a 6GB file may exceed 60 seconds).
During RDB generation, the master caches all new write commands in memory. After the slave saves the RDB, the master sends the new write commands to the slave.
If memory buffer consumption exceeds 64MB or 256MB in a single instance during replication, replication is stopped, and it fails.
``` bash
Copy code
client-output-buffer-limit slave 256MB 64MB 60
```
After receiving the RDB, the slave clears its old data and loads the new RDB into memory. Note that the slave will continue to serve requests based on the old dataset until the old data is cleared.
If AOF is enabled on the slave, it immediately performs BGREWRITEAOF to rewrite the AOF.
### Incremental Replication
If the network connection is lost during full replication, incremental replication is triggered when the slave reconnects to the master.
The master retrieves the missing data from its backlog and sends it to the slave. By default, the backlog is 1MB.
The master retrieves data from the backlog based on the offset sent by the slave in the psync command.
Heartbeat
Master and slave nodes exchange heartbeat information.

The master sends a heartbeat every 10 seconds, and the slave sends a heartbeat every second.

### Asynchronous Replication
The master writes data internally upon receiving write commands and asynchronously sends it to the slave.

### How Redis Achieves High Availability
If the system can provide 99.99% availability over 365 days, it is considered highly available.

If a slave fails, it does not affect availability, as other slaves continue to provide the same data and service.

However, if the master node fails, writing data becomes impossible, and write caches become invalid. The slave nodes become ineffective as they no longer receive data from the master, rendering the system unusable.

Redis achieves high availability through failover fault tolerance, also known as master-slave switching.

When the master node fails, it automatically detects the failure and promotes a slave node to master. This process ensures high availability in Redis's master-slave architecture.

Redis's high availability using Sentinel will be discussed in detail later.

