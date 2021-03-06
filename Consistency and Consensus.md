# Overview
1. The word consistency is terribly overloaded: 
   * replica consistency and the issue of eventual consistency that arises in asynchronously replicated systems (see “Problems with Repli‐ cation Lag” on page 161).
   * Consistent hashing is an approach to partitioning that some systems use for reba‐ lancing (see “Consistent Hashing” on page 204).
   * In the CAP theorem (see Chapter 9), the word consistency is used to mean linearizability (see “Linearizability” on page 324).
   * In the context of ACID, consistency refers to an application-specific notion of the database being in a “good state.”
1. The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, implement them once, and then let applications rely on those guarantees. This is the same approach as we used with transactions: by using a transaction, the application can pretend that there are no crashes (atomicity), that nobody else is concurrently accessing the database (isolation), and that storage devices are perfectly reliable (durability).
2. one of the most important abstractions for distributed systems is consensus: that is, getting all of the nodes to agree on something
3. Data inconsistency between replicas can happen  no  matter what  replication  method  the  database  uses  (single-leader,  multi-leader,  or  leaderlessreplication)
4. What is consistency?
  1. What processes can expect when RD/WR shared data concurrently
5. When do consistency concerns arise?
  1. With replication and caching

6. What is a consistency model?
  1. Contract between a distributed data system (e.g., DFS, DSM) and processes constituting its applications
    * E.g.: “If a process reads a certain piece of data, I (the DFS/DSM) pledge to return the value of the last write”
7. Variations boil down to:
  1. The allowable staleness of reads
  2. The ordering of writes across all replicas

# Consistency Guarantees
1. Most replicated databases provide at least eventual consistency, which means that if you stop writing to the database and wait for some unspecified length of time, then eventually all read requests will return the same value
2. However, this is a very weak guarantee—it doesn’t say anything about when the repli‐ cas will converge.
3. The edge cases of eventual consistency only become apparent when there is a fault in the system (e.g., a network interruption) or at high concurrency
4. systems with stronger guarantees may have worse performance or be less fault-tolerant than systems with weaker guaran‐ tees.
5. (Compare with Transaction Isolations) Transaction isolation is primarily about avoiding race conditions due to concurrently executing transactions, whereas distributed consistency is mostly about coordinating the state of replicas in the face of delays and faults.

## Types of Consistency Models

Strict Serializability: Total order + real time guarantees over transactions
Linearizability: Total order + real time guarantees over operations
Sequential consistency: Total order + process order
Causal+ consistency: Causally ordered + replicas eventually converge
Eventual consistency: Eventually everyone should agree on state

### Strict Serializability
1. Total order: There exists a legal total ordering of transactions.
  1. Legal: In the total ordering, a read operation sees the latest write operation.
2. Preserves real-time ordering: Any transaction A that completes before transaction B begins, occurs before B in the total order.
3. Properties
4. Writes in a completed transaction appear to all future reads
5. Once a read sees a value, all future reads must also return the same value (until new write)
6. Pros: Easily reason about correctness of transactions
7. Cons: High read and write latencies
8. A database may provide both serializability and linearizability, and this combinationis known as strict serializability or strong one-copy serializability (strong-1SR)
### Linearizability (Atomic Consistency/Strong Consistency/Immediate  Consistency/External  Consistency)
1. Total order: There exists a legal total ordering of operations
  1. Legal: In the total ordering, a read operation sees the latest write operation.
2. Preserves real-time ordering: Any operation A that completes before operation B begins, occurs before B in the total order.
3. Difference from strict serializability?
  1. In Linearizability, clients only have consistency guarantees for operations, where strict serializability allows clients to use transactions.
    1. It does not prevent problems such as write skew, unless you take additional measures such as materializing conflicts
4. Properties
  1. A completed write appears to all future reads
  1. Once a read sees a value, all future reads must also return the same value (until new write) even if the reads are concurrent with write
5. Pros: Easy to reason about correctness
6. Cons: High read and write latencies

### Sequential Consistency
Total order: There exists a legal total ordering of operations.
○ Legal: In the total ordering, a read operation sees the latest write operation.
● Preserves process ordering: All of a process’ operations appear in that order
in the total order.
● Difference from linearizability?
○ Sequence of ops across processes not determined by real-time
Pros: Can allow more orderings than linearizability
Cons: Many possible sequential executions

### Causal + Consistency
1. Partial order: Order causally related ops the same way across all processes
  1. Replicas eventually converge
  1. Difference from sequential consistency?
2. Only causally related ops need to be ordered: no total order
3. Concurrent ops may be ordered differently across different processes
4. Pros: Preserves causality while improving efficiency
5. Cons: Need to reason about concurrency

### Eventual Consistency
1. This is a very weak guarantee—it doesn’t say anything about whenthe repli‐cas  will  converge.
  1. No "Read-Your-Own-Write" guarantee
  2. The edge cases of eventual consistency only become apparent when thereis a fault in the system (e.g., a network interruption) or at high concurrency.

Eventual convergence: If no more writes, all replicas eventually agree
● Difference from causal consistency?
○ Does not preserve causal relationships
○ Is the “+” in causal+
● Frequently used with application conflict resolution, anti-entropy
Pros: Super duper highly available
Cons: No safety guarantees, need conflict resolution

# Linearizability
1. the basic idea is to make a system appear as if there were only one copy of the data, and all operations on it are atomic. With  this  guarantee,  even  though  there  may  bemultiple replicas in reality, the application does not need to worry about them.
3. linearizability is a *recency guarantee*: it guarantees that the value read is the most recent, up-to-date value, and doesn’t come from a stale cache or replica.
4. it has the downside of being slow, especially in environments with large network delays
## What Makes a System Linearizable?
1. In a linearizable system we imagine that there must be some point in time (between the start and end of the write operation) at which the value of x atomically flips from 0 to 1.
2. recency guarantee we discussed earlier: once a new value has been written or read, all subsequent reads see the value that was written, until it is overwritten again. once a new value has been writtenor  read,  all  subsequent  reads  see  the  value  that  was  written,  until  it  is  overwrittenagain.

3. Any  read  operations  that  overlap  in  time  with  the  write  operation  might  return either 0 or 1, because we don’t know whether or not the write has taken effect at the  time  when  the  read  operation  is  processed.  These  operations  are  concurrent with the write.
4. However,  that  is  not  yet  sufficient  to  fully  describe  linearizability:  if  reads  that  areconcurrent with a write can return either the old or the new value, then readers couldsee a value flip back and forth between the old and the new value several times whilea  write  is  going  on.  That  is  not  what  we  expect  of  a  system  that  emulates  a  “singlecopy of the data.”
  1. Timing Dependency: In a linearizable system we imagine that there must be some point in time (betweenthe start and end of the write operation) at which the value of x atomically flips from 0  to  1.  Thus,  if  one  client’s  read  returns  the  new  value  1,  all  subsequent  reads  must also return the new value, even if the write operation has not yet completed

### Linearizability Versus Serializability
1. Serializability is an isolation property of transactions, where every transaction may read and write multiple objects (rows, documents, records)
   * It guarantees that transactions behave the same as if they had executed in some serial order (each transaction running to completion before the next transaction starts). It is okay for that serial order to be different from the order in which transactions were actually run.
1. Linearizability is a recency guarantee on reads and writes of a register (an indi‐ vidual object). It doesn’t group operations together into transactions, so it does not prevent problems such as write skew, unless you take additional measures such as materializing conflicts
1. A database may provide both serializability and linearizability, and this combination is known as strict serializability or strong one-copy serializability (strong-1SR). 
2. Implementations of serializability based on two-phase locking or actual serial execution are typically linearizable.
3. However, serializable snapshot isolation is not linearizable: by design, it makes reads from a consistent snapshot, to avoid lock contention between readers and writers. The whole point of a consistent snapshot is that it does not include writes that are more recent than the snapshot, and thus reads from the snapshot are not linearizable.

## Relying on Linearizability
1. There a few areas in whichlinearizability is an important requirement for making a system work correctly.

### Locking and leader election
1. One way of electing a leader is to use a lock: every node that starts up tries to acquire the lock, and the one that succeeds becomes the leader. No matter how this lock is implemented, it must be linearizable: all nodes must agree which node owns the lock; otherwise it is useless
2. Coordination services like Apache ZooKeeper and etcd are often used to implement distributed locks and leader election. They use consensus algorithms to implement linearizable operations in a fault-tolerant way. However, a linearizable storage service is the basic foundation for these coordination tasks.
3. Strictly speaking,ZooKeeper and etcd provide linearizable writes, but reads may be stale, since by default they can be served by any one of the replicas. You can optionally request a linearizable read: etcd calls this a quorum read, and in ZooKeeper you need to call sync() before the read

### Constraints and uniqueness guarantees
1. If you want to enforce this constraint as the data is written (such that if two people try to concurrently create a user or a file with the same name, one of them will be returned an error), you need linearizability
2. This situation is actually similar to a lock: when a user registers for your service, you can think of them acquiring a “lock” on their chosen username. The operation is also very similar to an atomic compare-and-set, setting the username to the ID of the user who claimed it, provided that the username is not already taken.
3. a  hard  uniqueness  constraint,  such  as  the  one  you  typically  find  in  rela‐tional  databases,  requires  linearizability.  Other  kinds  of  constraints,  such  as  foreignkey  or  attribute  constraints,  can  be  implemented  without  requiring  linearizability

### Cross-channel timing dependencies
1. This problem arises because there are two different communication channels between the web server and the resizer: the file storage and the message queue. Without the recency guarantee of linearizability, race conditions between these two channels are possible. 
2. Linearizability is not the only way of avoiding this race condition, but it’s the simplestto understand.  you  can  use  alternativeapproaches similar to what we discussed in “Reading Your Own Writes” on page 162,at the cost of additional complexity.

## Implementing Linearizable Systems
1. Since linearizability essentially means “behave as though there is only a single copy of the data, and all operations on it are atomic,” the simplest answer would be to really only use a single copy of the data.
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
1. Intuitively, it seems as though strict quorum reads and writes should be linearizable in a Dynamo-style model. However, when we have variable network delays, it is possible to have race conditions
2. Interestingly, it is possible to make Dynamo-style quorums linearizable at the cost of reduced performance: a reader must perform read repair (see “Read repair and anti-entropy” on page 178) synchronously, before returning results to the application, and a writer must read the latest state of a quorum of nodes before sending its writes
3. However, Riak does not perform synchronous read repair due to the performance penalty. Cassandra does wait for read repair to complete on quo‐ rum reads, but it loses linearizability if there are multiple concurrent writes to the same key, due to its use of last-write-wins conflict resolution.
4. Moreover,  only  linearizable  read  and  write  operations  can  be  implemented  in  thisway;  a  linearizable  compare-and-set  operation  cannot,  because  it  requires  a  consensus algorithm.
5. In summary, it is safest to assume that a leaderless system with Dynamo-style replication does not provide linearizability

## The Cost of Linearizability
1. With a multi-leader database, each datacenter can continue operating normally: since writes from one datacenter are asynchronously replicated to the other, the writes are simply queued up and exchanged when network connectivity is restored.
2. if single-leader replication is used, then the leader must be in oneof the datacenters. Any writes and any linearizable reads must be sent to the leader—thus, for any clients connected to a follower datacenter, those read and write requestsmust be sent synchronously over the network to the leader datacenter.
3. If the network between datacenters is interrupted in a single-leader setup, clients con‐ nected to follower datacenters cannot contact the leader, so they cannot make any writes to the database, nor any linearizable reads. They can still make reads from the follower, but they might be stale (nonlinearizable). If the application requires linearizable reads and writes, the network interruption causes the application to become unavailable in the datacenters that cannot contact the leader.
  1. If the network between datacenters is interrupted in a single-leader setup, clients con‐nected  to  follower  datacenters  cannot  contact  the  leader,  so  they  cannot  make  anywrites to the database, nor any linearizable reads. They can still make reads from thefollower, but they might be stale (nonlinearizable). If the application requires linear‐izable  reads  and  writes,  the  network  interruption  causes  the  application  to  becomeunavailable in the datacenters that cannot contact the leader.
3. any linearizable database has this problem, no matter how it is implemented.

### The CAP Theorem
1. The trade-off is as follows
2. If your application requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the network problem is fixed, or return an error (either way, they become unavailable).
3. If your application does not require linearizability, then it can be written in a way that each replica can process requests independently, even if it is disconnected from other replicas (e.g., multi-leader). In this case, the application can remain available in the face of a network problem, but its behavior is not linearizable
4. Thus, applications that don’t require linearizability can be more tolerant of network problems. This insight is popularly known as the CAP theorem
### The Unhelpful CAP Theorem
1. CAP is sometimes presented as Consistency, Availability, Partition tolerance: pick 2 out of 3. Unfortunately, putting it this way is misleading because network partitions are a kind of fault, so they aren’t something about which you have a choice: they will happen whether you like it or not
2. At  times  when  the  network  is  working  correctly,  a  system  can  provide  both  consis‐tency (linearizability) and total availability. 
3. Thus, a better way of phrasing CAP would be either Consistent or Available when Partitioned
4. The CAP theorem as formally defined is of very narrow scope: it only considers one consistency model (namely linearizability) and one kind of fault (network parti‐ tions, or nodes that are alive but disconnected from each other). It doesn’t say any‐thing about network delays, dead nodes, or other trade-offs.
5. Thus, although CAP hasbeen historically influential, it has little practical value for designing systems 

### Linearizability and network delays
1. Surprisingly  few  systems  are  actuallylinearizable in practice. For example, even RAM on a modern multi-core CPU is notlinearizable  [43]:  if  a  thread  running  on  one  CPU  core  writes  to  a  memory  address,and a thread on another CPU core reads the same address shortly afterward, it is notguaranteed  to  read  the  value  written  by  the  first  thread  (unless  a  memory  barrier  orfence [44] is used).
  1.  The  reason  for  this  behavior  is  that  every  CPU  core  has  its  own  memory  cache  and store  buffer.  
  2.  The  reason  fordropping linearizability is performance, not fault tolerance.
3. The same is true of many distributed databases that choose not to provide lineariza‐ ble guarantees: they do so primarily to increase performance, not so much for fault tolerance. Linearizability is slow—and this is true all the time, not only during a network fault
  1. Attiya and Welch prove that if you want linearizability,the response time of read and write requests is at least proportional to the uncertainty of  delays  in  the  network.

# Ordering Guarantees
1. The definition of linearizability implies that operations are executed in some well-defined order. It turns out that there are deep connections between ordering, linearizability, and consensus.
1. Ordering Examples
   * The main purpose of the leader in single-leader replication is to determine the order of writes in the replication log, that is, the order in which followers apply those writes.
   * Serializability is about ensuring that transactions behave as if they were executed in some sequential order. It can be achievedby literally executing transactions in that serial order, or by allowing concurrentexecution while preventing serialization conflicts (by locking or aborting). 
   * The use of timestamps and clocks in distributed systems, for  example  to  determinewhich one of two writes happened later.

## Ordering and Causality
1. There are several reasons why ordering keeps coming up, and one of the reasons is that it helps preserve causality.
2. Why casuality is important
  1. There's causal dependency between the question and the answer as in “Consistent Prefix Reads”
  2.  the  replicationbetween  three  leaders (multi-leader replication)  and  noticed  that  some  writes  could  “overtake”  others  dueto  network  delays.  From  the  perspective  of  one  of  the  replicas  it  would  look  asthough there was an update to a row that did not exist. Causality here means thata row must first be created before it can be updated
  3.  happened beforerelationshipis  another  expression  of  causality: If A and B are concur‐rent,  there  is  no  causal  link  between  them;  in  other  words,  we  are  sure  that  nei‐ther knew about the other.
  4. In the context of snapshot isolation for transactions, we said that a transaction reads from a consistent snapshot. But what does “consistent” mean in this context? It means consistent with causality: if the snapshot contains an answer, it must also contain the question being answered. Observing the entire database at a single point in timemakes it consistent with causality: the effects of all operations that happened cau‐sally  before  that  point  in  time  are  visible,  but  no  operations  that  happened  causally  afterward  can  be  seen.  Read  skew  (non-repeatable  reads,  as  illustrated  inFigure 7-6) means reading data in a state that violates causality.
  5. Write  skew  between  transactions  also demonstrated causal dependencies:Serializable snapshot isolation (see“Serializable Snapshot Isolation (SSI)” on page 261) detects write skew by track‐ing the causal dependencies between transactions.
3. Causality imposes an ordering on events: cause comes before effect
4. If a system obeys the ordering imposed by causality, we say that it is causally consistent. For example, snapshot isolation provides causal consistency: when you read from the database, and you see some piece of data, then you must also be able to see any data that causally precedes it (assuming it has not been deleted in the meantime).

### The causal order is not a total order
1. Unlike linearizability, which puts all operations in a single, totally ordered timeline, causality provides us with a weaker consistency model: some things can be concurrent, so the version history is like a timeline with branching and merging. Causal consistency does not have the coordi‐ nation overhead of linearizability and is much less sensitive to network problems
1. A total order allows any two elements to be compared, so if you have two elements, you can always say which one is greater and which one is smaller.
2. partially ordered: in some cases one set is greater than another (if one set contains all the elements of another), but in other cases they are incomparable.
3. Linearizability: In a linearizable system, we have a total order of operations. : if the system behavesas  if  there  is  only  a  single  copy  of  the  data,  and  every  operation  is  atomic,  thismeans  that  for  any  two  operations  we  can  always  say  which  one  happened  first.
4. Causality: We said that two operations are concurrent if neither happened before the other. Put another way, two events are ordered if they are causally related (one happened before the other), but they are incomparable if they are concurrent. This means that causality defines a partial order, not a total order: some operations are ordered with respect to each other, but some are incomparable. 
5. Therefore, according to this definition, there are no concurrent operations in a linearizable datastore.  there  must  be  a  single  timeline  along  which  all  operations  aretotally ordered.
6. Concurrency would mean that the timeline branches and merges again—and in this case, operations on different branches are incomparable (i.e., concurrent). Ex.git

### Linearizability is stronger than causal consistency
1. linearizability implies causality: any system that is linearizable will preserve causality correctly
2. Making a system linearizable can harm its performance and availability, especially if the system has significant network delays
3. The good news is that a middle ground is possible. Linearizability is not the only way of preserving causality—there are other ways too.
4. A system can be causally consistent without incurring the performance hit of making it linearizable (in particular, the CAP theorem does not apply). In fact, causal consistency is the strongest possible consistency model that does not slow down due to network delays, and remains available in the face of network failures
5. In many cases, systems that appear to require linearizability in fact only really require causal consistency, which can be implemented more efficiently.

### Capturing causal dependencies
1. In  order  to  maintain  causality,  you  need  to  know  which  operation  happened  beforewhich  other  operation.  This  is  a  partial  order:  concurrent  operations  may  be  pro‐cessed in any order, but if one operation happened before another, then they must beprocessed in that order on every replica.
2. In  order  to  determine  causal  dependencies,  we  need  some  way  of  describing  the“knowledge” of a node in the system. If a node had already seen the value X when itissued the write Y, then X and Y may be causally related. 
3. Causal  consis‐tency  goes  further:  it  needs  to  track  causal  dependencies  across  the  entire  database,not just for a single key. Version vectors can be generalized to do this 
4. In order to determine the causal ordering, the database needs to know which version of the data was read by the application

## Sequence Number Ordering
1. Although causality is an important theoretical concept, actually keeping track of all causal dependencies can become impractical.
2. However, there is a better way: we can use sequence numbers or timestamps (better come from a logical clock,  which  is  an  algorithm  to  generate  a  sequence  ofnumbers  to  identify  operations,  typically  using  counters  that  are  incremented  forevery operation.) to order events.
3. Such sequence numbers or timestamps are compact (only a few bytes in size), and they provide a total order: that is, every operation has a unique sequence number, and you can always compare two sequence numbers to determine which is greater (i.e., which operation happened later).
4. In particular, we can create sequence numbers in a total order that is consistent with causality. we promise that if operation A causally happened before B, then A occursbefore  B  in  the  total  order  (A  has  a  lower  sequence  number  than  B). Concurrentoperations  may  be  ordered  arbitrarily. 
5. Such a total order captures all the causality information, but also imposes more ordering than strictly required by causality.
  1. In a database with single-leader replication, the replication log defines a total order of write operations that is consistent with causality. The leader can simply increment a counter for each operation, and thus assign a monotonically increasing sequence number to each operation in the replication log.
  1. If a follower applies the writes in the order they appear in the replication log, the state of the follower is always causally consistent (even if it is lagging behind the leader).

### Noncausal sequence number generators
1. If there is not a single leader (perhaps because you are using a multi-leader or leader‐less database, or because the database is partitioned), it is less clear how to generate sequence numbers for operations.
   * Each node can generate its own independent set of sequence numbers. Put node id into the bits so never generate duplicate numbers
   * You can attach a timestamp from a time-of-day clock (physical clock) to each operation (Used in LWW conflict resolution)
   * You can preallocate blocks of sequence numbers for each node.
1. These three options all perform better and are more scalable than pushing all operations through a single leader that increments a counter.
2. However, they all have a problem: the sequence numbers they generate are not consistent with causality
3. The causality problems occur because these sequence number generators do not correctly capture the ordering of operations across different nodes:
   * Each node may process a different number of operations per second. Thus, If you have an odd-numbered operation and an even-numbered operation, you cannot accurately tell which one causally happened first.
   * Timestamps from physical clocks are subject to clock skew, which can make them inconsistent with causality
   * In the case of the block allocator, one operation may be given a sequence number in the range from 1,001 to 2,000, and a causally later operation may be given a number in the range from 1 to 1,000. Here, again, the sequence number is incon‐ sistent with causality.

### Lamport timestamps
1. Although the three sequence number generators just described are inconsistent with causality, there is actually a simple method for generating sequence numbers that is consistent with causality. It is called a Lamport timestamp
2. Each node has a unique identifier, and each node keeps a counter of the number of operations it has processed. The Lamport timestamp is then simply a pair of (counter, node ID).
3. it provides total ordering: if you have two timestamps, the one with a greater counter value is the greater timestamp; if the counter values are the same, the one with the greater node ID is the greater timestamp
4. The key idea about Lamport timestamps, which makes them consistent with causality, is the following: every node and every client keeps track of the maximum counter value it has seen so far, and includes that maximum on every request. When a node receives a request or response with a maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum.
  1. As long as the maximum counter value is carried along with every operation, this scheme ensures that the ordering from the Lamport timestamps is consistent with causality, because every causal dependency results in an increased timestamp
7. version vectors can distinguish whether two operations are concurrent or whether one is causally dependent on the other, whereas Lamport timestamps always enforce a total ordering.From the total ordering of Lamport time‐stamps, you cannot tell whether two operations are concurrent or whether they are causally dependent. The advantage of Lamport timestamps over version vectors is that they are more compact.

### Timestamp ordering is not sufficient
1. Lamport timestamps are not quite sufficient to solve many common problems in distributed systems
   * unique username constraint: 
      * However, it is not sufficient when a node has just received a request from a user to create a username, and needs to decide right now whether the request should succeed or fail. At that moment, the node does not know whether another node is concurrently in the process of creating an account with the same username, and what timestamp that other node may assign to the operation. 
      * In order to be sure that no other node is in the process of concurrently creating an account with the same username and a lower timestamp, you would have to check with every other node to see what it is doing.
      * The problem here is that the total order of operations only emerges after you have collected all of the operations. If another node has generated some operations, but you don’t yet know what they are, you cannot construct the final ordering of opera‐ tions: the unknown operations from the other node may need to be inserted at vari‐ ous positions in the total order.
      * To conclude: in order to implement something like a uniqueness constraint for user names, it’s not sufficient to have a total ordering of operations—you also need to know when that order is finalized. 
      * If one node is going to accept a registration, it needs to somehow know that another node isn’t concurrently in the process of registering the same name. This problem led us toward consensus.
      * This idea of knowing when your total order is finalized is captured in the topic of *total order broadcast*.

## Total Order Broadcast
1. In a distributed system, getting all nodes to agree on the same total ordering of operations is tricky. Ordering by timestamps or sequence numbers, is found that it is not as powerful as single-leader replication (if you use timestamp ordering to implement a uniqueness constraint, you cannot tolerate any faults).
2. Single-leader replication determines a total order of operations by choosing one node as the leader and sequencing all operations on a single CPU core on the leader. The challenge then is how to scale the system if the throughput is greater than a single leader can handle, and also how to handle failover if the leader fails. This problem is known as total order broadcast or atomic broadcast
3. Scope of ordering guarantee: Partitioned databases with a single leader per partition often main‐ tain ordering only per partition, which means they cannot offer consistency guarantees (e.g., consistent snapshots, foreign key ref‐ erences) across partitions. Total ordering across all partitions is possible, but requires additional coordination
4. Total order broadcast is usually described as a protocol for exchanging messages between nodes. Informally, it requires that two safety properties always be satisfied
   * Reliable delivery: No messages are lost: if a message is delivered to one node, it is delivered to all nodes.
   * Totally ordered delivery: Messages are delivered to every node in the same order

### Using total order broadcast
1. Consensus services such as ZooKeeper and etcd actually implement total order broadcast. This fact is a hint that there is a strong connection between total order broadcast and consensus.
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
4. To be precise, the procedure described here provides sequential consistency, sometimes also known as timeline consistency, a slightly weaker guarantee than linearizability.
### Implementing total order broadcast using linearizable storage
1. Note that unlike Lamport timestamps, the numbers you get from incrementing the linearizable register form a sequence with no gaps. in fact, this is the key difference between total order broadcast and timestamp ordering.
2. This is no coincidence: it can be proved that a linearizable compare-and-set (or increment-and-get) register and total order broadcast are both equivalent to consen‐ sus

# Distributed Transactions and Consensus
1. Consensus, informally, the goal is simply to get several nodes to agree on something. 
2. There are a number of situations in which it is important for nodes to agree.
   * Leader Election:  con‐sensus is important to avoid a bad failover, resulting in a split brain situation inwhich  two  nodes  both  believe  themselves  to  be  the  leader.
   * Atomic Commit: In a database that supports transactions spanning several nodes or partitions, we have the problem that a transaction may fail on some nodes but succeed on others. Either  they  all  abort/roll  back  (if  anything  goes  wrong)  or  they  allcommit  (if  nothing  goes  wrong). This instance of consensus is known as the atomic commit problem
1. The Impossibility of Consensus
   * FLP result proves that there is no algorithm that is always able to reach consensus if there is a risk that a node may crash.
   * The answer is that the FLP result is proved in the asynchronous system model (see “System Model and Reality” on page 306), a very restrictive model that assumes a deterministic algorithm that cannot use any clocks or timeouts.
   * Distributed systems can usually achieve consensus in practice

## Atomic Commit and Two-Phase Commit (2PC)
1. The most common way of solving atomic commit and which is implemented in various databases, messaging systems, and application servers. It turns out that 2PC is a kind of consensus algorithm—but not a very good one
2. Atomicity prevents failed transactions from littering the database with half-finished results and half-updated state. This is especially important for multi-object transactions and databases that maintain secondary indexes. Atomicity ensures that the secondary index stays consistent with the primary data.

### From single-node to distributed atomic commit
1. For transactions that execute at a single database node, atomicity is commonly implemented by the storage engine
2. When the client asks the database node to commit the transaction, the database makes the transaction’s writes durable (typically in a WAL) and then appends a commit record to the log on disk. If the database crashes in the middle of this process, the transaction is recovered from the log when the node restarts: if the commit record was successfully written to disk before the crash, the transaction is considered committed; if not, any writes from that transaction are rolled back.
3. Thus, on a single node, transaction commitment crucially depends on the order in which data is durably written to disk: first the data, then the commit record. The key deciding moment for whether the transaction commits or aborts is the moment at which the disk finishes writing the commit record: before that moment, it is still possible to abort (due to a crash), but after that moment, the transaction is committed (even if the database crashes). Thus, it is a single device (the controller of one par‐ ticular disk drive, attached to one particular node) that makes the commit atomic.
4. what if multiple nodes are involved in a transaction? 
   * it is not sufficient to simply send a commit request to all of the nodes and independently commit the transaction on each one
   * If some nodes commit the transaction but others abort it, the nodes become inconsistent with each other
1. A transaction commit must be irrevocable—you are not allowed to change your mind and retroactively abort a transaction after it has been committed. The reason for this rule is that once data has been committed, it becomes visible to other transac‐ tions, and thus other clients may start relying on that data; this principle forms the basis of read committed isolation.
### Introduction to two-phase commit
1. Two-phase commit is an algorithm for achieving atomic transaction commit across multiple nodes—i.e., to ensure that either all nodes commit or all nodes abort
2. It is a classic algorithm in distributed databases: 2PC is also made available to applications in the form of XA transactions (which are supported by the Java Transaction API, for example) or via WS-AtomicTransaction for SOAP web services
3. 2PC uses a new component that does not normally appear in single-node transactions: a coordinator (also known as transaction manager)
4.   The  coordinator  is  often implemented  as  a  library  within  the  same  application  process  that  is  requesting  thetransaction (e.g., embedded in a Java EE container), but it can also be a separate pro‐cess  or  service
5. When the application is ready to commit, the coordinator begins phase 1: it sends a prepare request to each of the nodes, asking them whether they are able to commit. The coordinator then tracks the responses from the participants. 
6. If all participants reply “yes,” indicating they are ready to commit, then the coor‐dinator  sends  out  a  commit  request  in  phase  2,  and  the  commit  actually  takesplace.
7. If any of the participants replies “no,” the coordinator sends an abort request toall nodes in phase 2.
8. When the application wants to begin a distributed transaction, it requests a transaction ID from the coordinator. This transaction ID is globally unique.

### A system of promises
1. the protocol contains two crucial “points of no return”: when a participant votes “yes,” it promises that it will definitely be able to commit later (although the coordinator may still choose to abort); and once the coordinator decides, that decision is irrevocable.
2. Once  the  coordinator’s  decision  has  been  written  to  disk,  the  commit  or  abortrequest is sent to all participants. If this request fails or times out, the coordinatormust retry forever until it succeeds. There is no more going back: if the decisionwas  to  commit,  that  decision  must  be  enforced,  no  matter  how  many  retries  ittakes.
### Coordinator failure
1. if any of the prepare requests fail or time out, the coordinator aborts the transaction; if any of the commit or abort requests fail, the coordinator retries them indefinitely
2. If the coordinator fails before sending the prepare requests, a participant can safely abort the transaction.
3. But once the participant has received a prepare request and voted “yes,” it can no longer abort unilaterally—it must wait to hear back from the coordinator whether the transaction was committed or aborted. If the coordinator crashes or the network fails at this point, the participant can do nothing but wait. A participant’s transaction in this state is called **in doubt or uncertain**
4. Even a timeout does not helphere: if database 1 unilaterally aborts after a timeout, it will end up inconsistent withdatabase  2,  which  has  committed.  Similarly,  it  is  not  safe  to  unilaterally  commit,because another participant may have aborted.
5. Without  hearing  from  the  coordinator,  the  participant  has  no  way  of  knowingwhether to commit or abort. In principle, the participants could communicate amongthemselves to find out how each participant voted and come to some agreement, butthat is not part of the 2PC protocol.
6. The only way 2PC can complete is by waiting for the coordinator to recover. This is why the coordinator must write its commit or abort decision to a transaction log on disk before sending commit or abort requests to participants: when the coordinator recovers, it determines the status of all in-doubt transactions by reading its transac‐ tion log. Any transactions that don’t have a commit record in the coordinator’s log are aborted. Thus, the commit point of 2PC comes down to a regular single-node atomic commit on the coordinator.
7. 
Three-phase
### Three-phase commit
1. Two-phase commit is called a blocking atomic commit protocol due to the fact that 2PC can become stuck waiting for the coordinator to recover
2. In general, nonblocking atomic commit requires a perfect failure detector— i.e., a reliable mechanism for telling whether a node has crashed or not
3. In a network with unbounded delay a timeout is not a reliable failure detector, because a request may time out due to a network problem even if no node has crashed.

## Distributed Transactions in Practice
1. 2PC are criticized for causing operational problems, killing performance, and promising more than they can deliver. 
2. Many cloud services choose not to implement distributed transactions due to the operational problems they engender
3. Some implementations of distributed transactions carry a heavy performance penalty. E.g. distributed  transactions  in  MySQL  are  reported  to  be  over  10  timesslower than single-node transactions
4. Much of the performance cost inherent in two-phase commit is due to the additional disk forcing (fsync) that is required for crash recovery, and the additional network round-trips
5. Two quite different types of distributed transactions are often conflated
   * Database-internal distributed transactions (easier): Some  distributed  databases  (i.e.,  databases  that  use  replication  and  partitioningin  their  standard  configuration)  support  internal  transactions  among  the  nodesof that database. For example, VoltDB and MySQL Cluster’s NDB storage enginehave such internal transaction support. In this case, all the nodes participating inthe transaction are running the same database software. 
   * Heterogeneous distributed transactions (harder): In  a  heterogeneous  transaction,  the  participants  are  two  or  more  different  tech‐nologies:  for  example,  two  databases  from  different  vendors,  or  even  non-database systems such as message brokers. A distributed transaction across thesesystems  must  ensure  atomic  commit,  even  though  the  systems  may  be  entirelydifferent under the hood

### Exactly-once message processing
1.  a message from a message queue can be acknowledgedas  processed  if  and  only  if  the  database  transaction  for  processing  the  message  was successfully  committed.  This  is  implemented  by  atomically  committing  the  messageacknowledgment  and  the  database  writes  in  a  single  transaction.
2.  Thus,  by  atomicallycommitting the message and the side effects of its processing, we can ensure that themessage is effectively processed exactly once, even if it required a few retries before itsucceeded. The abort discards any side effects of the partially completed transaction
3. Such a distributed transaction is only possible if all systems affected by the transaction are able to use the same atomic commit protocol, however.

### Atomic commit protocols
1.  Atomic  commit  protocol  that  allows  such  heterogeneous  distributedtransactions.

#### XA Transactions
1. X/Open XA (short for eXtended Architecture) is a standard for implementing two-phase commit across heterogeneous technologies
2. Supported Products: PostgreSQL, MySQL, DB2, SQL Server, and Oracle) and message brokers (including ActiveMQ, HornetQ, MSMQ, and IBM MQ
3. XA is not a network protocol—it is merely a C API for interfacing with a transaction coordinator
4. XA assumes that your application uses a network driver or client library to commu‐nicate with the participant databases or messaging services. If the driver supports XA,that  means  it  calls  the  XA  API  to  find  out  whether  an  operation  should  be  part  of  adistributed transaction—and if so, it sends the necessary information to the databaseserver.  The  driver  also  exposes  callbacks  through  which  the  coordinator  can  ask  theparticipant to prepare, commit, or abort.
5. The transaction coordinator implements the XA API
6. If the application process crashes, or the machine on which the application is running dies, the coordinator goes with it. Any participants with prepared but uncommitted transactions are then stuck in doubt. The database server cannot contact the coordinator directly,since all communication must go via its client library.
#### Holding locks while in doubt
1. Why do we care so much about a transaction being stuck in doubt? Can’t the rest of the system just get on with its work, and ignore the in-doubt transaction that will be cleaned up eventually?
2. The problem is with locking. data‐base transactions usually take a row-level exclusive lock on any rows they modify, toprevent  dirty  writes.  In  addition,  if  you  want  serializable  isolation,  a  database  usingtwo-phase  locking  would  also  have  to  take  a  shared  lock  on  any  rows  read  by  thetransaction
3.  The database cannot release those locks until the transaction commits or aborts. Therefore, when using two-phase commit, a transaction must hold onto the locks throughout the time it is in doubt.
#### Recovering from coordinator failure
1. In theory, if the coordinator crashes and is restarted, it should cleanly recover its statefrom  the  log  and  resolve  any  in-doubt  transactions. 
1. in practice, orphaned in-doubt transactions do occur [89, 90]—that is, transactions for which the coordina‐ tor cannot decide the outcome for whatever reason (e.g., because the transaction log has been lost or corrupted due to a software bug). These transactions cannot be resolved automatically, so they sit forever in the database, holding locks and blocking other transactions.
2. Even rebooting your database servers will not fix this problem, since a correct implementation of 2PC must preserve the locks of an in-doubt transaction even across restarts (otherwise it would risk violating the atomicity guarantee).
3. The only way out is for an administrator to manually decide whether to commit or roll back the transactions.
#### Limitations of distributed transactions
1. In particular, the key realization is that the transaction coordinator is itself a kind of database (in which transaction outcomes are stored), and so it needs to be approached with the same care as any other important database:
   * If the coordinator is not replicated but runs only on a single machine, it is a sin‐ gle point of failure for the entire system (since its failure causes other application servers to block on locks held by in-doubt transactions)
   * Many server-side applications are developed in a stateless model (as favored by HTTP), with all persistent state stored in a database, which has the advantage that application servers can be added and removed at will. However, when the coordinator is part of the application server, it changes the nature of the deployment. Suddenly, the coordinator’s logs become a crucial part of the durable sys‐ tem state—as important as the databases themselves, since the coordinator logs are required in order to recover in-doubt transactions after a crash. Such application servers are no longer stateless.
   * Since XA needs to be compatible with a wide range of data systems, it is necessar‐ily  a  lowest  common  denominator.  For  example,  it  cannot  detect  deadlocksacross  different  systems  (since  that  would  require  a  standardized  protocol  forsystems  to  exchange  information  on  the  locks  that  each  transaction  is  waitingfor), and it does not work with SSI (see “Serializable Snapshot Isolation (SSI)” onpage 261), since that would require a protocol for identifying conflicts across dif‐ferent systems
   * For  database-internal  distributed  transactions  (not  XA),  the  limitations  are  notso  great—for  example,  a  distributed  version  of  SSI  is  possible.  However,  thereremains  the  problem  that  for  2PC  to  successfully  commit  a  transaction,  all  par‐ticipants  must  respond.  Consequently,  if  any  part  of  the  system  is  broken,  thetransaction also fails. Distributed transactions thus have a tendency of amplifyingfailures, which runs counter to our goal of building fault-tolerant systems
   * for 2PC to successfully commit a transaction, all participants must respond. Consequently, if any part of the system is broken, the transaction also fails. Distributed. Distributed transactions thus have a tendency of amplifying failures, which runs counter to our goal of building fault-tolerant systems.


## Fault-Tolerant Consensus
1. The consensus problem is normally formalized as follows: one or more nodes may propose values, and the consensus algorithm decides on one of those values. This particular variant of consensus is called uniform consensus, which is equivalent to regular consensus in asynchronous systems with unreliable failure detector
2. a consensus algorithm must satisfy the following properties:
   * Uniform agreement: No two nodes decide differently. 
   * Integrity: No node decides twice. 
   * Validity: If a node decides value v, then v was proposed by some node. 
   * Termination: Every node that does not crash eventually decides some value.
1. The uniform agreement and integrity properties define the core idea of consensus: everyone decides on the same outcome, and once you have decided, you cannot change your mind
2. If you don’t care about fault tolerance, then satisfying the first three properties is easy: you can just hardcode one node to be the “dictator,” and let that node make all of the decisions. However, if that one node fails, then the system can no longer make any decisions. This is, in fact, what we saw in the case of two-phase commit: if the coordinator fails, in-doubt participants cannot decide whether to commit or abort
3. The termination property formalizes the idea of fault tolerance. It essentially says that a consensus algorithm cannot simply sit around and do nothing forever—in other words, it must make progress. Even if some nodes fail, the other nodes must still reach a decision. (Termination is a liveness property, whereas the other three are safety properties
4. The system model of consensus assumes that when a node “crashes,” it suddenly dis‐ appears and never comes back. In this system model, any algorithm that has to wait for a node to recover is not going to be able to satisfy the termination property. In particular, 2PC does not meet the requirements for termination.
5. There is a limit to the number of failures that an algorithm can tolerate: in fact, it can be proved that any consensus algorithm requires at least a majority of nodes to be functioning correctly in order to assure termination. That majority can safely form a quorum
6. Thus, the termination property is subject to the assumption that fewer than half of the nodes are crashed or unreachable. However, most implementations of consensus ensure that the safety properties—agreement, integrity, and validity—are always met, even if a majority of nodes fail or there is a severe network problem [92]. Thus, a large-scale outage can stop the system from being able to process requests, but it can‐ not corrupt the consensus system by causing it to make invalid decisions
7. Most consensus algorithms assume that there are no Byzantine faults,  That is, if a node does not correctly follow the proto‐col (for example, if it sends contradictory messages to different nodes), it may breakthe  safety  properties  of  the  protocol. 
### Consensus algorithms and total order broadcast
1. The best-known fault-tolerant consensus algorithms are Viewstamped Replication (VSR), Paxos, Raft, and Zab
2. Most algorithms  decide on a sequence of values, which makes them total order broadcast algorithms. They actually don’t directly use the formal model described here (proposing and deciding on a single value, while satisfying the agreement, integrity, validity, and termination properties).
3. Remember that total order broadcast requires messages to be delivered exactly once, in the same order, to all nodes. If you think about it, this is equivalent to performing several rounds of consensus: in each round, nodes propose the message that they want to send next, and then decide on the next message to be delivered in the total order. So, total order broadcast is equivalent to repeated rounds of consensus (each consen‐ sus decision corresponding to one message delivery)
4. Viewstamped Replication, Raft, and Zab implement total order broadcast directly, because that is more efficient than doing repeated rounds of one-value-at-a-time consensus. In the case of Paxos, this optimization is known as Multi-Paxos

### Single-leader replication and consensus
1. Isn’t this essentially total order broadcast? How come we didn’t have to worry about consensus?
2. The answer comes down to how the leader is chosen. 
   * If the leader is manually chosen and configured by the humans in your operations team, you essentially have a “consensus algorithm” of the dictatorial variety: only one node is allowed to accept writes (i.e., make decisions about the order of writes in the replication log) but it does not satisfy the termination property of consensus because it requires human intervention in order to make progress. and if that nodegoes  down,  the  system  becomes  unavailable  for  writes  until  the  operators  manuallyconfigure a different node to be the leader.
   * Some databases perform automatic leader election and failover, promoting a follower to be the new leader if the old leader fails. This brings us closer to fault-tolerant total order broadcast, and thus to solving consensus.
   * However, there is a problem. It seems that in order to elect a leader, we first need a leader. In order to solve con‐ sensus, we must first solve consensus. How do we break out of this conundrum?
### Epoch numbering and quorums
1. All of the consensus protocols discussed so far internally use a leader in some form or another, but they don’t guarantee that the leader is unique. Instead, they can make a weaker guarantee: the protocols define an epoch number (called the ballot number in Paxos, view number in Viewstamped Replication, and term number in Raft) and guarantee that within each epoch, the leader is unique.
2. Every time the current leader is thought to be dead, a vote is started among the nodes to elect a new leader. This election is given an incremented epoch number, and thusepoch numbers are totally ordered and monotonically increasing. If there is a conflictbetween  two  different  leaders  in  two  different  epochs  (perhaps  because  the  previousleader  actually  wasn’t  dead  after  all),  then  the  leader  with  the  higher  epoch  numberprevails.
3. Before a leader is allowed to decide anything, it must first check that there isn’t some other leader with a higher epoch number which might take a conflicting decision
4. How does a leader know that it hasn’t been ousted by another node? Recall “The Truth Is Defined by the Majority”: a node cannot necessarily trust its own judgment. Instead, it must collect votes from a quorum of nodes. For  every  decision  that  a  leader  wants  to  make,  it  must  sendthe proposed value to the other nodes and wait for a quorum of nodes to respond infavor of the proposal.
5. Thus, we have two rounds of voting: once to choose a leader, and a second time to vote on a leader’s proposal. The key insight is that the quorums for those two votes must overlap: if a vote on a proposal succeeds, at least one of the nodes that voted for it must have also participated in the most recent leader election.
6. Thus,  if  thevote  on  a  proposal  does  not  reveal  any  higher-numbered  epoch,  the  current  leadercan conclude that no leader election with a higher epoch number has happened, andtherefore  be  sure  that  it  still  holds  the  leadership.  It  can  then  safely  decide  the  pro‐posed value
7. This voting process looks superficially similar to two-phase commit. The biggest dif‐ ferences are that in 2PC the coordinator is not elected, and that fault-tolerant consensus algorithms only require votes from a majority of nodes, whereas 2PC requires a “yes” vote from every participant. Moreover, consensus algorithms define a recovery process by which nodes can get into a consistent state after a new leader is elected, ensuring that the safety properties are always met. These differences are key to the correctness and fault tolerance of a consensus algorithm
### Limitations of consensus
1. Consensus algorithms are a huge breakthrough for distributed systems: they bring concrete safety properties (agreement, integrity, and validity) to systems where every‐ thing else is uncertain, and they nevertheless remain fault-tolerant (able to make progress as long as a majority of nodes are working and reachable). They  provide  totalorder  broadcast,  and  therefore  they  can  also  implement  linearizable  atomic  opera‐tions in a fault-tolerant way
2. Nevertheless, they are not used everywhere, because the benefits come at a cost.
3. The process by which nodes vote on proposals before they are decided is a kind of synchronous replication. databases are often configured to use asynchronous replication.In  this  configuration,  some  committed  data  can  potentially  be  lost  on  failover—butmany people choose to accept this risk for the sake of better performance.
4. Consensus systems always require a strict majority to operate. This means you need a minimum of three nodes in order to tolerate one failure. If a network failure cuts off some nodes from the rest, only the majority portion of the network can make progress, and the rest is blocked
5. Most consensus algorithms assume a fixed set of nodes that participate in voting, which means that you can’t just add or remove nodes in the cluster. Dynamic mem‐ bership extensions to consensus algorithms are less understood.
6. Consensus systems generally rely on timeouts to detect failed nodes. frequent leader elections result in terrible performance because the system can end up spend‐ ing more time choosing a leader than doing any useful work.
7. Sometimes, consensus algorithms are particularly sensitive to network problems. 

## Membership and Coordination Services
1. Projects like ZooKeeper or etcd are often described as “distributed key-value stores” or “coordination and configuration services.” The API of such a service looks pretty much like that of a database: you can read and write the value for a given key, and iterate over keys. So
2. HBase, Hadoop YARN, OpenStack Nova, and Kafka all rely on ZooKeeper running in the background.
3. ZooKeeper and etcd are designed to hold small amounts of data that can fit entirely in memory (although they still write to disk for durability)—so you wouldn’t want to store all of your application’s data here. That small amount of data is replicated across all the nodes using a fault-tolerant total order broadcast (hence consensus) algorithm. As discussed previously, total order broadcast is just what you need for database replica‐ tion: if each message represents a write to the database, applying the same writes in the same order keeps replicas consistent with each other.
4. ZooKeeper is modeled after Google’s Chubby lock service [14, 98], implementing notonly total order broadcast (and hence consensus), but also an interesting set of otherfeatures that turn out to be particularly useful when building distributed systems: Of  these  features,  only  the  linearizable  atomic  operations  really  require  consensus.
   * Linearizable atomic operations (requires consensus): 
      * Using an atomic compare-and-set operation, you can implement a lock: if several nodes concurrently try to perform the same operation, only one of them will suc‐ ceed. The consensus protocol guarantees that the operation will be atomic and linearizable, even if a node fails or the network is interrupted at any point.
      * A distributed lock is usually implemented as a lease, which has an expiry time so that it is eventually released in case the client fails
   * Total ordering of operations: 
      * when some resource is protected by a lock or lease, you need a fencing token to prevent clients from con‐ flicting with each other in the case of a process pause. 
      * ZooKeeper provides this by totally ordering all operations and giving each operation a monotonically increasing transaction ID (zxid) and version number (cversion)
   * Failure detection
   * Change notifications
      * Not only can one client read locks and values that were created by another client, but it can also watch them for changes.
### Allocating work to nodes
1. if you have several instances of a process or service, and one of them needs to be chosen as leader or primary. If the leader fails, one of the other nodes should take over. This is of course useful for single-leader databases, but it’s also useful for job schedulers and similar stateful systems.
2. when you have some partitioned resource (database, message streams, file storage, distributed actor system, etc.) and need to decide which parti‐ tion to assign to which node. As new nodes join the cluster, some of the partitions need to be moved from existing nodes to the new nodes in order to rebalance the load (see “Rebalancing Partitions” on page 209). As nodes are removed or fail, other nodes need to take over the failed nodes’ work.
  1. These kinds of tasks can be achieved by judicious use of atomic operations, ephem‐ eral nodes, and notifications in ZooKeeper. If
4. An  application  may  initially  run  only  on  a  single  node,  but  eventually  may  grow  tothousands of nodes. Trying to perform majority votes over so many nodes would beterribly  inefficient.ZooKeeper runs on a fixed number of nodes (usually three or five) and performs its majority votes among those nodes while supporting a potentially large number of clients. Thus, ZooKeeper provides a way of “outsourcing” some of the work of coordinating nodes (consensus, operation ordering, and failure detection) to an external service.
  1. Normally, the kind of data managed by ZooKeeper is quite slow-changing: it repre‐ sents information like “the node running on 10.1.1.23 is the leader for partition 7,” which may change on a timescale of minutes or hours. It is not intended for storing the runtime state of the application, which may change thousands or even millions of times per second. If

### Service discovery
1. find out which IP address you need to connect to in order to reach a particular service. 
2. it is less clear whether service discovery actually requires consensus. 
  1. DNS is the traditional way of looking up the IP address for a service name, and it uses multi‐ ple layers of caching to achieve good performance and availability: Reads from DNS are absolutely not linearizable, and it is usually not considered problematic if the results from a DNS query are a little stale. It is more important that DNS is reliably available and robust to network interruptions.
  1. Although service discovery does not require consensus, leader election does.
  1. if your consensus system already knows who the leader is, then it can make sense to also use that information to help other services discover who the leader is. For  thispurpose, some consensus systems support read-only caching replicas. These replicasasynchronously receive the log of all decisions of the consensus algorithm, but do notactively  participate  in  voting.  They  are  therefore  able  to  serve  read  requests  that  donot need to be linearizable
### Membership services
1. A membership service determines which nodes are currently active and live members of a cluster.
2. A membership service determines which nodes are currently active and live membersof a cluster. As we saw throughout Chapter 8, due to unbounded network delays it’snot possible to reliably detect whether another node has failed. However, if you cou‐ple  failure  detection  with  consensus,  nodes  can  come  to  an  agreement  about  whichnodes should be considered alive or not.
3. For  example,  choosing  aleader  could  mean  simply  choosing  the  lowest-numbered  among  the  current  mem‐bers, but this approach would not work if different nodes have divergent opinions onwho the current members are.

# Summary
1. Linearizability:  its goal is to make replicated data appear as though there were only a single copy, and to make all operations act on it atomically. Although linearizability is appealing becauseit  is  easy  to  understand—it  makes  a  database  behave  like  a  variable  in  a  single-threaded  program—it  has  the  downside  of  being  slow,  especially  in  environmentswith large network delays.
2. Causality,  imposes  an  ordering  on  events  in  a  system  (what happened before what, based on cause and effect). Unlike linearizability, which puts all operations in a single, totally ordered timeline, causality provides us with a weaker consistency  model:  some  things  can  be  concurrent,  so  the  version  history  is  like  a timeline  with  branching  and  merging.  Causal  consistency  does  not  have  the  coordination overhead of linearizability and is much less sensitive to network problems.However,  even  if  we  capture  the  causal  ordering  (for  example  using  Lamport  time‐stamps),  we  saw  that  some  things  cannot  be  implemented  this  way:  in  “Timestampordering is not sufficient” on page 347 we considered the example of ensuring that a username is unique and rejecting concurrent registrations for the same username. If one  node  is  going  to  accept  a  registration,  it  needs  to  somehow  know  that  anothernode isn’t concurrently in the process of registering the same name. This problem led us toward consensus.
3. We  saw  that  achieving  consensus  means  deciding  something  in  such  a  way  that  allnodes  agree  on  what  was  decided,  and  such  that  the  decision  is  irrevocable.  Withsome  digging,  it  turns  out  that  a  wide  range  of  problems  are  actually  reducible  toconsensus  and  are  equivalent  to  each  other  (in  the  sense  that  if  you  have  a  solutionfor  one  of  them,  you  can  easily  transform  it  into  a  solution  for  one  of  the  others).Such equivalent problems includes:


# Equivalent Problems
1. a wide range of problems are actually reducible to consensus and are equivalent to each other
2. Linearizable compare-and-set registers
   * The register needs to atomically decide whether to set its value, based on whether its current value equals the parameter given in the operation. 
1. Atomic transaction commit 
   * A database must decide whether to commit or abort a distributed transaction.
1. Total order broadcast
   * The messaging system must decide on the order in which to deliver messages. 
1. Locks and leases
   * When several clients are racing to grab a lock or lease, the lock decides which one successfully acquired it.
1. Membership/coordination service
   * Given a failure detector (e.g., timeouts), the system must decide which nodes are alive, and which should be considered dead because their sessions timed out.
1. Uniqueness constraint
   * When several transactions concurrently try to create conflicting records with the same key, the constraint must decide which one to allow and which should fail with a constraint violation.
1. All of these are straightforward if you only have a single node, or if you are willing to assign the decision-making capability to a single node. This is what happens in a single-leader database: all the power to make decisions is vested in the leader, which is why such databases are able to provide linearizable operations, uniqueness constraints, a totally ordered replication log, and more.
1. However, if that single leader fails, or if a network interruption makes the leader unreachable, such a system becomes unable to make any progress. There are three ways of handling that situation:
   * Wait for the leader to recover, and accept that the system will be blocked in the meantime. Many XA/JTA transaction coordinators choose this option. This approach does not fully solve consensus because it does not satisfy the termina‐ tion property: if the leader does not recover, the system can be blocked forever.
   * Manually fail over by getting humans to choose a new leader node and reconfig‐ ure the system to use it. Many relational databases take this approach. he  speed  of  failover  is  limited  by  the  speed  at  whichhumans can act, which is generally slower than computers.
   * Use an algorithm to automatically choose a new leader. This approach requires a consensus algorithm, and it is advisable to use a proven algorithm that correctly handles adverse network conditions
1. Although a single-leader database can provide linearizability without executing a consensus algorithm on every write, it still requires consensus to maintain its leader‐ ship and for leadership changes. Thus, in some sense, having a leader only “kicks the can down the road”: consensus is still required, only in a different place, and less frequently. 
2. Nevertheless, not every system necessarily requires consensus: for example, leaderless and multi-leader replication systems typically do not use global consensus. The conflicts that occur in these systems (see “Handling Write Conflicts” on page 171) are a consequence of not having consensus across different leaders, but maybe that’s okay: maybe we simply need to cope without linearizability and learn to work better with data that has branching and merging version histories.
3. 

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

