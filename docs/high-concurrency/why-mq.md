## Interview Questions

-   Why use a message queue?
-   What are the advantages and disadvantages of message queues?
-   What are the differences between Kafka, ActiveMQ, RabbitMQ, and RocketMQ, and which scenarios are they suitable for?

## Interviewer’s Perspective

The interviewer is mainly interested in:

-   **First**, do you understand why your system uses a message queue?

    Many candidates mention using Redis or MQ in their projects, but they don’t really know why they’re using it. Essentially, they’re using it just for the sake of using it, or because someone else designed the architecture, and they’ve never really thought about it.

    Candidates who haven’t questioned their architecture are likely not thinking critically, and interviewers generally don’t have a good impression of such candidates. The concern is that once you’re in the team, you might just do things mechanically without thinking for yourself.

-   **Second**, since you’ve used a message queue, do you know what the pros and cons are?

    If you haven’t considered this, and you blindly introduce MQ into the system, what will happen if problems arise? Will you just leave and create a mess for the company? If you haven’t considered the potential drawbacks and risks of introducing a technology, the interviewer might worry that hiring you would be like hiring someone who digs holes that others have to fill after you leave.

-   **Third**, if you’ve used MQ, did you research which one to use?

    Don’t just choose an MQ based on personal preference without research, like Kafka, without ever looking into what other MQs are available. Each MQ has its strengths and weaknesses. There’s no absolutely good or bad choice; it’s all about using the right tool for the right scenario, leveraging its strengths and avoiding its weaknesses.

    If a candidate doesn’t consider technology choices carefully and is brought into the team, the leader might assign them a task to design a system, and they might use a technology that isn’t necessarily suitable, which could create problems.

## Analysis of Interview Questions

### Why Use a Message Queue

This question is really about asking you to describe the scenarios where message queues are used and the specific scenario in your project. The interviewer expects you to explain a business scenario in your company, the technical challenges it posed, and how using MQ provided benefits.

Let’s start with some common scenarios where message queues are used: **decoupling**, **asynchronous processing**, and **peak shaving**.

#### Decoupling

Consider this scenario: System A sends data to Systems B, C, and D via API calls. What if System E also needs this data? What if System D no longer needs it? The person in charge of System A is close to breaking down...

![mq-1](./images/mq-1.png)

In this scenario, System A is tightly coupled with various other systems. System A generates some critical data that many systems need. System A has to constantly consider what to do if Systems B, C, D, and E go down: should it resend the data, or store the messages? It’s a huge headache!

By using MQ, System A sends data to MQ, and whichever system needs the data can consume it from MQ. If a new system needs the data, it can consume it directly from MQ; if a system no longer needs the data, it can stop consuming messages from MQ. This way, System A doesn’t need to worry about who to send the data to, maintaining the code, or handling scenarios where the other systems fail or time out.

![mq-2](./images/mq-2.png)

**Summary**: By using MQ and a Pub/Sub model, System A is completely decoupled from the other systems.

**Interview Tip**: Consider whether your system has similar scenarios where a system or module interacts with multiple other systems or modules, making the interactions complex and hard to maintain. If the interaction doesn’t need to be synchronous, you could use MQ to decouple the systems. Reflect this in your resume, showing how you used MQ for decoupling.

#### Asynchronous Processing

Let’s look at another scenario: System A receives a request, writes to its local database, and also writes to the databases of Systems B, C, and D. Writing to the local database takes 3ms, while writing to the databases of Systems B, C, and D takes 300ms, 450ms, and 200ms, respectively. The total latency is 3 + 300 + 450 + 200 = 953ms, nearly 1 second. The user perceives this as extremely slow. Waiting for 1 second after clicking a button is almost unacceptable.

![mq-3](./images/mq-3.png)

In internet companies, the general requirement is that every user operation should be completed within 200ms, so the user doesn’t notice any delay.

If you **use MQ**, System A could send three messages to MQ in 5ms. From receiving the request to responding to the user, System A would take 3 + 5 = 8ms. The user would feel like the system is extremely fast.

![mq-4](./images/mq-4.png)

#### Peak Shaving

From 00:00 to 12:00, System A handles about 50 concurrent requests per second. But from 12:00 to 13:00, the number of concurrent requests suddenly spikes to over 5,000 per second. Since the system is based on MySQL, a large number of requests hit the database, executing about 5,000 SQL queries per second.

A typical MySQL database can handle up to 2,000 requests per second. If it suddenly handles 5,000 requests per second, it could crash, bringing the system down, and users wouldn’t be able to use it.

However, after the peak hour, in the afternoon, the load drops, with only about 50 requests per second, posing almost no stress on the system.

![mq-5](./images/mq-5.png)

By using MQ, 5,000 requests per second can be written into MQ, and System A can handle up to 2,000 requests per second, as that’s the maximum MySQL can handle. System A would slowly pull requests from MQ, processing up to 2,000 per second, ensuring the system remains stable even during peak hours. During the peak hour, millions of requests might pile up in MQ.

![mq-6](./images/mq-6.png)

This temporary backlog during peak hours is fine because after the peak, System A can still process 2,000 requests per second, quickly clearing the backlog.

### What Are the Advantages and Disadvantages of Message Queues?

The advantages have been discussed: **decoupling**, **asynchronous processing**, and **peak shaving** in specific scenarios.

The disadvantages are as follows:

-   **Reduced system availability**

    The more external dependencies you introduce, the more likely the system is to fail. Initially, System A just needed to call the APIs of Systems B, C, and D, and everything was fine. But now, you’ve introduced MQ; what happens if MQ goes down? If MQ fails, the whole system could crash.

-   **Increased system complexity**

    By adding MQ, how do you [ensure messages aren’t consumed twice](/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md)? How do you handle [message loss](/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)? How do you ensure the order of message delivery? There are so many problems to deal with, making it very challenging.

-   **Consistency issues**

    System A might process a request and return success, but what if Systems B and D succeed, and System C fails? Your data would be inconsistent.

    Message queues introduce complexity into the architecture. They offer many benefits but also require additional technical solutions to mitigate their drawbacks, making the system much more complex—perhaps ten times more complex. However, when necessary, they are indispensable.

### What Are the Advantages and Disadvantages of Kafka, ActiveMQ, RabbitMQ, and RocketMQ?

| Feature                  | ActiveMQ                              | RabbitMQ                                           | RocketMQ                                                                                                              | Kafka                                                                                                                                           |
| ------------------------ | ------------------------------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Single-machine throughput | Tens of thousands, lower than RocketMQ and Kafka | Same as ActiveMQ                                    | Hundreds of thousands, supports high throughput                                                                                       | Hundreds of thousands, used for real-time data processing and log collection in big data systems                                               |
| Impact of topic count on throughput |                                      |                                                    | Topics can reach hundreds or thousands with only a small decrease in throughput, a major advantage of RocketMQ          | Throughput drops significantly when topic count exceeds dozens to hundreds. Kafka requires more resources if many topics are needed.           |
| Latency                  | ms-level                              | Microsecond-level, a key feature of RabbitMQ, with minimal latency | ms-level                                                                                                               | Under ms-level latency                                                                                                                            |
| Availability             | High, achieved through a master-slave architecture | Same as ActiveMQ                                    | Very high, distributed architecture                                                                                                   | Very high, distributed, data is replicated multiple times, and a few machines going down won’t cause data loss or unavailability                |
| Message reliability      | Low probability of data loss          | Almost no data loss                                 | Can be configured to achieve zero data loss                                                                                           | Same as RocketMQ                                                                                                                                 |
| Feature support          | Extremely comprehensive in the MQ field | Developed in Erlang, strong concurrency, excellent performance, low latency | Fairly comprehensive MQ features, distributed, good scalability                                                              | Simple functionality, mainly supports basic MQ features, widely used in real-time computing and log collection in big data domains.             |

Based on the table above, when selecting an MQ:

-   If you need a high throughput MQ, consider Kafka or RocketMQ.
-   If your MQ requires strong functionality support, including scenarios like transactions, delayed messages, and scheduling, RocketMQ is a good choice.
-   If latency is critical, RabbitMQ should be considered.
-   If you’re looking for an easy-to-learn, low-barrier MQ for a project that doesn’t require high throughput or a distributed nature, ActiveMQ is a good option.
