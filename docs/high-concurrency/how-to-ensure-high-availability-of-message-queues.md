## Interview Question

How do you ensure the high availability of message queues?

## Interviewer's Psychological Analysis

If someone asks you about MQ (Message Queue), **high availability is definitely going to be asked**. [In a previous lesson](/docs/high-concurrency/why-mq.md), it was mentioned that MQ can **reduce system availability**. So, as long as you’re using MQ, the key points that follow are definitely going to be about how to solve the disadvantages of MQ.

If you’re clueless and just use an MQ without considering various issues, you’re in trouble. The interviewer will feel that you only know how to use a technology simply, without any deep thinking. Their impression of you will worsen. If you’re applying for a regular role with a salary under 20k, maybe that’s still acceptable, but if it’s for a more senior role with a 20k+ salary, then it’s a disaster. If you're asked to design a system, it will probably have many pitfalls, and if something goes wrong, the company suffers, and the team takes the blame.

## Analysis of the Interview Question

This question is well-phrased because you can’t ask how Kafka ensures high availability or how ActiveMQ ensures high availability. An interviewer who asks this lacks understanding because the person might be using RabbitMQ and not Kafka. Why ask about Kafka then? This would clearly make it difficult for the candidate.

A good interviewer asks: "How do you ensure the high availability of an MQ?" This way, you can explain your understanding of the high availability of the specific MQ you've used.

### High Availability of RabbitMQ

RabbitMQ is a representative example because it ensures high availability based on a **master-slave** (non-distributed) architecture. Let's take RabbitMQ as an example to explain how to ensure high availability for the first type of MQ.

RabbitMQ has three modes: Single node mode, Standard Cluster mode, and Mirrored Cluster mode.

#### Single Node Mode

Single node mode is demo-level and generally only used locally for testing. No one uses single-node mode in production.

#### Standard Cluster Mode (No High Availability)

In standard cluster mode, multiple RabbitMQ instances are launched on multiple machines, with one instance per machine. The **queue you create will only be placed on one RabbitMQ instance**, but each instance synchronizes the metadata of the queue (metadata can be understood as the configuration information of the queue; through metadata, the instance of the queue can be found). When you consume messages, if you connect to another instance, that instance will pull the data from the instance where the queue is located.

![mq-7](./images/mq-7.png)

This method is quite cumbersome and not ideal. **It's not a distributed system**, just a standard cluster. This forces consumers to either randomly connect to an instance and pull data or consistently connect to the instance where the queue resides. The former incurs **data pull overhead**, while the latter causes a **single instance performance bottleneck**.

Moreover, if the instance holding the queue goes down, other instances cannot pull data from it. If you have **enabled message persistence**, RabbitMQ will store the messages to disk, so **messages may not be lost**, but you will have to wait for the instance to recover before pulling data from the queue again.

This is quite awkward as it does not provide **true high availability**. **This approach is mainly used to increase throughput**, allowing multiple nodes in the cluster to handle read and write operations for a queue.

#### Mirrored Cluster Mode (High Availability)

This mode provides **high availability** for RabbitMQ. Unlike standard cluster mode, in mirrored cluster mode, the queue you create, including both metadata and messages, is stored across **multiple instances**. In other words, each RabbitMQ node contains a **complete mirror** of the queue, meaning all the queue’s data. Every time you write a message to the queue, it will automatically be **synchronized** to the queues on multiple instances.

![mq-8](./images/mq-8.png)

So, **how do you enable mirrored cluster mode**? It's quite simple. RabbitMQ has a great management console. In the backend, you can add a policy for **mirrored cluster mode**, which can either synchronize data to all nodes or to a specified number of nodes. When creating a new queue, apply this policy, and data will be automatically synchronized to other nodes.

The advantage of this is that if one machine goes down, no problem—other machines (nodes) still have the complete data of the queue, and other consumers can continue to consume from those nodes. The downside is that, first, this method incurs a huge performance cost because messages need to be synchronized to all machines, which creates significant pressure and consumption of network bandwidth. Second, this approach is not **distributed**, meaning there is **no scalability**. If a queue has a heavy load, adding more machines won't help, as new machines will also contain all the data of the queue, and **there’s no way to scale the queue linearly**. Imagine if the queue’s data volume grows so large that the capacity of one machine is insufficient—what do you do then?

### High Availability of Kafka

A basic understanding of Kafka architecture: it consists of multiple brokers, with each broker being a node. When you create a topic, it can be divided into multiple partitions, and each partition can exist on a different broker, with each partition storing part of the data.

This makes Kafka a **naturally distributed message queue**, meaning that a topic’s data is **spread across multiple machines, with each machine storing part of the data**.

In fact, RabbitMQ is not a distributed message queue; it's a traditional message queue that simply provides cluster and **High Availability (HA)** mechanisms. No matter how it's configured, all the data for a RabbitMQ queue is stored on one node, and in mirrored cluster mode, all nodes store a full copy of that queue's data.

Before Kafka 0.8, there was no HA mechanism. If a broker went down, the partition on that broker would become unusable, meaning you couldn’t read or write to it, so there was no high availability.

For example, let’s assume you create a topic with 3 partitions, which are distributed across three machines. If the second machine goes down, **one-third of the data for that topic is lost**, making high availability impossible.

![kafka-before](./images/kafka-before.png)

Starting from Kafka 0.8, Kafka introduced the HA mechanism, specifically the replica (replication) feature. The data of each partition is synchronized to other machines, creating multiple replica copies. All replicas elect a leader, and both production and consumption interact with this leader, while the other replicas act as followers. When writing, the leader is responsible for synchronizing data to all followers, and when reading, you read directly from the leader. 

Why only read/write from the leader? If you could freely read/write from each follower, **you would need to ensure data consistency**, which adds complexity and makes the system prone to errors. Kafka evenly distributes all the replicas of a partition across different machines, improving fault tolerance.

![kafka-after](./images/kafka-after.png)

With this setup, **high availability** is achieved. If a broker goes down, it's fine—other machines still have replicas of the partitions from that broker. If a partition’s leader goes down, a new leader is elected from the followers, and consumers can continue to read/write from the new leader. This ensures high availability.

When **writing data**, the producer writes to the leader, which writes the data to its local disk, and the followers pull the data from the leader. Once all followers have successfully synchronized the data, they send an acknowledgment (ack) to the leader, which then sends a success message to the producer. (Of course, this is just one mode; the behavior can be adjusted.)

When **consuming data**, you can only read from the leader. However, you can only consume messages that have been successfully acknowledged by all followers.

By now, you should have a basic understanding of how Kafka ensures high availability. You should be able to explain this and even draw diagrams for the interviewer. If the interviewer is a Kafka expert and dives deep, you can simply admit that you haven’t researched those deeper aspects yet.
