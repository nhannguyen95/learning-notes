---

  

mindmap-plugin: basic

  

---

# Partitioning

## Introduction
- Also known as *sharding*
- Divide data in such a way that each piece of data belongs to exactly one partition
- Each partition is a small db of its own
	- The DB may support operations that touch multiple partitions at the same time
- Using with Replication
	- Partitioning is usually combined with replication
	- Copies of each partition are stored on multiple *nodes*
		- Shard first, then replicate the shard
		- Not replicate first, then shard the replication
	- A node may store more than one partition
		- Node 1: P1 Leader, P2 Follower, P3 Follower
		- Node2: P1 Follower, P2 Leader, P3 Follower
		- so on.
- *Skewed*
	- Some partitions have more data/queries than others
	- (Ideally, data/query load are spread evenly across partitions)
- *Hot spot*
	- A partition with disproportionately high load

## Partitioning strategies
- Randomization
	- Assigning records to nodes randomly
	- Pros
		- Data is distributed quite evenly across nodes (avoid having hot spots)
	- Cons
		- No way to know which node a particular item is on, have to query all nodes
- By Key Range
	- Assigning a continuous range of keys to each partition
	- Within each partition, we can keep keys in sorted order for ease of range queries
		- You can treat the key as a concatenated index in order to fetch several related records in 1 query
	- Used in
		- Bigtable, HBase, RethinkDB, MongoDB pre 2.4
	- Cons
		- Certain access patterns can lead to hot spots
- By Hash of Key
	- Motivation: reduce risk of skew and hot spots
	- A given key's partition is determined by a hash function
		- For partitioning purpose, hash function need not be cryptographically strong
	- Each partition is assigned a range of hashes (rather than a range of keys)
		- Key whose value falls within a partition's range will be stored in that partition
		- The partition boundaries can be evenly spaced, or chosen pseudorandomly
	- Cons: inability to do efficient range queries
	- Hot spots can't be avoided entirely
		- When most reads/writes are for the same key
		- Most systems today can't automatically compensate for such a hot key
		- Application can help mitigate
			- A random number is added to the beginning or end of hot key
				- A 2-digit decimal can split the key evenly across 100 different keys
			- Writes are further distributed to different partitions, but reads now have to do extra work
			- Only makes sense to split hot key, doing this for cold key would add overheads
				- Need to keep track of which keys are split

## Secondary Indexes
- Secondary indexes don't map neatly to partitions
- 2 main approaches
	- **Document-based partitioning**
	- **Term-based partitioning**