# Overview
1. The word consistency is terribly overloaded: 
   * replica consistency and the issue of eventual consistency that arises in asynchronously replicated systems (see “Problems with Repli‐ cation Lag” on page 161).
   * Consistent hashing is an approach to partitioning that some systems use for reba‐ lancing (see “Consistent Hashing” on page 204).
   * In the CAP theorem (see Chapter 9), the word consistency is used to mean linearizability (see “Linearizability” on page 324).
   * In the context of ACID, consistency refers to an application-specific notion of the database being in a “good state.”
1. The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once, and then let applications rely on those guarantees. This is the same approach as we used with transactions: by using a transaction, the application can pretend that there are no crashes (atomicity), that nobody else is concurrently accessing the database (isola‐ tion), and that storage devices are perfectly reliable (durability).
2. one of the most important abstractions for distributed systems is consensus: that is, getting all of the nodes to agree on something

# Consistency Guarantees
1. Most replicated databases provide at least eventual consistency, which means that if you stop writing to the database and wait for some unspecified length of time, then eventually all read requests will return the same value
2. However, this is a very weak guarantee—it doesn’t say anything about when the repli‐ cas will converge.
3. The edge cases of eventual consistency only become apparent when there is a fault in the system (e.g., a network interruption) or at high concurrency
4. systems with stronger guarantees may have worse performance or be less fault-tolerant than systems with weaker guaran‐ tees.
5. transaction isolation is primarily about avoiding race conditions due to concurrently executing transactions, whereas distributed consistency is mostly about coordinating the state of replicas in the face of delays and faults.

# Linearizability
1. Linearizability/atomic consistency/strong consistency/immediate consistency/external consistency: one of the strongest consistency models
2. the basic idea is to make a system appear as if there were only one copy of the data, and all operations on it are atomic. With
3. Maintaining the illusion of a single copy of the data means guaranteeing that the value read is the most recent, up-to-date value, and doesn’t come from a stale cache or replica. In other words, linearizability is a *recency guarantee*
## What Makes a System Linearizable?
1. In a linearizable system we imagine that there must be some point in time (between the start and end of the write operation) at which the value of x atomically flips from 0 to 1.
2. recency guarantee we discussed earlier: once a new value has been written or read, all subsequent reads see the value that was written, until it is overwritten again.
### Linearizability Versus Serializability
1. Serializability is an isolation property of transactions, where every transaction may read and write multiple objects (rows, documents, records)
   * It guarantees that transactions behave the same as if they had executed in some serial order (each transaction running to completion before the next transaction starts). It is okay for that serial order to be different from the order in which transactions were actually run.
1. Linearizability is a recency guarantee on reads and writes of a register (an indi‐ vidual object). It doesn’t group operations together into transactions, so it does not prevent problems such as write skew, unless you take additional measures such as materializing conflicts
1. A database may provide both serializability and linearizability, and this combination is known as strict serializability or strong one-copy serializability (strong-1SR). 
2. Implementations of serializability based on two-phase locking or actual serial execution are typically linearizable.
3. However, serializable snapshot isolation is not linearizable: by design, it makes reads from a consistent snapshot, to avoid lock contention between readers and writers. The whole point of a consistent snapshot is that it does not include writes that are more recent than the snapshot, and thus reads from the snapshot are not linearizable.

## Relying on Linearizability
### Locking and leader election
1. One way of electing a leader is to use a lock: every node that starts up tries to acquire the lock, and the one that succeeds becomes the leader. No matter how this lock is implemented, it must be linearizable: all nodes must agree which node owns the lock; otherwise it is useless
2. Coordination services like Apache ZooKeeper and etcd are often used to implement distributed locks and leader election. They use consensus algorithms to implement linearizable operations in a fault-tolerant way
3. ZooKeeper and etcd provide linearizable writes, but reads may be stale, since by default they can be served by any one of the replicas. You can optionally request a linearizable read: etcd calls this a quorum read, and in ZooKeeper you need to call sync() before the read
### Constraints and uniqueness guarantees
1. If you want to enforce this constraint as the data is written (such that if two people try to concurrently create a user or a file with the same name, one of them will be returned an error), you need linearizability
2. This situation is actually similar to a lock: when a user registers for your service, you can think of them acquiring a “lock” on their chosen username. The operation is also very similar to an atomic compare-and-set, setting the username to the ID of the user who claimed it, provided that the username is not already taken.
### Cross-channel timing dependencies
This problem arises because there are two different communication channels between the web server and the resizer: the file storage and the message queue. Without the recency guarantee of linearizability, race conditions between these two channels are possible. This

## Implementing Linearizable Systems
1. Since linearizability essentially means “behave as though there is only a single copy of the data, and all operations on it are atomic,” the simplest answer would be to really only use a single copy of the data. However,
2. However, that approach would not be able to tolerate faults: if the node holding that one copy failed, the data would be lost, or at least inaccessible until the node was brought up again.
3. The most common approach to making a system fault-tolerant is to use replication
4. Single-leader replication (potentially linearizable)
   * If you make reads from the leader, or from synchronously updated followers, they have the poten‐ tial to be linearizable. Partitioning (sharding) a single-leader database, so that there is a separate leader per partition, does not affect linearizability, since it is only a single-object guarantee. Cross-partition transactions are a different mat‐ ter
1. Consensus algorithms (linearizable)
   * Some consensus algorithms bear a resemblance to single-leader replication. However, consensus protocols contain measures to prevent split brain and stale replicas. Thanks to these details, consensus algorithms can implement linearizable storage safely. This is how Zoo‐ Keeper and etcd work, for example.
1. Multi-leader replication (not linearizable)
   * because they concurrently process writes on multiple nodes and asynchronously replicate them to other nodes. For this reason, they can produce conflicting writes that require resolution
1. Leaderless replication (probably not linearizable)
   * Depending on the exact configuration of the quorums, and depending on how you define strong consis‐ tency, you may not obtain “strong consis‐ tency”
   * “Last write wins” conflict resolution methods based on time-of-day clocks are almost cer‐ tainly nonlinearizable, because clock timestamps cannot be guaranteed to be consistent with actual event ordering due to clock skew. 
   * Sloppy quorums also ruin any chance of linearizability. 
   * Even with strict quorums, nonlinearizable behavior is possible

### Linearizability and quorums
1. Intuitively, it seems as though strict quorum reads and writes should be linearizable in a Dynamo-style model. However, when we have variable network delays, it is pos‐ sible to have race conditions
2. Interestingly, it is possible to make Dynamo-style quorums linearizable at the cost of reduced performance: a reader must perform read repair (see “Read repair and anti- entropy” on page 178) synchronously, before returning results to the application, and a writer must read the latest state of a quorum of nodes before sending its writes
3. However, Riak does not perform synchronous read repair due to the performance penalty [26]. Cassandra does wait for read repair to complete on quo‐ rum reads [27], but it loses linearizability if there are multiple concurrent writes to the same key, due to its use of last-write-wins conflict resolution.
4. In summary, it is safest to assume that a leaderless system with Dynamo-style replica‐ tion does not provide linearizability

## The Cost of Linearizability
1. 


# Ordering Guarantees
## Ordering and Causality
## Sequence Number Ordering
## Total Order Broadcast
# Distributed Transactions and Consensus
## Atomic Commit and Two-Phase Commit (2PC)
## Distributed Transactions in Practice
## Fault-Tolerant Consensus
## Membership and Coordination Services

## Consistency Patterns

### Weak consistency
1. After a write, reads may or may not see it. A best effort approach is taken. 
2. This approach is seen in systems such as memcached. Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games. 
For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

### Eventual consistency
1. After a write, reads will eventually see it (typically within milliseconds). Data is replicated asynchronously.
2. This approach is seen in systems such as DNS and email. Eventual consistency works well in highly available systems.

### Strong consistency
1. After a write, reads will see it. Data is replicated synchronously.
2. This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.

