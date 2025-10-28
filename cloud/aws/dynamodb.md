## Concepts

### Primary key

The primary key uniquely identifies each item in the table.

DynamoDB supports 2 kinds of primary keys:
- Primary key that is composed of 1 attribute known as the **partition key**, the partition key uniquely identifies each item.
- Primary key (**composite primary key**) that is composed of 2 attributes known as the **partition key** and **sort key**, the (parition key, sort key) pair uniquely identifies each item.

DynamoDB internally hashes the partition key to determine the partition in which the item will be stored, all items with the same partition key value are stored in ascending order determined by the sort key value (and are returned also in the order they are stored by default).

Partition key is also known as **hash attribute**, sort key is also known as **range attribute**. You can query items in a table by applying equality expressions on the partition key or range expressions on the sort key.

Each primary key attribute (partition key, sort key) must be a scalar, e.g. string, number or binary.

### Secondary indexes

If you want to (efficiently) query data in a table using an alternate key other than the primary key, you can create an index on the table, and then query/scan the index in much the same way as you would query the table.

DynamoDB supports 2 kinds of indexes:
- Global secondary index: partition key and sort key can be different from the table.
- Local secondary index: same partition key with the table but different sort key.

Default quota: each table can have 20 global and 5 local seconday indexes.

Every secondary index is associated with exactly one table from which it obtains its data, this is called the base table for the index.

When you CUD an item in the table, DynamoDB automatically update the base table's indexes.

When you create an index, besides the index's partition key and sort key, you also need to specify which extra attributes that will be copied/projected over. At a minimum, DynamoDB projects the key attributes from the base table to the index.

DynamoDB ensures that secondary indexes are eventually consistent with their base table. You can request strongly consistent `Query` or `Scan` actions on a table or a local secondary index, global secondary indexes only support eventualy consistency.

## How it works

### Partitions and data distribution

A **partition** is an allocation of storage for a table, backed by SSDs and automatically replicated across multiple AZs within an AWS Region. Partition management is handled entirely by DynamoDB.

### Adaptive capacity

Core features:
- Dynamic partitioning.
- High-traffic item isolation.
- Throughput boosting.










