## Interview Question

Do you know about the rehash process in Redis?

## Interviewer Psychology

This knowledge point is relatively rare in Redis interviews, but when discussing rehashing in HashMap or ConcurrentHashMap, you can proactively mention that you also understand the rehash process in Redis.

Redis is renowned for its speed and performance. We know that Redis starts with a limited capacity, and when capacity is insufficient, it needs to be expanded. What is the expansion method? Does it involve transferring all data at once? With data volumes reaching tens or even hundreds of millions, this would definitely block Redis from executing commands. Therefore, it is crucial to understand the rehash process in Redis.

## Analysis of the Interview Question

As we know, Redis is primarily used for storing key-value pairs, and this storage is implemented using a dictionary. Underlying this dictionary in Redis is a hash table. The nodes in the hash table store the key-value pairs of the dictionary, similar to how Java's HashMap maps keys to hash table node positions using a hash function.

The data structure of the dictionary in Redis is as follows:


```c
// 字典对应的数据结构，有关hash表的结构可以参考redis源码，再次就不进行描述
typedef struct dict {
    dictType *type;  // 字典类型
    void *privdata;  // 私有数据
    dictht ht[2];    // 2个哈希表，这也是进行rehash的重要数据结构，从这也看出字典的底层通过哈希表进行实现。
    long rehashidx;   // rehash过程的重要标志，值为-1表示rehash未进行
    int iterators;   //  当前正在迭代的迭代器数
} dict;
```

When expanding or shrinking the hash table, the program needs to rehash all key-value pairs from the existing hash table into the new hash table. The specific process is as follows:

### 1. Allocate Space for the Backup Hash Table

- **Expansion Operation**: If performing an expansion operation, the size of the backup hash table is the next power of 2 greater than twice the number of key-value pairs in the current hash table (e.g., `5*2=10`, so the backup hash table size would be the next power of 2 greater than 10, which is 16).

- **Shrink Operation**: If performing a shrink operation, the size of the backup hash table is the next power of 2 greater than or equal to the number of key-value pairs in the current hash table (`ht[0].used`).

### 2. Incremental Rehash

The rehash process is not completed all at once, especially with very large data volumes (tens or hundreds of millions). Instead, it is done incrementally. The benefit of incremental rehashing is that it avoids impacting the server.

The essence of incremental rehashing:

- Using `rehashidx`, the computational work required for rehashing key-value pairs is distributed across each add, delete, search, and update operation on the dictionary, avoiding the huge computation load of a centralized rehash.

- During the rehash process, every time an operation is performed on the dictionary (add, delete, search, or update), the program not only executes the specified operation but also rehashes all key-value pairs from the original hash table at the `rehashidx` index into the backup hash table. After rehashing is complete, the program increments the value of the `rehashidx` attribute.
