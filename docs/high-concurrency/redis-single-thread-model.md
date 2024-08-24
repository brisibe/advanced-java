## Interview Questions

What are the differences between Redis and Memcached? What is Redis's threading model? Why can Redis support high concurrency even though it is single-threaded?

## Interviewer’s Perspective

These are fundamental questions when discussing Redis. A basic internal principle and characteristic of Redis is that it operates on a **single-threaded model**. If you don't know this, it will be hard to troubleshoot issues when working with Redis in the future.

The interviewer may also ask about the differences between Redis and Memcached. While Memcached was commonly used by many internet companies in the past, Redis has become the predominant choice in recent years, and Memcached is now rarely used.

## Analysis of the Interview Question

### What are the differences between Redis and Memcached?

#### Redis supports complex data structures

Compared to Memcached, Redis offers [more data structures](#) and supports more complex data operations. If your caching needs involve more complex structures and operations, Redis is a better choice.

#### Redis natively supports cluster mode

Starting with Redis 3.x, it supports cluster mode, whereas Memcached does not have native cluster support and relies on the client to implement data sharding across a cluster.

#### Performance Comparison

Redis uses a **single-core** approach, while Memcached can utilize **multi-core** systems. Therefore, Redis generally performs better than Memcached for small data storage on a per-core basis. However, for data sizes above 100k, Memcached often performs better than Redis. Although Redis has been optimizing performance for large data storage recently, it is still slightly less efficient compared to Memcached.

### Redis’s Threading Model

Redis internally uses a file event handler, which is single-threaded, hence Redis operates on a single-threaded model. It employs an IO multiplexing mechanism to listen to multiple sockets simultaneously, queuing the sockets that generate events into an in-memory queue. The event dispatcher selects the appropriate event handler based on the event type of each socket.

The structure of the file event handler includes four parts:

- Multiple sockets
- IO multiplexing program
- File event dispatcher
- Event handlers (connection acceptance handler, command request handler, command reply handler)

Multiple sockets may concurrently generate different operations, each corresponding to different file events. However, the IO multiplexing program listens to multiple sockets and queues the sockets that generate events. The event dispatcher retrieves each socket from the queue and hands it over to the appropriate event handler based on the event type.

Here is the process of a single communication between a client and Redis:

![Redis-single-thread-model](./images/redis-single-thread-model.png)

Communication is completed through sockets; if you are unfamiliar with this, you may want to review socket network programming.

Initially, when the Redis server process is initialized, it associates the `AE_READABLE` event of the server socket with the connection acceptance handler.

When client socket01 requests to establish a connection with the Redis server socket, the server socket generates an `AE_READABLE` event. The IO multiplexing program detects this event and queues the socket. The file event dispatcher retrieves the socket from the queue and hands it to the **connection acceptance handler**. The connection acceptance handler creates a socket01 for communication with the client and associates the `AE_READABLE` event of socket01 with the command request handler.

If the client sends a `set key value` request, Redis's socket01 will generate an `AE_READABLE` event. The IO multiplexing program queues socket01, and the event dispatcher retrieves it and hands it to the command request handler, as the `AE_READABLE` event of socket01 is already associated with it. The command request handler reads the `key value` from socket01 and performs the `key value` setting in memory. After completing the operation, it associates the `AE_WRITABLE` event of socket01 with the command reply handler.

If the client is ready to receive the result, Redis's socket01 will generate an `AE_WRITABLE` event, which is also queued. The event dispatcher finds the associated command reply handler, which then inputs the result of the operation, such as `ok`, and disassociates the `AE_WRITABLE` event from the command reply handler.

This completes the communication. For a comprehensive understanding of Redis's communication process, readers are encouraged to refer to 《[Redis Design and Implementation by Huang Jianhong](https://github.com/doocs/technical-books#database)》.

### Why is Redis's single-threaded model so efficient?

- It operates purely in memory.
- It uses a non-blocking IO multiplexing mechanism.
- Implemented in C, which is closer to the operating system and generally faster in execution.
- Single-threaded design avoids issues related to frequent context switching and competition problems inherent in multi-threading.

### Introduction of Multi-threading in Redis 6.0

**Note!** Starting with Redis 6.0, Redis has moved away from a purely single-threaded model and **now selectively uses a multi-threaded model**.

Previously emphasizing the efficiency of Redis's single-threaded model, the introduction of multi-threading indicates that single-threading has limitations in some aspects. Network read/write system calls consume a significant amount of CPU time during Redis operations, and multi-threading for network data read/write and protocol parsing can significantly enhance performance.

**The multi-threading in Redis is used only for handling network data read/write and protocol parsing, while command execution remains single-threaded.** This design choice avoids the complexity of managing concurrency issues related to keys, Lua scripts, transactions, LPUSH/LPOP, etc., in a multi-threaded environment.

### Summary

Redis primarily uses a single-threaded model for client request handling because CPU is not a bottleneck for Redis servers. The performance gains from multi-threading do not outweigh the associated development and maintenance costs. The system's performance bottleneck mainly lies in network I/O operations. Redis’s introduction of multi-threading is aimed at improving performance for large data operations and non-blocking memory release without blocking network I/O read/write, thereby enhancing overall efficiency.
