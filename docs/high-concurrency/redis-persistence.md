## Interview Questions

What are the different persistence mechanisms in Redis? What are the advantages and disadvantages of each persistence mechanism? How are these persistence mechanisms implemented at a low level?

## Interviewer’s Psychological Analysis

If Redis only caches data in memory, all data will be lost if Redis crashes and is restarted. You need to use Redis's persistence mechanisms to asynchronously write data to disk files while keeping it in memory, ensuring data persistence.

If Redis crashes and restarts, it can automatically load previously persisted data from disk. While there might be some data loss, it prevents the loss of all data.

These questions address potential issues in a production environment, such as ensuring Redis remains available and recoverable after a crash.

## Breakdown of Interview Questions

Persistence is primarily about disaster recovery and data recovery, and can be considered part of high availability. For example, if Redis crashes, you want it to become available as quickly as possible. Without data backups, Redis would be unusable if restarted, as all data would be lost.

This could lead to scenarios where all cached data is missing, causing a **cache avalanche**—all requests fall back to the database, potentially overwhelming and crashing it.

By implementing robust persistence and backup strategies, you can quickly restore Redis and continue providing service even after a failure.

### Redis Persistence Mechanisms

- **RDB (Redis Database)**: RDB persistence involves periodically creating snapshots of the data in Redis.
- **AOF (Append-Only File)**: AOF logs every write operation in an append-only file. On restart, Redis can reconstruct the dataset by replaying the AOF log.

Both RDB and AOF can be used to persist Redis data to disk and back it up to other locations, such as cloud services. In case of a failure, data can be restored from these backups, and Redis can be restarted to recover the in-memory data.

If both RDB and AOF are used, Redis will prefer **AOF** for recovery during restart, as AOF provides a more complete dataset.

#### RDB Advantages and Disadvantages

- **Advantages**:
  - RDB creates multiple data files representing the state of Redis at various points in time, which is ideal for **cold backups**. These files can be stored securely on remote storage services, like Amazon S3 or Alibaba Cloud's ODPS.
  - RDB has minimal impact on Redis's read/write performance as it only requires the main process to fork a child process for disk I/O operations.
  - Recovery from RDB files is generally faster compared to AOF.

- **Disadvantages**:
  - RDB does not perform well for scenarios where minimal data loss is crucial, as snapshots are typically created every 5 minutes or longer.
  - During large RDB snapshot creation, the process may cause a brief pause in service, potentially lasting milliseconds to seconds.

#### AOF Advantages and Disadvantages

- **Advantages**:
  - AOF provides better data protection by performing `fsync` operations every second, minimizing data loss to at most one second.
  - AOF's append-only mode has high write performance and is less prone to file corruption.
  - AOF can be used for **disaster recovery** by manually editing the log file to remove erroneous commands, such as `flushall`.
  - Even large AOF files have minimal impact on client performance due to background log rewriting, which compresses and merges logs efficiently.

- **Disadvantages**:
  - AOF files are usually larger than RDB snapshots for the same data.
  - Write QPS is typically lower with AOF compared to RDB due to frequent `fsync` operations. Real-time writing can significantly reduce performance.
  - Earlier AOF implementations had bugs where data recovery might not be identical to the original dataset. However, newer versions mitigate this by reconstructing commands from in-memory data, improving robustness.

### Choosing Between RDB and AOF

- Do not rely solely on RDB as it may lead to significant data loss.
- Do not rely solely on AOF due to its slower recovery speed and potential bugs.
- The recommended approach is to use both RDB and AOF. Use AOF to ensure minimal data loss and as the primary recovery method. Use RDB for cold backups and faster recovery if AOF files are lost or corrupted.
