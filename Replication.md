1. Why we want to replicate data?
   * Keep data geographically close to your users (and thus reduce latency)
   * All the system to continue working even if some of its parts have failed (increase availability)
   * Scale out the number of machines that can serve read queries (increase read throughput)
1. All of the difficulty in replication lies in handling changes to replicated data
   * How do we ensure that all the data ends up on all the replicas? 
3. Trade-offs to consider with replication
   * Synchronous or asynchronous replication?
   * How to handle failed replicas? and How do you ensure that the new follower has an accurate copy of the leader’s data?

# Replication Types 
## Single-Leader (Active/Passive or Leader-follower or Master-Slave)
1. Write requests are sent to the leader, which first writes the new data to its local storage.
2. Whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of a *replication log or change stream*. 
3. Each follower takes the log from the leader and updates its local copy of the database accordingly, by applying all writes in the same order as they were processed on the leader.
4. When a client wants to read from the database, it can query either the leader or any of the followers.
5. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned. 
6. Disadvantage(s): master-slave replication
   * Additional logic is needed to promote a slave to a master.
7. Products
   * Relational DB: PostgreSQL, MySQL, Oracle Data Guard, SQL Server
   * Nonrelational DB: MongoDB, RethinkDB, Espresso
   * Distributed Message Brokers: Kafka, RabbitMQ highly available queues
   * Some network filesystems and replicated block devices such as DRBD

### Synchronous Versus Asynchronous Replication
#### Synchronous Replication
1. The advantage of synchronous replication is that the follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader.
2. The disadvantage is that if the synchronous follower doesn’t respond (because it has crashed, or there is a network fault, or for any other reason), the write cannot be processed. The leader must block all writes and wait until the synchronous replica is available again.
3. For that reason, it is impractical for all followers to be synchronous: any one node outage would cause the whole system to grind to a halt.
4. Semi-Synchronous Configuration: In practice, if you enable syn‐ chronous replication on a database, it usually means that one of the followers is syn‐ chronous, and the others are asynchronous. If the synchronous follower becomes unavailable or slow, one of the asynchronous followers is made synchronous. This guarantees that you have an up-to-date copy of the data on at least two nodes: the leader and one synchronous follower.

#### Asynchronous Replication
1. Often, leader-based replication is configured to be completely asynchronous.
1. If the leader fails and is not recoverable, any writes that have not yet been replicated to followers are lost. This means that a write is not guaranteed to be durable, even if it has been confirmed to the client. However, a fully asynchronous configuration has the advantage that the leader can continue processing writes, even if all of its followers have fallen behind.
2. Weakening durability may sound like a bad trade-off, but asynchronous replication is nevertheless widely used, especially if there are many followers or if they are geographically distributed.

### How to Set Up New Followers?
1. Simply copying data files from one node to another is typically not sufficient: clients are constantly writing to the database. so a standard file copy would see different parts of the database at different points in time.
2. You could make the files on disk consistent by locking the database (making it unavailable for writes), but that would go against our goal of high availability.
3. Solution
   * Take a consistent snapshot of the leader’s database at some point in time—if pos‐ sible, without taking a lock on the entire database.
   * Copy the snapshot to the new follower node.
   * The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken. This requires that the snapshot is associated with an exact position in the leader’s replication log.
      * log sequence number in PostgreSQL
      * binlog coordinates in MySQL
   * When the follower has processed the backlog of data changes since the snapshot, we say it has *caught up*. It can now continue to process data changes from the leader as they happen.

### Handling Node Outages
#### Follower Failure: Catch-up Recovery
1. On its local disk, each follower keeps a log of the data changes it has received from the leader.
2. from its log, it knows the last transaction that was processed before the fault occur‐ red. Thus, the follower can connect to the leader and request all the data changes that occurred during the time when the follower was disconnected.

#### Leader Failure: Failover
1. Failover Process: one of the followers needs to be promoted to be the new leader, clients need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader. 
2. Failover Process Steps
   * Determining that the leader has failed: 
      * Most systems use a timeout
   * Choosing a new leader (consensus problem)
      * Could be done through an election process
      * A new leader could be appointed by a previously elected *controller node*
   * Reconfiguring the system to use the new leader
      * The system needs to ensure that the old leader becomes a follower and recongnizes the new leader after it recovers
1. Problems with Failover
   * If asynchronous replication is used, the new leader may not have received all the writes from the old leader before it failed. If the former leader rejoins the cluster after a new leader has been chosen, what should happen to those writes? The new leader may have received conflicting writes in the meantime.
   * Discarding writes is especially dangerous if other storage systems outside of the database need to be coordinated with the database contents.
   * Split Brain: In certain fault scenarios, it could happen that two nodes both believe that they are the leader. it is dangerous: if both leaders accept writes, and there is no process for resolving conflicts, data is likely to be lost or corrupted.
      * Solution: Fencing or STONITH(Shoot The Other Node in the Head). But you may endup with both nodes being shut down.
   * What is the right timeout before the leader is declared dead?
      * A longer timeout means a longer time to recovery in the case where the leader fails. However, if the timeout is too short, there could be unnecessary failovers, which makes the situation worse in case of high load or network problem

## Multi-Leader (Leader-Leader or Master-Master)
1. Leader-based replication has one major downside: there is only one leader, and all writes must go through it. If the database is partitioned, each partition has one leader. Different partitions may have their leaders on different nodes, but each partition must nevertheless have one leader node.
1. Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.
2. In this setup, each leader simultaneously acts as a follower to the other leaders.
3. Multiple leaders to remove the single point of failur from one leader (after its failure and before a follower is promoted to leader)  
4. Disadvantage(s): master-master replication
   * You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
   * Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
   * Conflict resolution comes more into play as more write nodes are added and as latency increases.
### Use Cases for Multi-Leader Replication
1. It rarely makes sense to use a multi-leader setup within a single datacenter, because the benefits rarely outweigh the added complexity.
#### Multi-datacenter Operation
1. Within each datacenter, regular leader– follower replication is used; between datacenters, each datacenter’s leader replicates its changes to the leaders in other datacenters. 
##### multi- datacenter deployment
3. Performance
   * In a multi-leader configuration, every write can be processed in the local datacenter and is replicated asynchronously to the other datacenters.
   * Better than single-leader configuration
4. Tolerance of datacenter outages
   * More tolerant of the leader failure than single-leader configuration
5. Tolerance of network problems
   * A single-leader configu‐ ration is very sensitive to problems in this inter-datacenter link, because writes are made synchronously over this link. A multi-leader configuration with asyn‐ chronous replication can usually tolerate network problems better: a temporary network interruption does not prevent writes being processed.
1. Big Downside of multi-leader Configuration
   * Conflict Resolution: The same data may be concurrently modified in two different datacenters, and those write conflicts must be resolved
1. multi-leader replication is often considered danger‐ ous territory that should be avoided if possible 
#### Clients with offline Operation
1. if you have an application that needs to continue to work while it is disconnected from the internet
2. every device has a local database that acts as a leader (it accepts write requests), and there is an asynchronous multi-leader replication process (sync) between the replicas of your calendar on all of your devices. The replication lag may be hours or even days, depending on when you have internet access available.
#### Collaborative Editing
1. Google Doc/Wiki: When one user edits a document, the changes are instantly applied to their local replica (the state of the document in their web browser or client application) and asynchronously replicated to the server and any other users who are editing the same document.

### Handling Write Conflicts
1. A write conflict can be caused by two leaders concurrently updating the same record. This problem does not occur in a single-leader data‐ base.

#### Synchronous versus asynchronous conflict detection
1. In a multi-leader setup, both writes are successful, and the conflict is only detected asynchronously at some later point in time
3. If you want synchronous conflict detection, you might as well just use single-leader replication.
#### Conflict Avoidance
1.The simplest strategy for dealing with conflicts is to avoid them: if the application can ensure that all writes for a particular record go through the same leader, then con‐ flicts cannot occur. 
#### Converging toward a consistent state
1. In a multi-leader configuration, there is no defined ordering of writes, so it’s not clear what the final value should be.
2. If each replica simply applied writes in the order that it saw the writes, the database would end up in an inconsistent state
3. the database must resolve the conflict in a *convergent* way, which means that all replicas must arrive at the same final value when all changes have been replicated.
   * Give each write a unique ID, pick the write with the highest ID as the winner, and throw away the other writes. If a timestamp is used, this technique is known as **last write wins (LWW)**. It's prone to data loss
   * Give each replica a unique ID, and let writes that originated at a higher- numbered replica always take precedence over writes that originated at a lower- numbered replica. This approach also implies data loss.
   * Somehow merge the values together—e.g., order them alphabetically and then concatenate them
   * Record the conflict in an explicit data structure that preserves all information, and write application code that resolves the conflict at some later time

#### Custom conflict resolution logic
1. Note that conflict resolution usually applies at the level of an individual row or docu‐ ment, not for an entire transaction [36].
   * Execute On Write
   * Execute On Read
1. Automatic Conflict Resolution is hard

### Multi-Leader Replication Topologies
1. A replication topology describes the communication paths along which writes are propagated from one node to another.
   * Circular Topology: MySQL
   * Start Topology
   * All-to-All topology
1. A problem with circular and star topologies is that if just one node fails, it can inter‐ rupt the flow of replication messages between other nodes, causing them to be unable to communicate until the node is fixed.
2. all-to-all topologies can have issues too: some replication messages may “overtake” others. This is a problem of causality: Simply attaching a timestamp to every write is not sufficient, because clocks cannot be trusted to be sufficiently in sync to correctly order these events at leader 2
3. To order these events correctly, a technique called **version vectors** can be used

## Leaderless Replication (Buddy Replication)


## Tree-Replication

# Implementation of Replication Logs
## Statement-based Replication
1. the leader logs every write request (statement) that it executes and sends that statement log to its followers
2. Problems
   * Any statement that calls a nondeterministic function, such as NOW() to get the current date and time or RAND() to get a random number, is likely to generate a different value on each replica.
   * If statements use an autoincrementing column, or if they depend on the existing data in the database, they must be executed in exactly the same order on each replica, or else they may have a differ‐ ent effect.This can be limiting when there are multiple concurrently executing transactions.
   * Statements that have side effects (e.g., triggers, stored procedures, user-defined functions) may result in different side effects occurring on each replica, unless the side effects are absolutely deterministic.
1. by default MySQL now switches to row-based replication if there is any nondeterminism in a statement.

## Write-Ahead Log (WAL) Shippiing
1. the log is an append-only sequence of bytes containing all writes to the database.We can use the exact same log to build a replica on another node
2. PostgreSQL and Oracle use this method
3. The main disadvantage is that the log describes the data on a very low level: a WAL con‐ tains details of which bytes were changed in which disk blocks. 
4. This makes replication closely coupled to the storage engine. If the database changes its storage format from one version to another, it is typically not possible to run different versions of the database software on the leader and the followers. This may cause downtime durinig upgrades if not handled.

## Logic (row-based) log Replication
1. Logical log: Use different log formats for replication and for the storage engine, which allows the replication log to be decoupled from the storage engine internals.
2. A logical log for a relational database is usually a sequence of records describing writes to database tables at the granularity of a row
   * MySQL's binlog
3. Since a logical log is decoupled from the storage engine internals, it can more easily be kept backward compatible, allowing the leader and the follower to run different versions of the database software, or even different storage engines.
4. Change Data Capture: A logical log format is also easier for external applications to parse. This aspect is use‐ ful if you want to send the contents of a database to an external system, such as a data warehouse for offline analysis, or for building custom indexes and caches.

## Trigger-based Replication
1. use features that are available in many relational databases: triggers and stored procedures.
2. A trigger lets you register custom application code that is automatically executed when a data change (write transaction) occurs in a database system. The trigger has the opportunity to log this change into a separate table, from which it can be read by an external process. That
   * Databus for Oracle, Bucardo for Postgres

# Problems with Replication Lag

1. Read-Scaling architecture: For workloads that consist of mostly reads and only a small percentage of writes (a common pattern on the web), there is an attrac‐ tive option: create many followers, and distribute the read requests across those followers.
2. However, this approach only realistically works with asynchronous replication—if you tried to synchronously replicate to all followers, a single node failure or network outage would make the entire system unavailable for writing.
3. And the more nodes you have, the likelier it is that one will be down, so a fully synchronous configuration would be very unreliable.
4. Unfortunately, if an application reads from an asynchronous follower, it may see out‐ dated information if the follower has fallen behind. This leads to apparent inconsistencies in the database
5. eventual consistency: This inconsistency is just a temporary state—if you stop writing to the database and wait a while, the followers will eventually catch up and become consis‐ tent with the leader
6. Replication lag:the delay between a write happening on the leader and being reflected on a follower
   * if the system is operating near capacity or if there is a problem in the network, the lag can easily increase to several seconds or even minutes.

## Problems with reading from asynchronous followers

### Reading Your Own Writes
1. Read-after-write/Read-Your-Writes Consistency: This is a guarantee that if the user reloads the page, they will always see any updates they submitted themselves.
2. cross-device read-after-write consistency
   * Approaches that require remembering the timestamp of the user’s last update become more difficult, because the code running on one device doesn’t know what updates have happened on the other device.
   * If your replicas are distributed across different datacenters, there is no guarantee that connections from different devices will be routed to the same datacenter. If your approach requires reading from the leader, you may first need to route requests from all of a user’s devices to the same datacenter.

### Monotonic Reads
1. moving backward in time: This can happen if a user makes several reads from different replicas.
   * Example: , first to a follower with little lag, then to a follower with greater lag. (This scenario is quite likely if the user refreshes a web page, and each request is routed to a random server.)
1. Monotonic reads [23] is a guarantee that this kind of anomaly does not happen. 
   * When you read data, you may see an old value; monotonic reads only means that if one user makes several reads in sequence, they will not see time go backward— i.e., they will not read older data after having previously read newer data.
1. One way of achieving monotonic reads is to make sure that each user always makes their reads from the same replica.  (e.g. based on hash)

### Consistent Prefix Reads
1. consistent prefix reads: This guarantee says that if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order so not to violate causality.
2. This is a particular problem in partitioned (sharded) databases.
   * in many distributed databases, different partitions operate independently, so there is no global ordering of writes: when a user reads from the database, they may see some parts of the database in an older state and some in a newer state.
1. One solution is to make sure that any writes that are causally related to each other are written to the same partition—but in some applications that cannot be done effi‐ ciently.
2. There are also algorithms that explicitly keep track of causal dependencies

### Solutions for Replication Lag
1. When working with an eventually consistent system, it is worth thinking about how the application behaves if the replication lag increases to several minutes or even hours.
2. It would be better if application developers didn’t have to worry about subtle replica‐ tion issues and could just trust their databases to “do the right thing.”
3. This is why **transactions** exist: they are a way for a database to provide stronger guarantees so that the application can be simpler
4. 
# Replication Disadvantages
1. There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
2. Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
3. The more read slaves, the more you have to replicate, which leads to greater replication lag.
4. On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
5. Replication adds more hardware and additional complexity.


# Active-Passive Replication
## Push: Active Replication

## Pull: Passive Replication
1. Data not available, read from peers, then store locally
2. Works well with time-out caches


# More Applications
3. We can have replication of network swtiches as well
4. Have the system in a different Data Centers
   * Need global load balancer (DNS) to direct the traffic to different DCs
   * Need to make sure you get sticky session 



