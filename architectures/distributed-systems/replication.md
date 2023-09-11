## Replication

Replication means keeping a copy of the same data on multiple machines that are connected via a network.

Reasons for replication:
- To keep data geographically close to your users (and thus reduce latency).
- To allow the system to continue working even if some of its parts have failed (and thus increase availability).
- To scale out the number of machines that can serve read queries (and thus increase read throughput).

All of the difficulty in replication lies in handling changes to replicated data.

## Leaders and Followers

Each node that stores a copy of the database is called a *replica*. With multiple replicas, every write to the database needs to be processed by every replica. The most common solution for this is called *leader-based replication*:
- One of the replicas is designated the *leader* (master, primary): write requests are first sent to the leader.
- The other replicas are known as *followers* (read replicas, slaves, secondaries, hot standbys): whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of a *replication log* or *change stream*. Each follower applies all writes in the same order as they were processed on the leader.
- From clients' point of view, they can read either the leader or any of the followers, writes are only accepted on the leader.

This mode of replication is a built-in feature of many sysmtes:
- Relational databases: PostgreSQL, MySQL, Oracle Data Guard, etc
- Nonrelational databases: MongoDB, RethinkDB, and Espresso.
- Distributed message brokers: Kafka and RabbitMQ highly available queues.
- Network filesystems, replicated block devices (DRBD).

The replication can happen sync or async. (In relational db, this is often a configurable option.):
- Sync replication: advantage is that the follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader. The disadvantage is that if the follower does not respond (due to being crashed, network fault, etc.), the write cannot be processed. The leader must block all writes and wait until the synchronous replica is available again.
  Therefore, practically not all followers are sync. In practice, if sync replication is enabled on a db, it usually means that one of the followers is sync, and the others are async. If the sync follower becomes unavailable or slow, one of the async followers is made sync. This guarantees that you have an up-to-date copy of the data on at least 2 nodes. This configuration is sometimes called *semi-sync*.
- Async replication: in a fully async replication config, if the leader fails, any writes that have not yet been replicated to followers are lost, this means a write is not guaranteed to be durable. However this config has the advantage that the leader can continue processing writes, even if all of its followers have fallen behind (this is called *replication lag*). Despite that disadvantage, async replication is widely used, especially if there are many followers or if they are geographically distributed.

So there's an obvious trade-off: sync replication makes sure that data is not lost, while async replication offers better performance and availibility.

## Setting up new followers

From time to time you will need to set up new followers, question is how to ensure they have an accurate copy of the leader's data. Several options:
- Simple copying data from the leader to the new followers. The problem is that clients are constantly writing to the db, thus the data is always in flux, so a standard copy would see different parts/versions of the db at different points in time.
- Locking the db/leader while copying. This goes against our high availability goal.
- No downtime process:
  - Take a consistent snapshot of the leader (without locking the db).
  - Copy the snapshot to new followers.
  - The followers connect to the leader and request all the data changes that have happened since the snapshot was taken.
  - When the follower has processed the backlog of data changes since the snapshot, we say it has caught up. It can now continue to process data changes from the leader as they happen.

## Handling node outages

The goal is to achieve high availability with leader-based replication: keep the system running despite individual node failures, and to keep the impact of a node outage as small as possible.

**Follower failure: catch-up recovery**

Each follower keeps a log of data changes it has received from the leader on its local disk. After issues such as crash or interrupted network, the follower can recover easity by knowing the last transaction that was processed before the fault occurred from the log and requesting all the data changes from the leader.

**Leader failure: failover**

When the leader fails, one of the followers needs to be promoted to be the new leader, clients need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader. This process is called *failover*.

An automatic failover process usually consists of the following steps:
- Determining that the leader has failed: most systems use a timeout: if a node doesn't respond for some period of time (say 30 sec), it is assumed to be dead.
- Choosing a new leader: best candidate is usually the replica with the most up-to-date data changes from the old leader (to minimize data loss).
- Reconfiguring the system to use the new leader: clients send new write requests to the new leader. The old leader (after it comes back) becomes a follower and recognizes the new leader.

Things that can go wrong:
- If async rep is used, the new leader may not have received all the writes from the old leader before it failed. Most common solution is for the old leader's unreplicated writes to be discarded, which may violate clients' durability expectations.
- Discarding writes is especially dangerous if other storage systems outside of the db need to be coordinated with the db contents. For example the new leader can reused some primary keys that were previously assigned by the old leader, which cause data inconsistency.
- In some certain fault scenarios, 2 nodes could both believe that they are the leader (called *split brain*). This is dangerous, data is likely to be lost or corrupted if both leaders accept writes. Some systems have a mechanism to shut down one node if 2 leaders are detected, however we can end up with both nodes being shut down if it is not carefully designed.
- Need to fine tune the timeout parameter: a long timeout means a longer time to recovery in case the leader fails, a short one could cause unnecessary failovers.

There are no easy solutions to these problems, which are fundamental in distributed systems. Some operation teams prefer to perform failover manually

## Replication Log

Several different replication methods are used in practice.

**Statement-based replication**

Leaders log every write request (statement) that it executes and send that statement log to its followers. Problems:
- Statements that contain nondeterministic functions (`NOW()`, `RAND()`) are likely to generate different value on each replica. Leader can replace them with fixed value before sending however there are just too many edge cases.
- In some cases, statements must be executed in exactly the same order on each replica, or else they may have a different effect.
- Statements that have side effects may result in different side effects occuring on each replica, unless the side effects are deterministic.

Due to those problems, other replication methos are preferred.

**Write-ahead log (WAL) shipping**

All writes that the leader applied are logged to an append-only sequence of bytes. The leader then writes the log to disk and sends it across the network to its followers. When the follower process the log, it builds a copy of the exact same data structures as found on the leader.

Disadvantages:
- The log describes the data on a very low level (details of which bytes were changed in which disk blocks), this makes replication closely coupled to the storage engine.

This method of replication is used in PostgreSQL, Oracle, etc.

**Logical (row-based) log replication**

An alternative is to use different log formats (logical log) for replication and for the storage engine (physical log), which allows the replication log to be decoupled from the storage engine internals.

**Trigger-based replication**

The above approaches are implemented by the database system. But sometimes you may want to have more flexibility and move replication up to the application layer.

Features available in many relational databases that can make data changes available to an application by reading the db log: *triggers* and *stored procedures*. A trigger lets you register custome application code that is automatically executed when a data change occurs (write transaction).

Disadvantage:
- has greater overheads than other methods.
- is more prone to bugs and limitations than built-in replication methods.

However it can still be useful due to its flexibility.

## Replication Lag

Replication lag is the delay between a write happening on the leader and being reflected on a follower. Some problems that are likely to occur when there is replication lag:

**Reading your own writes**

When a user submits new data then views it shortly after, they may see the old data. This happens when the read request is handled by a replica and new data have not reached it yet.

In this case, we need read-after-write (read-your-writes) consistency: users always see their up-to-date data submitted themselves. Some implementation:
- When a user read something that the user may have modified, read it from the leader. For example, on a social network system, always read the user's own profile from the leader, and any other users' profiles from a follower.
- If most things in the application are editable by the user, other criteria may be used to decide whether to read from the leader:
  - Tracking the last update, read from the leader within 1 minute after that.
  - Preventing queries on followers that is more than 1 minute behind the leader.
- Ensure that the replica only serves reads for a user if it reflects updates at least until the user's most recent write's timestamp. If it does not, either the read can be handled by another replica or the query can wait until the replica has caught up.
- If replicas are distributed across multiple datacenters, any request that needs to be served by the leader must be routed to the datacenter that contains the leader.

Another problem: users enter some data on one device then view it on another device. In this case we need cross-device read-after-write consistency. Addtional issues to consider:
- Timestamp on users' last update become more difficult, since the code running on one device doesn't know what updates have happened on the other device. This metadata will need to be centralized.
