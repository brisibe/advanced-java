## Interview Questions

Why perform database sharding and table partitioning (when designing a high-concurrency system, how should the database layer be designed)? What sharding and partitioning middleware have you used? What are the advantages and disadvantages of different sharding and partitioning middleware? How specifically do you perform vertical and horizontal sharding or partitioning in your database?

## Interviewer Psychological Analysis

This question is usually related to **high concurrency** issues. Database sharding and table partitioning are crucial for handling **high concurrency and large data volumes**. In internet companies, especially during interviews, this topic often comes up because it is a common technical issue. Not knowing about it can be problematic.

## Interview Question Analysis

### Why Perform Database Sharding and Table Partitioning? (How to Design the Database Layer for High-Concurrency Systems?)

In essence, sharding and partitioning are different concepts. You can perform sharding without partitioning, or vice versa.

Let’s consider a scenario:

Imagine a startup company (or a new department in a rapidly growing company) with 200,000 registered users, 10,000 daily active users, and a peak of 10 requests per second. With such a system, you could easily manage with a few experienced developers and a couple of newly trained ones.

However, if the company grows rapidly, the user count might reach 20 million, with 1 million daily active users, 100,000 rows of data per day, and a peak of 1,000 requests per second. The system might still handle it with a few machines and load balancing.

But if the company continues to grow, reaching 100 million users and 50 million new rows per day, the single table could grow to 20-30 million rows. This will eventually lead to performance degradation and inability to handle the load.

#### Table Partitioning

If a table grows to millions of rows, its performance can degrade significantly. Partitioning involves dividing a single table into multiple tables, each with a manageable amount of data. For example, partition tables by user ID, placing each user's data into separate tables. This keeps individual table sizes under control (e.g., 2 million rows per table).

#### Database Sharding

Sharding involves splitting a single database into multiple databases to handle increased load and storage capacity. Each database will handle a portion of the total data, with each database having the same table structure. Sharding allows handling higher concurrency and provides more storage capacity.

| #            | Before Sharding and Partitioning         | After Sharding and Partitioning                |
| ------------ | ---------------------------------------- | ---------------------------------------------- |
| Concurrency   | Single MySQL instance cannot handle high concurrency | Multiple MySQL instances handle much higher concurrency |
| Disk Usage    | Disk capacity of single MySQL instance is nearly full | Disk usage is distributed across multiple instances |
| SQL Performance | SQL performance degrades with large table sizes | Performance improves with smaller table sizes   |

### What Sharding and Partitioning Middleware Have You Used? What Are the Advantages and Disadvantages of Different Middleware?

Common middleware solutions include:

- **Cobar**: Developed and open-sourced by Alibaba's B2B team, it’s a proxy-layer solution between the application and database servers. Applications connect via JDBC, and Cobar routes SQL queries to different database instances based on sharding rules. It’s outdated and lacks support for certain features like read/write separation and cross-database joins.

- **TDDL**: Developed by Taobao, it’s a client-layer solution. It supports basic CRUD operations and read/write separation but lacks support for joins and multi-table queries. It relies on Taobao's diamond configuration management system.

- **Atlas**: Developed by 360, it’s also a proxy-layer solution. It has not been updated in recent years and is now rarely used.

- **Sharding-JDBC**: Developed by Dangdang, it’s a client-layer solution and part of the [ShardingSphere](https://shardingsphere.apache.org) project. It supports sharding, read/write separation, distributed ID generation, and flexible transactions. It is widely used and actively maintained, making it a viable option.

- **Mycat**: A proxy-layer solution based on Cobar, with extensive features and an active community. It’s gaining popularity but is less mature compared to Sharding-JDBC.

**Summary**: For medium-sized companies, Sharding-JDBC is recommended due to its low operational cost and simplicity. For large companies, Mycat or similar proxy-layer solutions might be better due to their ability to handle complex systems and larger teams.

### How Do You Perform Vertical and Horizontal Sharding or Partitioning?

**Horizontal Sharding** involves dividing a table's data into multiple tables across multiple databases, with each database having an identical schema. The purpose is to distribute the load and storage capacity across multiple databases.

![Horizontal Sharding](./images/database-split-horizon.png)

**Vertical Sharding** involves splitting a table with many columns into multiple tables with different sets of columns. High-frequency access columns are placed in one table, and low-frequency columns in another. This improves performance by caching frequently accessed rows more effectively.

![Vertical Sharding](./images/database-split-vertically.png)

**Table Partitioning** involves splitting a single table into multiple tables to control the size and improve performance. This can be based on different criteria, such as user ID or data range.

There are two main strategies for sharding and partitioning:

- **Range-based Sharding**: Data is divided into ranges, typically by time. This method is simple to implement but can lead to hotspots, with heavy load on recent data.

- **Hash-based Sharding**: Data is distributed based on a hash of a field value, ensuring even distribution. This method is more balanced but requires data migration during scaling.

Choose the appropriate strategy based on your project's needs, system complexity, and growth expectations.
