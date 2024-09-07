## Interview Question

How do you ensure message order?

## Interviewer’s Psychological Analysis

This is also a standard question when discussing MQ (Message Queue). First, the interviewer wants to see if you understand the concept of message order. Second, they want to know if you have a method for ensuring that messages are processed in order. This is a common issue in production systems.

## Question Breakdown

Let me give an example: We once developed a MySQL `binlog` synchronization system that dealt with very high loads, synchronizing over a billion rows daily. This means data from one MySQL database was synced intact to another MySQL database (MySQL -> MySQL). A common scenario is when a big data team needs to sync a MySQL database and perform various complex operations on the company's business system data.

When you perform insert, update, or delete operations in MySQL, you generate three corresponding `binlog` logs. These three logs are then sent to the MQ and consumed sequentially. At a minimum, you must ensure that these logs are processed in order. Otherwise, an originally correct sequence like: insert, update, delete, might end up being executed as delete, update, insert, which would be incorrect.

In the end, the data should be deleted as intended; however, if you get the sequence wrong, the data might end up being preserved, leading to errors in data synchronization.

Let's look at two scenarios where order can be disrupted:

- **RabbitMQ**: One queue, multiple consumers. For example, if the producer sends three pieces of data to RabbitMQ in the sequence data1/data2/data3, and these are pushed into a RabbitMQ memory queue, with three consumers each consuming one of the three pieces of data, consumer 2 might finish first, storing data2 in the database, followed by data1/data3. This would clearly disrupt the order.

![rabbitmq-order-01](./images/rabbitmq-order-01.png)

- **Kafka**: Suppose we have a topic with three partitions. When the producer writes data, it can specify a key, such as an order ID. Data related to this order will be distributed to the same partition, and the data within this partition will maintain order. Consumers reading from the partition will also see the data in order. However, if the consumer uses **multiple threads to process messages concurrently**, the order might be disrupted. If a single-threaded consumer processes data, and processing is time-consuming (e.g., processing a message takes several milliseconds), it may handle only a few messages per second, which is inefficient. Multiple threads might lead to order disruptions.

![kafka-order-01](./images/kafka-order-01.png)

### Solutions

#### RabbitMQ

Split into multiple queues, with one consumer per queue. This approach adds complexity and can reduce throughput. Alternatively, use a single queue with one consumer, and the consumer can use an internal memory queue to handle and distribute messages to different workers.

Note that the consumer does not directly consume messages but hashes messages based on a key (e.g., order ID). Messages with the same hash value are stored in the same memory queue. This ensures that messages requiring ordering are stored in the same memory queue and processed by a single worker.

#### Kafka

- A single topic, single partition, single consumer, with single-threaded consumption. Single-threaded processing is inefficient, so this is generally not used.
- Create N memory queues, where data with the same key goes to the same memory queue. Use N threads, with each thread consuming from a different memory queue to maintain order.

![kafka-order-02](./images/kafka-order-02.png)
