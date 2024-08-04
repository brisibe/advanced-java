# Advanced Knowledge for Internet Java Engineers

[![stars](https://img.shields.io/github/stars/doocs/advanced-java?color=42b883&logo=github&style=flat-square&logoColor=ffffff)](https://github.com/doocs/advanced-java/stargazers)
[![forks](https://img.shields.io/github/forks/doocs/advanced-java?color=42b883&logo=github&style=flat-square&logoColor=ffffff)](https://github.com/doocs/advanced-java/network/members)
[![license](https://img.shields.io/github/license/doocs/advanced-java?color=42b883&style=flat-square&logo=homeassistantcommunitystore&logoColor=ffffff)](./LICENSE)
[![doocs](https://img.shields.io/badge/org-join%20us-42b883?style=flat-square&logo=homeassistantcommunitystore&logoColor=ffffff)](https://doocs.github.io/#/?id=how-to-join)

Most of the content in this project comes from Zhonghua Shishan, and the copyright belongs to the author. The content covers areas such as [high concurrency](#high-concurrency-architecture), [distributed systems](#distributed-system), [high availability](#high-availability-architecture), [microservices](#microservices-architecture), and [mass data processing](#mass-data-processing). We have systematically organized this knowledge for readers to learn and refer to conveniently.

We are also updating the algorithm project! If you are preparing for an interview with algorithm questions or want to further improve your coding skills, feel free to Star and follow [doocs/leetcode](https://github.com/doocs/leetcode).

Before diving into this project, check out what technical interviewers say in the [Discussions forum](https://github.com/doocs/advanced-java/discussions/9). This project welcomes developers to share their thoughts and practical experiences in the Discussions forum. You can also Star and follow [doocs/advanced-java](https://github.com/doocs/advanced-java) to stay updated on the latest developments.

## High Concurrency Architecture

### [Message Queue](/docs/high-concurrency/mq-interview.md)

-   [Why use a message queue? What are the advantages and disadvantages of message queues? What are the advantages and disadvantages of Kafka, ActiveMQ, RabbitMQ, and RocketMQ?](/docs/high-concurrency/why-mq.md)
-   [How to ensure the high availability of message queues?](/docs/high-concurrency/how-to-ensure-high-availability-of-message-queues.md)
-   [How to ensure that messages are not consumed repeatedly? (How to ensure the idempotency of message consumption)](/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md)
-   [How to ensure reliable message transmission? (How to handle message loss)](/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)
-   [How to ensure the order of messages?](/docs/high-concurrency/how-to-ensure-the-order-of-messages.md)
-   [How to deal with message queue delays and expiration issues? How to handle full message queues? How to resolve the issue of millions of messages being backlogged for hours?](/docs/high-concurrency/mq-time-delay-and-expired-failure.md)
-   [If you were to design a message queue, how would you architect it? Share your thoughts.](/docs/high-concurrency/mq-design.md)

### [Search Engine](/docs/high-concurrency/es-introduction.md)

-   [Can you explain the distributed architecture principles of Elasticsearch (How does ES implement distribution)?](/docs/high-concurrency/es-architecture.md)
-   [What is the working principle of data writing in ES? What is the working principle of data querying in ES? Can you introduce the underlying Lucene? Do you understand inverted indexing?](/docs/high-concurrency/es-write-query-search.md)
-   [How to improve query efficiency in ES when dealing with large amounts of data (tens of billions)?](/docs/high-concurrency/es-optimizing-query-performance.md)
-   [What is the deployment architecture of an ES production cluster? How much data does each index roughly hold? How many shards does each index have?](/docs/high-concurrency/es-production-cluster.md)

### Cache

-   [How is caching used in projects? What are the consequences of improper cache usage?](/docs/high-concurrency/why-cache.md)
-   [What are the differences between Redis and Memcached? What is the thread model of Redis? Why is single-threaded Redis much more efficient than multi-threaded Memcached?](/docs/high-concurrency/redis-single-thread-model.md)
-   [What are the data types of Redis? In which scenarios are they suitable for use?](/docs/high-concurrency/redis-data-types.md)
-   [What are the expiration policies of Redis? Can you handwrite the LRU implementation?](/docs/high-concurrency/redis-expiration-policies-and-lru.md)
-   [How to ensure high concurrency and high availability in Redis? Can you introduce the master-slave replication principle of Redis? Can you introduce the sentinel principle of Redis?](/docs/high-concurrency/how-to-ensure-high-concurrency-and-high-availability-of-redis.md)
-   [What is the master-slave architecture of Redis?](/docs/high-concurrency/redis-master-slave.md)
-   [What are the different persistence methods in Redis? What are the advantages and disadvantages of each? How are they implemented at the underlying level?](/docs/high-concurrency/redis-persistence.md)
-   [Can you explain the working principle of Redis cluster mode? How is key addressing handled in cluster mode? What are the distributed addressing algorithms? Do you know about the consistent hashing algorithm? How to dynamically add and delete nodes?](/docs/high-concurrency/redis-cluster.md)
-   [Do you understand what Redis avalanche, penetration, and breakdown are? What happens after Redis crashes? How should the system handle this? How to handle Redis penetration?](/docs/high-concurrency/redis-caching-avalanche-and-caching-penetration.md)
-   [How to ensure consistency between cache and database for double writes?](/docs/high-concurrency/redis-consistence.md)
-   [What is the concurrency competition problem in Redis? How to solve this problem? Do you understand the CAS solution of Redis transactions?](/docs/high-concurrency/redis-cas.md)
-   [How is Redis deployed in a production environment?](/docs/high-concurrency/redis-production-environment.md)
-   [Do you know about the Redis rehash process?](/docs/high-concurrency/redis-rehash.md)

### Database Sharding

-   [Why is database sharding needed (How should the database layer be designed when designing high-concurrency systems)? What database sharding middleware have you used? What are the advantages and disadvantages of different database sharding middleware? How do you perform vertical or horizontal sharding of databases?](/docs/high-concurrency/database-shard.md)
-   [How would you design a system to dynamically switch from no sharding to sharding in the future?](/docs/high-concurrency/database-shard-method.md)
-   [How to design a database sharding solution that can dynamically scale in and out?](/docs/high-concurrency/database-shard-dynamic-expand.md)
-   [How to handle primary keys after database sharding?](/docs/high-concurrency/database-shard-global-id-generate.md)

### Read-Write Separation

-   [How to implement MySQL read-write separation? What is the principle of MySQL master-slave replication? How to solve the delay problem of MySQL master-slave synchronization?](/docs/high-concurrency/mysql-read-write-separation.md)

### High-Concurrency System

-   [How to design a high-concurrency system?](/docs/high-concurrency/high-concurrency-design.md)

## Distributed System

### [Interview Questions](/docs/distributed-system/distributed-system-interview.md)

### System Splitting

-   [Why perform system splitting? How to perform system splitting? Can you do without Dubbo after splitting?](/docs/distributed-system/why-dubbo.md)

### Distributed Service Framework

-   [Can you explain the working principle of Dubbo? Can communication continue if the registry crashes?](/docs/distributed-system/dubbo-operating-principle.md)
-   [What serialization protocols does Dubbo support? Can you explain the data structure of Hessian? Do you know PB? Why is PB the most efficient?](/docs/distributed-system/dubbo-serialization-protocol.md)
-   [What are the load balancing strategies and cluster fault tolerance strategies of Dubbo? What about dynamic proxy strategies?](/docs/distributed-system/dubbo-load-balancing.md)
-   [What is the spi thought of Dubbo?](/docs/distributed-system/dubbo-spi.md)
-   [How to perform service governance, service degradation, retry on failure, and retry on timeout based on Dubbo?](/docs/distributed-system/dubbo-service-management.md)
-   [How to design idempotent distributed service interfaces (e.g., to prevent repeated deductions)?](/docs/distributed-system/distributed-system-idempotency.md)
-   [How to ensure the order of distributed service interface requests?](/docs/distributed-system/distributed-system-request-sequence.md)
-   [How would you design a similar RPC framework to Dubbo?](/docs/distributed-system/dubbo-rpc-design.md)
-   [What does the P in the CAP theorem stand for?](/docs/distributed-system/distributed-system-cap.md)

### Distributed Lock

-   [What are the application scenarios of Zookeeper?](/docs/distributed-system/zookeeper-application-scenarios.md)
-   [How to design a distributed lock using Redis? Can you design a distributed lock using Zookeeper? Which implementation method is more efficient?](/docs/distributed-system/distributed-lock-redis-vs-zookeeper.md)

### Distributed Transactions

-   [Do you understand distributed transactions? How do you solve the distributed transaction problem? What if TCC encounters network disconnection? How to ensure consistency with XA?](/docs/distributed-system/distributed-transaction.md)

### Distributed Session

-   [How to implement distributed sessions in cluster deployment?](/docs/distributed-system/distributed-session.md)

## High Availability Architecture

-   [Introduction to Hystrix](/docs/high-availability/hystrix-introduction.md)
-   [E-commerce Website Detail Page System Architecture](/docs/high-availability/e-commerce-website-detail-page-architecture.md)
-   [Implementing Resource Isolation with Hystrix Thread Pool](/docs/high-availability/hystrix-thread-pool-isolation.md)
-   [Implementing Resource Isolation with Hystrix Semaphore Mechanism](/docs/high-availability/hystrix-semphore-isolation.md)
-   [Fine-grained Control of Hystrix Isolation Strategies](/docs/high-availability/hystrix-execution-isolation.md)
-   [In-depth Principles of Hystrix Execution](/docs/high-availability/hystrix-process.md)
-   [Optimizing Batch Data Query Interface with Request Cache Technology](/docs/high-availability/hystrix-request-cache.md)
-   [Fallback Mechanism Based on Local Cache](/docs/high-availability/hystrix-fallback.md)
-   [In-depth Principles of Hystrix Circuit Breaker](/docs/high-availability/hystrix-circuit-breaker.md)
-   [In-depth Thread Pool Isolation and Current Limiting with Hystrix](/docs/high-availability/hystrix-thread-pool-current-limiting.md)
-   [Timeout Mechanism for Secure Protection of Service Interface Calls](/docs/high-availability/hystrix-timeout.md)

### High Availability System

-   How to design a high availability system?

### Rate Limiting

-   [How to implement rate limiting? How is it done in practice? Explain the specific implementation.](/docs/high-concurrency/how-to-limit-current.md)

### Circuit Breaking

-   How to implement circuit breaking?
-   What circuit breaking frameworks are available? Do you know the specific implementation principles?
-   [How to choose a circuit breaking framework? Should you use Sentinel or Hystrix?](/docs/high-availability/sentinel-vs-hystrix.md)

### Degradation

-   How to implement degradation?

## Microservices Architecture

-   [The entire chapter on microservices architecture is an additional update. It will be updated later, and readers are welcome to contribute and improve it](https://github.com/doocs/advanced-java).
-   [Introduction to Microservices Architecture](/docs/micro-services/microservices-introduction.md)
-   [Migrating from a Monolithic Architecture to a Microservices Architecture](/docs/micro-services/migrating-from-a-monolithic-architecture-to-a-microservices-architecture.md)
-   [Event-Driven Data Management for Microservices](/docs/micro-services/event-driven-data-management-for-microservices.md)
-   [Choosing a Microservices Deployment Strategy](/docs/micro-services/choose-microservice-deployment-strategy.md)
-   [Advantages and Disadvantages of Microservices Architecture](/docs/micro-services/advantages-and-disadvantages-of-microservice.md)

### Spring Cloud Microservices Architecture

-   [What is a microservice? How do microservices communicate independently?](/docs/micro-services/what's-microservice-how-to-communicate.md)
-   What are the differences between Spring Cloud and Dubbo?
-   Talk about your understanding of Spring Boot and Spring Cloud.
-   What is service circuit breaking? What is service degradation?
-   What are the pros and cons of microservices? Talk about the pitfalls you encountered during project development.
-   [What microservices technology stacks do you know?](/docs/micro-services/micro-services-technology-stack.md)
-   [Microservices Governance Strategies](/docs/micro-services/micro-service-governance.md)
-   What are the differences between Eureka and Zookeeper in providing service registration and discovery?
-   [Talk about the main invocation process of Eureka for service discovery and registration?](/docs/micro-services/how-eureka-enable-service-discovery-and-service-registration.md)
-   ......

## Mass Data Processing

-   [How to find the same URLs from a large number of URLs?](/docs/big-data/find-common-urls.md)
-   [How to find high-frequency words from a large amount of data?](/docs/big-data/find-top-100-words.md)
-   [How to find the IP that visited Baidu the most on a specific day?](/docs/big-data/find-top-1-ip.md)
-   [How to find non-repeating integers from a large amount of data?](/docs/big-data/find-no-repeat-number.md)
-   [How to determine if a number exists in a large amount of data?](/docs/big-data/find-a-number-if-exists.md)
-   [How to query the hottest query strings?](/docs/big-data/find-hotest-query-string.md)
-   [How to count the number of different phone numbers?](/docs/big-data/count-different-phone-numbers.md)
-   [How to find the median from 500 million numbers?](/docs/big-data/find-mid-value-in-500-millions.md)
-   [How to sort query strings by frequency?](/docs/big-data/sort-the-query-strings-by-counts.md)
-   [How to find the top 500 numbers?](/docs/big-data/find-rank-top-500-numbers.md)
-   [Talk about common solutions to TopK problems in big data?](/docs/big-data/topk-problems-and-solutions.md)

## Stars Trend

<a href="https://github.com/doocs/advanced-java/stargazers" target="_blank"><img src="./images/starcharts.svg" alt="Stargazers over time" /></a>

Note: This trend chart is automatically refreshed by [actions-starcharts](https://github.com/MaoLongLong/actions-starcharts), author [@MaoLongLong](https://github.com/maolonglong).

---

## Doocs Community Quality Projects

The Doocs technical community is committed to building a comprehensive and continuously growing learning ecosystem for internet developers! Below are some excellent projects under Doocs, and developers are welcome to keep following.

| #   | Project                                                           | Description                                                                                       | Popularity                                                                                                                      |
| --- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 1   | [advanced-java](https://github.com/doocs/advanced-java)           | Comprehensive knowledge for advanced internet Java engineers: covers high concurrency, distributed systems, high availability, microservices, and mass data processing. | ![](https://badgen.net/github/stars/doocs/advanced-java) <br>![](https://badgen.net/github/forks/doocs/advanced-java)           |
| 2   | [leetcode](https://github.com/doocs/leetcode)                     | LeetCode solutions in multiple programming languages, including solutions for "Sword Offer (2nd Edition)" and "Cracking the Coding Interview (6th Edition)".        | ![](https://badgen.net/github/stars/doocs/leetcode) <br>![](https://badgen.net/github/forks/doocs/leetcode)                     |
| 3   | [source-code-hunter](https://github.com/doocs/source-code-hunter) | Analysis of source code of commonly used internet components and frameworks.                                                           | ![](https://badgen.net/github/stars/doocs/source-code-hunter) <br>![](https://badgen.net/github/forks/doocs/source-code-hunter) |
| 4   | [jvm](https://github.com/doocs/jvm)                               | Summary of knowledge on the underlying principles of the Java Virtual Machine.                                                         | ![](https://badgen.net/github/stars/doocs/jvm) <br>![](https://badgen.net/github/forks/doocs/jvm)                               |
| 5   | [coding-interview](https://github.com/doocs/coding-interview)     | Collection of coding interview questions, including "Sword Offer" and "Beauty of Programming".                                        | ![](https://badgen.net/github/stars/doocs/coding-interview) <br>![](https://badgen.net/github/forks/doocs/coding-interview)     |
| 6   | [md](https://github.com/doocs/md)                                 | A highly concise WeChat Markdown editor.                                                                                               | ![](https://badgen.net/github/stars/doocs/md) <br>![](https://badgen.net/github/forks/doocs/md)                                 |
| 7   | [technical-books](https://github.com/doocs/technical-books)       | List of technical books worth reading.                                                                                                 | ![](https://badgen.net/github/stars/doocs/technical-books) <br>![](https://badgen.net/github/forks/doocs/technical-books)       |

## Contributors

Thanks to all the friends who have contributed to the [Doocs Technical Community](https://github.com/doocs). [Click here to join the project maintenance](https://doocs.github.io/#/?id=how-to-join).

<!-- ALL-CONTRIBUTORS-LIST: START -->
<a href="https://opencollective.com/doocs/contributors.svg?width=890&button=true"><img src="https://opencollective.com/doocs/contributors.svg?width=890&button=false" /></a>
<!-- ALL-CONTRIBUTORS-LIST: END -->

## Official Account

The only official WeChat account of the [Doocs](https://github.com/doocs) technical community, "Doocs". Welcome to scan the code to follow, **focused on sharing knowledge related to the technical field and the latest industry information**. Of course, you can also add my personal WeChat (note: GitHub) to join the technical exchange group.

<table>
  <tr>
    <td align="center" style="width: 260px;">
      <img src="https://cdn-doocs.oss-cn-shenzhen.aliyuncs.com/gh/doocs/images/qrcode-for-doocs.png" style="width: 400px;"><br>
    </td>
    <td align="center" style="width: 260px;">
      <img src="https://cdn-doocs.oss-cn-shenzhen.aliyuncs.com/gh/doocs/images/qrcode-for-yanglbme.png" style="width: 400px;"><br>
    </td>
  </tr>
</table>

Follow the "Doocs" official account and reply with **PDF** to get the offline PDF document of this project (283 pages of essence) for more convenient learning!

<img src="./images/pdf.png" style="width: 600px;"><br>
