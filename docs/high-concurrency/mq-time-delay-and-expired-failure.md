## Interview Question

How do you resolve message queue delays and expiration issues? What should you do if the message queue is full? How would you handle millions of messages piling up for several hours?

## Interviewer’s Psychological Analysis

The essence of this question targets scenarios where there might be issues on the consumer side, either due to lack of consumption or extremely slow processing. The problem becomes critical if your message queue cluster’s disks are nearing full capacity with no consumption happening. What would you do if the backlog lasted for several hours or if the messages expired due to RabbitMQ’s TTL settings?

This situation is quite common in production environments and can lead to major issues. A typical example is when the consumer, which writes to MySQL after each consumption, fails, causing the consumer to hang and stop working, or if the consumer encounters issues that slow down processing.

## Question Breakdown

Let’s break this down step-by-step. Assume a scenario where the consumer is down, and a large volume of messages is piling up in the MQ.

### Large Backlog in MQ for Several Hours

Imagine tens of millions of messages have been stuck in the MQ for seven to eight hours, from around 4 PM to after 11 PM. This is a real scenario we encountered, where a production issue led to such a backlog. To resolve this:

1. **Fix the Consumer Issue**: Ensure the consumer is repaired and able to consume messages at an appropriate speed. Simply waiting for hours for the consumer to catch up is not ideal.
   
2. **Temporarily Expand Resources**:
   - Create a new topic with 10 times the original number of partitions and queues.
   - Develop a temporary consumer program to handle the backlog without doing time-consuming processing. Instead, it should evenly distribute messages into the temporary queues.
   - Deploy 10 times the usual number of consumer machines to process messages from these temporary queues. This effectively increases both queue and consumer resources by 10 times, allowing data to be processed at a faster rate.
   - Once the backlog is cleared, revert to the original architecture and consumer machines.

### Messages Expired in MQ

Assuming you use RabbitMQ, which allows setting TTL (Time-To-Live) for messages. If messages exceed their TTL in the queue, they will be purged by RabbitMQ. This results in data loss rather than backlog.

In this case, since the messages are lost, you need to focus on **batch re-importation**. Here’s how:
- After the peak period (e.g., after midnight when traffic is lower), write a program to recover lost data and reinsert it into the MQ.
- For instance, if 10,000 orders were in the MQ and 1,000 were lost, manually write a program to retrieve those 1,000 lost orders and republish them to the MQ.

### MQ Nearly Full

If the MQ is almost full due to a backlog, what can be done? If the initial solution is too slow, implement a temporary program to discard or quickly process messages:
- **Consume and Discard**: Quickly consume and discard messages to free up space. Then, implement the second solution to re-import data once the system is stable.

---

For RocketMQ, the official solution to message backlog issues includes:

### 1. Increase Consumption Parallelism

Most message consumption is IO-intensive (e.g., database operations or RPC calls). To increase throughput:
- **Increase Consumer Instances**: Add more consumer instances within the same ConsumerGroup. Ensure you don’t exceed the number of subscribed queues.
- **Increase Threads per Consumer**: Adjust parameters like `consumeThreadMin` and `consumeThreadMax` to increase parallel threads in a single consumer.

### 2. Batch Consumption

If the business process supports batch consumption, it can significantly improve throughput. For instance, processing 10 orders at once might take only slightly more time than processing one order, thereby increasing throughput. Set the `consumeMessageBatchMaxSize` parameter to batch messages.

### 3. Skip Non-Essential Messages

If backlog exceeds consumption speed and data importance is low, consider discarding less important messages:
```java
public ConsumeConcurrentlyStatus consumeMessage(
            List<MessageExt> msgs,
            ConsumeConcurrentlyContext context) {
    long offset = msgs.get(0).getQueueOffset();
    String maxOffset =
            msgs.get(0).getProperty(Message.PROPERTY_MAX_OFFSET);
    long diff = Long.parseLong(maxOffset) - offset;
    if (diff > 100000) {
        // TODO: Special handling for message backlog
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
    // TODO: Normal consumption process
    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
}
4. Optimize Each Message Consumption Process
For example, a message consumption process might involve:

Querying data from the database multiple times.
Performing complex business logic.
Inserting results back into the database.
If each database interaction takes 5ms, and there are 4 interactions, total time could be 20ms plus 5ms for business logic, totaling 25ms. Reducing database interactions to 2 can optimize total time to 15ms, improving performance by 40%. Using SSDs instead of SCSI disks can also reduce response time significantly.

