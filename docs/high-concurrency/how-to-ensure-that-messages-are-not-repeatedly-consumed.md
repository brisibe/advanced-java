## Interview Question

How do you ensure that messages are not consumed multiple times? Or, how do you guarantee idempotency in message consumption?

## Interviewer’s Thought Process

This is actually a common question, and these two issues are often asked together. When consuming messages, you need to consider whether they might be consumed more than once. Can repeated consumption be avoided? Or, if messages are consumed multiple times, can the system handle it without errors? This is a basic problem in the domain of message queues (MQ), and at its core, it’s about asking how you can ensure **idempotency** when using a message queue. This is a key consideration in your system architecture.

## Question Breakdown

To answer this question, don’t get flustered when you hear about duplicate message consumption. You should start by **briefly explaining the possible reasons for repeated message consumption**.

For example, RabbitMQ, RocketMQ, and Kafka can all encounter situations where messages are consumed multiple times, and this is normal. Typically, the MQ itself doesn’t guarantee this; **it’s up to us as developers to ensure idempotency**. Let’s take Kafka as an example to explain how repeated consumption might occur.

Kafka has the concept of an offset. Every message written into Kafka gets assigned an offset, which is its sequence number. When a consumer consumes messages, **every so often**, it will commit the offset, saying, "I’ve consumed up to this point, so if I restart, continue from this offset."

But there are always exceptions. For instance, if your consumer process gets killed abruptly, some messages may have been processed but the offset wasn’t submitted in time. This can lead to messages being re-consumed after the system restarts.

### Example:

Imagine three messages (1, 2, 3) enter Kafka, and Kafka assigns them offsets (e.g., 152, 153, and 154). The consumer processes these messages in order. Now, suppose that after processing the message with offset 153, but before committing the offset, the consumer process is restarted. Kafka, having no record of the last successful offset commit, will resend messages 1 and 2 for processing.

![mq-10](./images/mq-10.png)

If the consumer’s task is to insert these messages into a database, it might insert message 1 or 2 multiple times, leading to incorrect data.

### Ensuring Idempotency

Duplicate consumption itself isn’t the real issue; the problem arises when you **fail to handle repeated messages in an idempotent manner**.

For example, if your system is inserting a record into a database for every message consumed, a duplicate message will result in two records. But, if the system checks whether the record already exists before inserting it, the duplicate message can be ignored, ensuring the data remains correct.

In this case, although the same message is consumed twice, only one record exists in the database, thereby ensuring idempotency.

Idempotency essentially means that, whether you process the same message once or multiple times, the resulting state of the system **remains consistent and error-free**.

### Strategies to Ensure Idempotency

Ensuring idempotency in message consumption depends on your specific use case, but here are a few strategies:

- If you’re inserting data into a database, query for the record by its primary key first. If it exists, update it instead of inserting a new record.
- If you're working with Redis, you don’t need to worry because `SET` operations are naturally idempotent.
- A more sophisticated approach could involve having the producer include a globally unique ID (like an order ID) with every message. When you consume a message, first check if this ID has already been processed (e.g., by querying Redis). If it has, skip processing. If it hasn’t, process the message and store the ID to avoid future duplicates.
- You can also rely on a **unique key constraint** in your database to ensure that duplicate messages don’t create duplicate records. The database will throw an error when a duplicate is inserted, but no incorrect data will be created.

![mq-11](./images/mq-11.png)

In practice, ensuring idempotency when consuming messages from a message queue depends on the specifics of your system and business logic.
