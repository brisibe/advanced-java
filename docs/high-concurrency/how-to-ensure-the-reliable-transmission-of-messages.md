## Interview Question

How do you ensure reliable message delivery? In other words, how do you handle message loss?

## Interviewer Analysis

This is a fundamental question. A basic principle of using message queues (MQ) is that **there should be neither too many messages nor too few**. "Not too many" relates to issues of repeated consumption and idempotency. "Not too few" means ensuring messages are not lost, especially for critical messages like billing or charging data, where you must guarantee no data is lost during message transmission.

## Question Breakdown

Data loss can occur at the producer, the message queue (MQ), or the consumer. Let’s analyze this for both RabbitMQ and Kafka.

### RabbitMQ

#### Data Loss at the Producer

When a producer sends data to RabbitMQ, it might get lost in transit due to network issues or other reasons. To avoid this, RabbitMQ offers a transaction feature. The producer can open a transaction (`channel.txSelect()`), send the message, and if RabbitMQ fails to receive it, the producer will get an error, rollback the transaction (`channel.txRollback()`), and retry sending the message. If RabbitMQ successfully receives the message, the producer commits the transaction (`channel.txCommit()`).

```java
try {
    // Create a connection through the factory
    connection = factory.newConnection();
    // Get a channel
    channel = connection.createChannel();
    // Start a transaction
    channel.txSelect();

    // Send a message
    channel.basicPublish(exchange, routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, msg.getBytes());

    // Simulate an error
    int result = 1 / 0;

    // Commit the transaction
    channel.txCommit();
} catch (IOException | TimeoutException e) {
    // Catch the exception and rollback the transaction
    channel.txRollback();
}
However, the transaction mechanism is synchronous and can reduce throughput as it is performance-intensive.

Instead, most systems use the confirm mode in RabbitMQ. With confirm, each message is assigned a unique ID, and RabbitMQ sends an acknowledgment (ack) once it successfully processes the message. If it fails to handle the message, RabbitMQ sends a negative acknowledgment (nack), and the producer can retry. You can maintain the status of each message ID in memory and retry if no ack is received within a certain time frame.

The key difference between transactions and confirm is that transactions are synchronous, meaning the process waits for the transaction to complete before proceeding. In contrast, confirm is asynchronous, allowing the producer to continue sending messages while RabbitMQ asynchronously notifies it of message receipt.

To prevent data loss on the producer side, it’s best to use confirm mode.

A channel in transaction mode cannot be set to confirm mode. These two modes are mutually exclusive.

There are three ways to implement producer confirm mode:

Simple confirm mode: After sending a message, call waitForConfirms() to wait for the server to confirm receipt. If the server returns false or doesn’t respond within a certain time, the client can retry.

java
Copy code
channel.basicPublish(ConfirmConfig.exchangeName, ConfirmConfig.routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, ConfirmConfig.msg_10B.getBytes());
if (!channel.waitForConfirms()) {
    // Message sending failed
    // ...
}
Batch confirm mode: After sending a batch of messages, call waitForConfirms() to wait for the server to confirm.

java
Copy code
channel.confirmSelect();
for (int i = 0; i < batchCount; ++i) {
    channel.basicPublish(ConfirmConfig.exchangeName, ConfirmConfig.routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, ConfirmConfig.msg_10B.getBytes());
}
if (!channel.waitForConfirms()) {
    // Message sending failed
    // ...
}
Asynchronous confirm mode: Provide a callback method that is invoked when the server confirms one or more messages.

java
Copy code
SortedSet<Long> confirmSet = Collections.synchronizedSortedSet(new TreeSet<Long>());
channel.confirmSelect();
channel.addConfirmListener(new ConfirmListener() {
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        if (multiple) {
            confirmSet.headSet(deliveryTag + 1).clear();
        } else {
            confirmSet.remove(deliveryTag);
        }
    }

    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("Nack, SeqNo: " + deliveryTag + ", multiple: " + multiple);
        if (multiple) {
            confirmSet.headSet(deliveryTag + 1).clear();
        } else {
            confirmSet.remove(deliveryTag);
        }
    }
});

while (true) {
    long nextSeqNo = channel.getNextPublishSeqNo();
    channel.basicPublish(ConfirmConfig.exchangeName, ConfirmConfig.routingKey, MessageProperties.PERSISTENT_TEXT_PLAIN, ConfirmConfig.msg_10B.getBytes());
    confirmSet.add(nextSeqNo);
}
Data Loss in RabbitMQ
To prevent RabbitMQ from losing data, enable persistence. When a message is written to RabbitMQ, it is saved to disk. Even if RabbitMQ crashes, it will recover the data from the disk upon restart. To ensure data is not lost, there are two steps:

Mark the queue as persistent when created. This ensures that RabbitMQ will persist the queue's metadata, but not the data in the queue.
Set the message’s deliveryMode to 2 when sending it. This marks the message as persistent, and RabbitMQ will persist the message to disk.
If persistence is enabled, RabbitMQ will store messages on disk. However, if RabbitMQ crashes before a message is persisted, some data may be lost. Combining persistence with the producer’s confirm mechanism can mitigate this issue: RabbitMQ will only acknowledge (ack) messages after they are persisted, so if a crash occurs before persistence, the producer will not receive an ack and can resend the message.

Data Loss at the Consumer
Consumers might lose data if they receive a message but fail to process it (e.g., due to a crash) before acknowledging (ack). In this case, RabbitMQ assumes the message was processed and the data is lost.

To prevent this, disable RabbitMQ’s auto ack and use manual acknowledgments. Only acknowledge a message (ack) after processing it. If the consumer crashes before ack, RabbitMQ will reassign the message to another consumer, ensuring it’s not lost.

RabbitMQ provides an acknowledgment mechanism to ensure messages are reliably delivered to consumers. When noAck=false, RabbitMQ waits for the consumer to send an explicit acknowledgment before removing the message from memory (or disk if it’s persistent).

Kafka
Data Loss at the Consumer
The only way a Kafka consumer might lose data is if it consumes a message, automatically commits the offset, and then crashes before processing the message. Kafka assumes the message has been fully processed, but it hasn't, leading to data loss.

To prevent this, disable automatic offset commit and commit offsets manually after processing each message. However, this may lead to duplicate consumption, where a message is consumed again if the consumer crashes before committing the offset. This can be addressed with idempotency.

One issue encountered in production environments is when Kafka consumers buffer data in an internal queue. If the consumer crashes before the data in the queue is processed, data is lost.

Data Loss in Kafka
A common scenario where Kafka might lose data is when a broker crashes and a new partition leader is elected. If the previous leader hadn’t fully replicated some data to followers, the new leader might be missing some messages, resulting in data loss.

To avoid this, configure Kafka with the following:

Set the topic's replication.factor to greater than 1, ensuring each partition has at least 2 replicas.
Set the Kafka broker’s min.insync.replicas parameter to greater than 1, ensuring at least one follower remains in sync with the leader.
On the producer side, set acks=all, which requires each message to be written to all replicas before it is considered successfully written.
Set retries=MAX on the producer side, which makes the producer retry indefinitely if a write fails.
In production, these settings ensure data is not lost when a broker fails and leader election occurs.

Data Loss at the Producer
If acks=all is set on the producer, data loss will not occur. The producer will only consider a message successfully sent if the leader and all replicas have written the data. If this condition is not met, the producer will retry indefinitely.

RocketMQ
Message Loss Scenarios
The producer may lose messages when sending them to MQ.
MQ may lose messages after receiving them but before writing them to disk.
The disk may fail after a message is written, causing data loss.
The consumer may lose messages after receiving them from MQ.
The entire MQ node may crash, leading to message loss.
Preventing Message Loss During Sending
RocketMQ has a transactional message mechanism to prevent message loss during sending. The producer first sends a half message (a wrapper of the original message) that is invisible to consumers. MQ returns an acknowledgment indicating whether it received the message. The producer then executes its local transaction and returns a status to MQ (Commit, Rollback, etc.). If the status is Commit, MQ delivers the message to downstream consumers. If it’s Rollback, MQ discards the message.
