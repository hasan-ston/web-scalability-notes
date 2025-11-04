# Chapter 5 — Data Layer Scaling

## MySQL Replication

### Master-Slave Replication

In MySQL replication, you synchronize the state of two servers - one **master** and one or more **slaves**.

**Key characteristics:**
- Data can only be read from slaves
- All modifications happen through the master
- Data-modifying commands are sent to the master
- Master executes the commands and writes them to a **binlog file**
- Master returns response to client
- Slaves can connect to master anytime and request incremental updates from the binlog

**Important:** MySQL replication is **asynchronous**, which decouples the master from slaves. The master doesn't wait for slaves to confirm they received the updates.

### Scaling Reads with Slaves

You can add multiple slave servers (clones) to increase read capacity. More copies of the same data means you can distribute read queries across more servers.

**Pattern:**
- Writes go to master
- Reads distributed across multiple slaves
- Each slave is a full copy of the master's data

### Master-Master Replication

An alternative topology where Master A replicates from Master B, and Master B replicates from Master A (circular replication).

MySQL allows this because each statement written to the binlog includes the name of the server it originally came from, preventing infinite loops.

**Typical failover sequence:**
1. Both masters functioning normally
2. Master A fails → outage begins
3. Failure detected, initiate failover
4. Direct all writes to Master B and reads to its slaves
5. Outage ends
6. Rebuild Master A
7. Master A becomes hot standby

### Replication Lag

**Replication lag** measures how far behind a slave is from its master.

Lag can spike suddenly because statements are executed one at a time on slaves. If the master executes a heavy query or batch of updates, slaves must replay them sequentially, which can cause temporary lag.

**Impact of data types:**
- **Inactive data:** May increase database index size but doesn't put much pressure on the database since it's rarely accessed
- **Active data:** Frequently accessed, so the database must either buffer it in memory or fetch from disk (where the bottleneck usually is)

### Bypassing Replication for Schema Changes

Sometimes you want to make changes on the master without replicating them to slaves (useful for heavy operations like ALTER TABLE).

**Process:**
1. Disable binlogging on the master for the operation
2. Master applies the change locally but doesn't record it in replication log
3. Slaves won't automatically receive the update
4. Manually run the same change on each slave

This keeps all databases in sync while avoiding the heavy replication load from large operations. The master and slaves end up with identical schemas through manual coordination instead of automatic replication.

**Important:** This is only safe for schema changes, not normal data writes, which should always replicate automatically.

---

## Data Partitioning (Sharding)

### What is Sharding?

**Sharding** involves dividing your dataset into smaller buckets and assigning each bucket to a single server. Servers become independent from one another.

The critical requirement is the **sharding key** - a way to locate which shard holds the data without having to ask all servers.

### Benefits

- Can be implemented in the application layer on top of any data store
- Allows horizontal scaling of database servers to almost any size
- No single server holds all the data

### ACID and Distributed Transactions

**ACID** refers to transaction properties supported by most relational databases:
- **A**tomicity: all or nothing
- **C**onsistency: data integrity rules maintained
- **I**solation: transactions don't interfere with each other
- **D**urability: committed data persists

Maintaining ACID properties across shards requires **distributed transactions**, which are complex and expensive to execute.

### Sharding Key Mapping

**Simple approach: Modulo operator**

`Modulo(n, x)` gives the remainder of dividing x by n.

Example: If you have 4 shards, user ID 123 goes to shard `123 % 4 = 3`

**Problem:** This approach depends on the number of servers. If you add servers, the mapping changes for almost every record, requiring massive data migration.

**Better approach: Mapping table**

Keep all shard mappings in a separate database that acts as the source of truth.

Can deploy a MySQL master server for the mapping table and replicate that data to all shards. This way, adding shards only requires updating the mapping table, not rehashing everything.

### Scaling Out Existing Shards

When an existing shard gets too large:

1. Set up new servers as replicas of current shards
2. Stop all writes briefly to allow in-flight updates to replicate
3. Once slaves catch up, disable replication to new servers (they shouldn't continue replicating data they won't be responsible for)
4. Change application configuration to use new servers
5. Resume traffic

### Challenges of Sharding

**Can't execute queries spanning multiple shards**

Any query touching multiple shards requires:
1. Execute parts of the query on each relevant shard
2. Merge results in the application layer

This adds complexity to queries that would be simple in a single database.

**Tools like Azure SQL Database Elastic Scale** can help manage some of this complexity.

---

## Data Normalization

**Data normalization** is a process of structuring data by breaking it into separate tables.

**Key aspects:**
- Break data down into separate fields
- Each row identified by a primary key
- Rows in different tables reference each other rather than duplicating information

**Benefits:**
- Reduces redundancy
- Better indexing and searching
- Increases data integrity

---

## NoSQL Overview

**NoSQL** is a broad term for data stores that diverge from traditional relational database models. They usually don't support SQL language.

Different NoSQL databases make different tradeoffs to achieve better scalability for specific use cases.

---

## CAP Theorem

**Eric Brewer's CAP theorem** states that it's impossible for a distributed system to simultaneously guarantee all three:

### Consistency
Any node can see the same data at the same time. All state changes must be serializable (happening one after another, not in parallel). This requires coordination across CPUs and servers to ensure latest data is returned.

**Note:** This differs from ACID consistency, which focuses on data integrity rules like foreign keys and uniqueness.

### Availability
Any available node can serve client requests even when other nodes fail.

### Partition Tolerance
System can operate even during network failures when communication between nodes is impossible.

**The tradeoff:** You can only guarantee two of the three. Most distributed systems must tolerate partitions (network failures happen), so you choose between consistency and availability.

---

## Eventual Consistency

**Eventual consistency** is a property where different nodes may have different versions of the data, but state changes eventually propagate to all servers.

If you query a single server, you can't tell whether you got the latest data or an older version because the server you chose might be lagging behind.

### Amazon's Shopping Cart Example

Amazon sacrificed consistency for high availability. Even if some servers were down, people could keep adding items to shopping carts.

**How it works:**
1. Writes go to different servers, potentially creating multiple versions of the same shopping cart
2. When multiple versions are discovered, they're merged by adding all items from all carts
3. Users never lose items added to cart, making it easier to complete purchases

This is **client-side conflict resolution** - the application handles merging conflicting versions.

---

## Quorum Consistency

**Quorum consistency** means the majority of replicas must agree on the result.

- **Writes:** Majority of servers must confirm they persisted the change
- **Reads:** Majority of servers must return the same value

This provides stronger consistency than eventual consistency while still allowing some level of availability.

Some systems like Cassandra let you tune consistency level per operation. MongoDB (a CP system) can enforce secondary node consistency on writes, though this isn't always the default.

---

## Cassandra

### Topology and Failover

Cassandra's architecture makes replacing failed nodes straightforward. Unlike MySQL (which requires backup recovery and replication offset tweaking), you just:
1. Add a new blank node
2. Tell Cassandra which IP address this node is replacing
3. Cassandra handles the rest

### Write Performance

Cassandra uses **append-only data structures**, allowing extremely efficient writes:
- Data is never overwritten in place
- Hard disks never perform random writes
- Greatly increases write throughput

### Tradeoff: Deletes and Updates

Because Cassandra is eventually consistent and uses append-only storage, deletes and updates are internally persisted as inserts.

**Problem:** Use cases with lots of adds and deletes become inefficient because:
- Deletes increase data size rather than reducing it (until compaction cleans them up)
- More disk space used temporarily
- More data to scan during reads

---

## Summary

Scaling the data layer is usually the most challenging area of a web application. 

You can achieve horizontal scalability by:
1. Carefully designing your application
2. Choosing the right data store for your use case
3. Applying three basic scalability techniques:
   - **Functional partitioning** (separate databases for separate functions)
   - **Replication** (multiple copies for read scaling and availability)
   - **Sharding** (splitting data across independent servers)

Each technique has tradeoffs in complexity, consistency, and operational overhead.
