## Interview Question

How is caching used in your project? Why use caching? What are the potential issues if caching is misused?

## Interviewer’s Perspective

This is a common question in tech companies. If someone isn't familiar with caching, it can be quite awkward.

When the topic of caching comes up, the first question is usually about where caching is used in your project. Why use it? What happens if you don’t use it? What are the possible downsides if you do use it?

This is to see if you have thought about the reasoning behind using caching. If you’re just using it blindly without being able to give a reasonable explanation, the interviewer might get the impression that you don’t think critically and just follow instructions without understanding.

## Analysis of the Interview Question

### How is caching used in your project?

This should be answered by referring to your project’s specific business needs.

### Why use caching?

Caching is mainly used for two purposes: **High Performance** and **High Concurrency**.

#### High Performance

Consider this scenario: you have an operation where a request comes in, and after various complex operations with MySQL, you get a result, which takes 600ms. However, this result might not change for the next few hours, or even if it does, it’s not crucial to immediately update the user. So, what should you do?

Cache it! The result that took 600ms to retrieve can be stored in the cache, where a key corresponds to a value. The next time someone queries it, instead of going through MySQL and taking 600ms, the cache can return the value in 2ms. This improves performance by 300 times.

So, for results that are complex to retrieve and don’t change often but are frequently requested, you can store them in the cache and read from there afterward.

#### High Concurrency

MySQL, being a heavy database, is not designed for high concurrency, although it can handle it to some extent, but not natively well. A single MySQL instance might start to struggle when it reaches around `2000QPS`.

If your system experiences peak traffic of 10,000 requests per second, a single MySQL instance will definitely crash. At this point, you should use caching. Store a lot of data in the cache instead of in MySQL. The cache is simple in functionality, essentially operating as `key-value` pairs, and a single instance can easily handle tens or even hundreds of thousands of requests per second. Supporting high concurrency is easy. The concurrent capacity of a single cache instance is dozens of times that of a single MySQL instance.

> Caching operates in memory, which is naturally suited for high concurrency.

### What are the potential issues after using caching?

Common cache-related issues include:

-   [Inconsistency between cache and database](/docs/high-concurrency/redis-consistence.md)
-   [Cache avalanche, cache penetration, cache breakdown](/docs/high-concurrency/redis-caching-avalanche-and-caching-penetration.md)
-   [Cache concurrency issues](/docs/high-concurrency/redis-cas.md)

Click on the hyperlinks to directly view the related problems and solutions for caching.
