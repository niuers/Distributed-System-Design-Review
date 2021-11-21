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
1. With a multi-leader database, each datacenter can continue operating normally: since writes from one datacenter are asynchronously replicated to the other, the writes are simply queued up and exchanged when network connectivity is restored.
2. If the network between datacenters is interrupted in a single-leader setup, clients con‐ nected to follower datacenters cannot contact the leader, so they cannot make any writes to the database, nor any linearizable reads. They can still make reads from the follower, but they might be stale (nonlinearizable). If the application requires linear‐ izable reads and writes, the network interruption causes the application to become unavailable in the datacenters that cannot contact the leader.
3. any linearizable database has this problem, no matter how it is implemented. The
### The CAP Theorem
1. The trade-off is as follows
2. If your application requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the network problem is fixed, or return an error (either way, they become unavailable).
3. If your application does not require linearizability, then it can be written in a way that each replica can process requests independently, even if it is disconnected from other replicas (e.g., multi-leader). In this case, the application can remain available in the face of a network problem, but its behavior is not linearizable
4. Thus, applications that don’t require linearizability can be more tolerant of network problems. This insight is popularly known as the CAP theorem
### The Unhelpful CAP Theorem
1. CAP is sometimes presented as Consistency, Availability, Partition tolerance: pick 2 out of 3. Unfortunately, putting it this way is misleading [32] because network parti‐ tions are a kind of fault, so they aren’t something about which you have a choice: they will happen whether you like it or not
2. Thus, a better way of phras‐ ing CAP would be either Consistent or Available when Partitioned
3. The CAP theorem as formally defined [30] is of very narrow scope: it only considers one consistency model (namely linearizability) and one kind of fault (network parti‐ tions, or nodes that are alive but disconnected from each other). It doesn’t say any‐thing about network delays, dead nodes, or other trade-offs.
### Linearizability and network delays
1. The same is true of many distributed databases that choose not to provide lineariza‐ ble guarantees: they do so primarily to increase performance, not so much for fault tolerance. Linearizability is slow—and this is true all the time, not only during a network fault
2. 
# Ordering Guarantees
1. Ordering Examples
   * the main purpose of the leader in single-leader replica‐ tion is to determine the order of writes in the replication log, —that is, the order in which followers apply those writes.
   * Serializability, which we discussed in Chapter 7, is about ensuring that transac‐ tions behave as if they were executed in some sequential order
   * The use of timestamps and clocks in distributed systems
1. It turns out that there are deep connections between ordering, linearizability, and consensus.

## Ordering and Causality
1. There are several reasons why ordering keeps coming up, and one of the reasons is that it helps preserve causality.
2. In the context of snapshot isolation for transactions, we said that a transaction reads from a consistent snapshot. But what does “consistent” mean in this context? It means consistent with causality: if the snapshot contains an answer, it must also contain the ques‐ tion being answered
3. Causality imposes an ordering on events: cause comes before effect
4. If a system obeys the ordering imposed by causality, we say that it is causally consis‐ tent. For example, snapshot isolation provides causal consistency: when you read from the database, and you see some piece of data, then you must also be able to see any data that causally precedes it (assuming it has not been deleted in the meantime).
### The causal order is not a total order
1. A total order allows any two elements to be compared, so if you have two elements, you can always say which one is greater and which one is smaller.
2. partially ordered: in some cases one set is greater than another (if one set contains all the elements of another), but in other cases they are incomparable.
3. Linearizability: In a linearizable system, we have a total order of operations
4. Causality: We said that two operations are concurrent if neither happened before the other. Put another way, two events are ordered if they are causally related (one happened before the other), but they are incomparable if they are concurrent. This means that causality defines a partial order, not a total order: some operations are ordered with respect to each other, but some are incomparable
5. Therefore, according to this definition, there are no concurrent operations in a line‐ arizable datastore
6. Concurrency would mean that the timeline branches and merges again—and in this case, operations on different branches are incomparable (i.e., concurrent). Ex.git
### Linearizability is stronger than causal consistency
1. linearizability implies causality: any system that is linearizable will preserve cau‐ sality correctly
2. making a system linearizable can harm its performance and availability, especially if the system has significant network delays
3. The good news is that a middle ground is possible. Linearizability is not the only way of preserving causality—there are other ways too.
4. A system can be causally consistent without incurring the performance hit of making it linearizable (in particular, the CAP theorem does not apply). In fact, causal consistency is the strongest possible consistency model that does not slow down due to network delays, and remains available in the face of network failures
5. In many cases, systems that appear to require linearizability in fact only really require causal consistency, which can be implemented more efficiently.
### Capturing causal dependencies
1. In order to determine the causal ordering, the database needs to know which version of the data was read by the application

## Sequence Number Ordering
1. Although causality is an important theoretical concept, actually keeping track of all causal dependencies can become impractical.
2. However, there is a better way: we can use sequence numbers or timestamps (better come from a logical clock) to order events.
3. Such sequence numbers or timestamps are compact (only a few bytes in size), and they provide a total order: that is, every operation has a unique sequence number, and you can always compare two sequence numbers to determine which is greater (i.e., which operation happened later).
4. In particular, we can create sequence numbers in a total order that is consistent with causality. 
5. Such a total order captures all the causality information, but also imposes more ordering than strictly required by causality.
6. In a database with single-leader replication (see “Leaders and Followers” on page 152), the replication log defines a total order of write operations that is consistent with causality. The leader can simply increment a counter for each operation, and thus assign a monotonically increasing sequence number to each operation in the replication log.
7. If a follower applies the writes in the order they appear in the replica‐ tion log, the state of the follower is always causally consistent (even if it is lagging behind the leader).

### Noncausal sequence number generators
1. If there is not a single leader (perhaps because you are using a multi-leader or leader‐ less database, or because the database is partitioned), it is less clear how to generate sequence numbers for operations.
   * Each node can generate its own independent set of sequence numbers. For
   * You can attach a timestamp from a time-of-day clock (physical clock) to each operation
   * You can preallocate blocks of sequence numbers. For
1. These three options all perform better and are more scalable than pushing all opera‐ tions through a single leader that increments a counter.
2. However, they all have a problem: the sequence numbers they generate are not consistent with causality
3. The causality problems occur because these sequence number generators do not cor‐ rectly capture the ordering of operations across different nodes:
   * Each node may process a different number of operations per second. Thus, If you have an odd-numbered operation and an even-numbered operation, you cannot accurately tell which one causally happened first.
   * Timestamps from physical clocks are subject to clock skew, which can make them inconsistent with causality
   * In the case of the block allocator, one operation may be given a sequence number in the range from 1,001 to 2,000, and a causally later operation may be given a number in the range from 1 to 1,000. Here, again, the sequence number is incon‐ sistent with causality.

### Lamport timestamps
1. Although the three sequence number generators just described are inconsistent with causality, there is actually a simple method for generating sequence numbers that is consistent with causality. It is called a Lamport timestamp
2. Each node has a unique identifier, and each node keeps a counter of the number of operations it has pro‐ cessed. The Lamport timestamp is then simply a pair of (counter, node ID).
3. it provides total ordering: if you have two timestamps, the one with a greater counter value is the greater timestamp; if the counter values are the same, the one with the greater node ID is the greater timestamp
4. The key idea about Lamport timestamps, which makes them consis‐ tent with causality, is the following: every node and every client keeps track of the maximum counter value it has seen so far, and includes that maximum on every request.
5. When a node receives a request or response with a maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum.
6. As long as the maximum counter value is carried along with every operation, this scheme ensures that the ordering from the Lamport timestamps is consistent with causality, because every causal dependency results in an increased timestamp
7. version vectors can distinguish whether two operations are concurrent or whether one is causally dependent on the other, whereas Lamport timestamps always enforce a total ordering.From the total ordering of Lamport time‐stamps, you cannot tell whether two operations are concurrent or whether they are causally dependent. The advantage of Lamport timestamps over version vectors is that they are more compact.
### Timestamp ordering is not sufficient
1. Lamport timestamps are not quite sufficient to solve many common problems in dis‐ tributed systems
   * unique username constraint
   * However, it is not sufficient when a node has just received a request from a user to create a username, and needs to decide right now whether the request should succeed or fail. At that moment, the node does not know whether another node is concurrently in the process of creating an account with the same username, and what timestamp that other node may assign to the operation. 
   * In order to be sure that no other node is in the process of concurrently creating an account with the same username and a lower timestamp, you would have to check with every other node to see what it is doing.
1. The problem here is that the total order of operations only emerges after you have collected all of the operations. If another node has generated some operations, but you don’t yet know what they are, you cannot construct the final ordering of opera‐ tions: the unknown operations from the other node may need to be inserted at vari‐ ous positions in the total order.
2. To conclude: in order to implement something like a uniqueness constraint for user‐ names, it’s not sufficient to have a total ordering of operations—you also need to know when that order is finalized. 
3. This idea of knowing when your total order is finalized is captured in the topic of *total order broadcast*.

## Total Order Broadcast
1. in a distributed system, getting all nodes to agree on the same total ordering of opera‐ tions is tricky. In the last section we discussed ordering by timestamps or sequence numbers, but found that it is not as powerful as single-leader replication (if you use timestamp ordering to implement a uniqueness constraint, you cannot tolerate any faults).
2. As discussed, single-leader replication determines a total order of operations by choosing one node as the leader and sequencing all operations on a single CPU core on the leader. The challenge then is how to scale the system if the throughput is greater than a single leader can handle, and also how to handle failover if the leader fails. In the distributed systems litera‐ ture, this problem is known as total order broadcast or atomic broadcast
3. Scope of ordering guarantee: Partitioned databases with a single leader per partition often main‐ tain ordering only per partition, which means they cannot offer consistency guarantees (e.g., consistent snapshots, foreign key ref‐ erences) across partitions. Total ordering across all partitions is possible, but requires additional coordination
4. Total order broadcast is usually described as a protocol for exchanging messages between nodes. Informally, it requires that two safety properties always be satisfied
   * Reliable delivery: No messages are lost: if a message is delivered to one node, it is delivered to all nodes.
   * Totally ordered delivery: Messages are delivered to every node in the same order
### Using total order broadcast
1. Consensus services such as ZooKeeper and etcd actually implement total order broadcast. This fact is a hint that there is a strong connection between total order broadcast and consensus
2. Total order broadcast is exactly what you need for database replication: if every mes‐ sage represents a write to the database, and every replica processes the same writes in the same order, then the replicas will remain consistent with each other (aside from any temporary replication lag). This principle is known as **state machine replication**
3. Similarly, total order broadcast can be used to implement serializable transactions: as discussed in “Actual Serial Execution” on page 252, if every message represents a deterministic transaction to be executed as a stored procedure, and if every node pro‐ cesses those messages in the same order, then the partitions and replicas of the data‐ base are kept consistent with each other
4. An important aspect of total order broadcast is that the order is fixed at the time the messages are delivered: a node is not allowed to retroactively insert a message into an earlier position in the order if subsequent messages have already been delivered. This fact makes total order broadcast stronger than timestamp ordering
5. Another way of looking at total order broadcast is that it is a way of creating a log (as in a replication log, transaction log, or write-ahead log): delivering a message is like appending to the log. 
6. Total order broadcast is also useful for implementing a lock service that provides fencing tokens (see “Fencing tokens” on page 303). Every request to acquire the lock is appended as a message to the log, and all messages are sequentially numbered in the order they appear in the log. The sequence number can then serve as a fencing token, because it is monotonically increasing. In ZooKeeper, this sequence number is called zxid
### Implementing linearizable storage using total order broadcast
1. how to build a linearizable compare-and-set operation from total order broadcast
1. Total order broadcast is asynchronous: messages are guaranteed to be delivered relia‐ bly in a fixed order, but there is no guarantee about when a message will be delivered (so one recipient may lag behind the others). By contrast, linearizability is a recency guarantee: a read is guaranteed to see the latest value written.
2. However, if you have total order broadcast, you can build linearizable storage on top of it. For example, you can ensure that usernames uniquely identify user accounts.
3. While this procedure ensures linearizable writes, it doesn’t guarantee linearizable reads—if you read from a store that is asynchronously updated from the log, it may be stale.
4. To be precise, the procedure described here provides sequential consistency [47, 64], sometimes also known as timeline consistency [65, 66], a slightly weaker guarantee than linearizability.
### Implementing total order broadcast using linearizable storage
1. Note that unlike Lamport timestamps, the numbers you get from incrementing the linearizable register form a sequence with no gaps. in fact, this is the key difference between total order broadcast and timestamp ordering.
2. This is no coincidence: it can be proved that a linearizable compare-and-set (or increment-and-get) register and total order broadcast are both equivalent to consen‐ sus

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

