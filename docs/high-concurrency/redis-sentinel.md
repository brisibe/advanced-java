# Redis Sentinel Cluster for High Availability

## Introduction to Sentinel

Sentinel is a crucial component in the Redis cluster architecture, responsible for several key functions:

- **Cluster Monitoring:** Monitors whether Redis master and slave processes are functioning correctly.
- **Message Notification:** Sends alerts to administrators if any Redis instance encounters issues.
- **Failover:** Automatically promotes a slave to master if the master node fails.
- **Configuration Center:** Updates clients with the new master address if a failover occurs.

Sentinel ensures high availability in Redis clusters and operates in a distributed manner, with multiple Sentinel instances working together:

- **Failover Decision:** To determine if a master node has failed, most Sentinels need to agree, involving a distributed voting mechanism.
- **Fault Tolerance:** Even if some Sentinel nodes fail, the Sentinel cluster can still function properly. A high availability system would be unreliable if it had a single point of failure.

## Core Sentinel Knowledge

- **Minimum Sentinel Instances:** At least 3 Sentinel instances are required to ensure robustness.
- **Redis Master-Slave Architecture:** The Sentinel + Redis master-slave deployment does not guarantee zero data loss but ensures high availability.
- **Testing and Simulation:** Thorough testing and simulation in both staging and production environments are essential for complex Sentinel + Redis master-slave deployments.

A Sentinel cluster must deploy more than 2 nodes. If only 2 Sentinel instances are deployed, the quorum is set to 1.


```
+----+         +----+
| M1 |---------| R1 |
| S1 |         | S2 |
+----+         +----+
```

With `quorum=1`, if the master fails, failover can occur as long as at least 1 Sentinel (out of S1 and S2) detects the master failure. S1 and S2 will elect one Sentinel to perform the failover. However, a majority of Sentinels need to be operational.


```
2 个哨兵，majority=2
3 个哨兵，majority=2
4 个哨兵，majority=2
5 个哨兵，majority=3
...
```

If only the M1 process fails but Sentinel S1 is operational, failover is possible. However, if both M1 and S1's machine fail, with only 1 Sentinel left, there is no majority to allow failover, even though another machine has R1. Thus, failover will not occur.

A classic 3-node Sentinel cluster looks like this:

```
       +----+
       | M1 |
       | S1 |
       +----+
          |
+----+    |    +----+
| R2 |----+----| R3 |
| S2 |         | S3 |
+----+         +----+
```

With `quorum=2`, if the machine hosting M1 fails, the remaining 2 out of 3 Sentinels (S2 and S3) can agree that the master has failed and elect one to perform the failover. The remaining 2 Sentinels are enough to allow the failover to occur.

## Redis Sentinel Master-Slave Switch Data Loss Issues

### Causes of Data Loss

During a master-slave switch, data loss can occur due to:

- **Asynchronous Replication Data Loss:** Since replication from master to slave is asynchronous, some data might not be copied to the slave before the master fails. This results in data loss.

  ![async-replication-data-lose-case](./images/async-replication-data-lose-case.png)

- **Split-Brain Data Loss:** Split-brain occurs when a master machine suddenly loses network connectivity with other slave machines but continues to operate. Sentinels may incorrectly assume the master has failed, leading to an election where other slaves are promoted to master. This results in two masters in the cluster (a split-brain scenario).

  ![Redis-cluster-split-brain](./images/redis-cluster-split-brain.png)

  When the old master recovers, it becomes a slave to the new master, clearing its data and re-syncing from the new master. This leads to data loss if clients continued writing to the old master during its downtime.

### Solutions to Data Loss Issues

Configure the following settings:

```bash
min-slaves-to-write 1
min-slaves-max-lag 10
```

This configuration ensures that at least 1 slave must be available, and data replication and synchronization delays cannot exceed 10 seconds. If delays exceed 10 seconds, the master will stop accepting requests.

### Reducing Asynchronous Replication Data Loss

With `min-slaves-max-lag`, if a slave's data replication delay is too long, write requests are rejected, reducing data loss due to asynchronous replication.

### Reducing Split-Brain Data Loss

If a master experiences split-brain, the configurations ensure that if a slave cannot receive data for over 10 seconds, client write requests are rejected, limiting data loss to at most 10 seconds.

### sdown and odown Transition Mechanisms

- **sdown (Subjective Down):** A master is considered subjective down if a single Sentinel determines it is down.
- **odown (Objective Down):** A master is considered objective down if a quorum number of Sentinels agree it is down.

The condition for `sdown` is straightforward. If a Sentinel pings a master and does not receive a response within the `is-master-down-after-milliseconds` time, it considers the master as `sdown`. If a quorum number of other Sentinels also consider the master as `sdown` within the specified time, it is considered `odown`.

### Sentinel Cluster Auto-Discovery Mechanism

Sentinels discover each other using Redis's `pub/sub` system. Each Sentinel sends a message to the `__sentinel__:hello` channel, which other Sentinels can consume to detect their presence.

Every 2 seconds, each Sentinel sends a message to the `__sentinel__:hello` channel of the master+slaves it monitors. This message includes its host, IP, runid, and monitoring configuration for the master.

Each Sentinel listens to the `__sentinel__:hello` channel of the master+slaves it monitors to detect other Sentinels monitoring the same master+slaves.

Sentinels also exchange and synchronize monitoring configurations for the master.

### Automatic Correction of Slave Configuration

Sentinels automatically correct some configurations of slaves, such as ensuring a slave intended to become a master candidate replicates data from the current master. If a slave connects to an incorrect master after a failover, Sentinels ensure it connects to the correct master.

### Slave-to-Master Election Algorithm

If a master is considered `odown`, and a majority of Sentinels allow a master-slave switch, a Sentinel will execute the switch. The election of a slave to become the new master considers:

- **Disconnection Duration:** The time a slave has been disconnected from the master.
- **Slave Priority:** Lower priority values mean higher preference.
- **Replication Offset:** More recent offsets mean higher preference.
- **Run ID:** Lower run IDs mean higher preference if other factors are equal.

If a slave has been disconnected for more than 10 times the `down-after-milliseconds` value plus the master failure duration, it is considered unsuitable for master election.

Slaves are then sorted based on:

- **Priority:** Lower priority values are preferred.
- **Replication Offset:** Higher offsets are preferred.
- **Run ID:** Lower IDs are preferred if other factors are equal.

### Quorum and Majority

For a Sentinel to perform a master-slave switch, a quorum number of Sentinels must agree on the `odown` status, and a majority of Sentinels must authorize the switch.

If `quorum < majority`, such as 5 Sentinels with a majority of 3 and a quorum of 2, then 3 Sentinels' authorization is required.

If `quorum >= majority`, such as 5 Sentinels with a quorum of 5, all 5 Sentinels must authorize the switch.

### Configuration Epoch

Sentinels monitor a set of Redis master+slaves with corresponding configurations.

The Sentinel performing the switch obtains a configuration epoch (a version number) from the new master (slave-to-master). Each switch must have a unique version number.

If the initially elected Sentinel fails, other Sentinels will wait for the failover timeout period before taking over the switch, obtaining a new configuration epoch as the new version number.

### Configuration Propagation

After a Sentinel completes a switch, it updates its local configuration with the latest master details and synchronizes this with other Sentinels via the `pub/sub` messaging mechanism.

The version number is crucial, as all messages are published and listened to on the same channel. After a switch, the new master configuration follows the new version number. Other Sentinels update their master configurations based on the version number.
