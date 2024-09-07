## Interview Questions

For a system that is currently not sharded or partitioned but needs to be migrated to a sharded and partitioned system in the future, how would you design the migration process to allow a **dynamic switch** from a single database to sharded databases?

## Interviewer Psychological Analysis

Now that you understand the reasons for database sharding and partitioning, the commonly used middleware, and how to design your sharding and partitioning strategy (horizontal and vertical partitioning, table partitioning), the next step is to consider how to migrate an existing single-database system to a sharded system.

This question is about evaluating whether you have experience with the entire migration process from a single-database to a sharded system.

## Interview Question Analysis

There are several ways to approach this migration, ranging from basic to more advanced strategies. Here are some common methods:

### Downtime Migration Strategy

A basic approach involves downtime for migration. Here's a simple method:

1. Announce the maintenance window in advance, such as from midnight to 6 AM.
2. At midnight, stop the system to halt all traffic and data writes. The old single-database system will be static during this period.
3. Use a pre-written **data migration tool** to read data from the old database and write it to the new sharded database.
4. After migration, update the system's database connection configuration, including any necessary changes to the code and SQL queries.
5. Restart the system with the new configuration and verify that everything works as expected.

This method is straightforward but involves downtime, which might not be acceptable for many systems.

![database-shard-method-1](./images/database-shard-method-1.png)

### Dual-Write Migration Strategy

A more advanced and commonly used method involves no downtime. Here's how it works:

1. **Dual Writing**: Modify all write operations in the system to update both the old and the new databases. This means that every insert, update, and delete operation will be executed on both the old and new databases simultaneously.

2. **Initial Data Migration**: Use a data migration tool to read data from the old database and write it to the new database. During this process, ensure that data is only written to the new database if it is newer or if it is missing from the new database. This prevents older data from overwriting newer data.

3. **Data Consistency Check**: After the initial migration, there may be data inconsistencies. Implement an automatic verification process to compare data between the old and new databases. For any discrepancies, re-read data from the old database and update the new database. Continue this process until all data is consistent.

4. **Final Switch**: Once data consistency is confirmed, redeploy the system using only the new sharded database configuration. This approach ensures minimal downtime, often just a few hours, and is reliable for maintaining data integrity.

![database-shard-method-2](./images/database-shard-method-2.png)
