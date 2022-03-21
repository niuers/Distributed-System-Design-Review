
# Overview

1. All of the difficulty in replication lies in handling changes to replicated data
   * How do we ensure that all the data ends up the same on all the replicas? 

## Why we want to replicate data?
1. Reduce Latency: Keep data geographically close to your users
1. Increase Availability: Keeping the system running, even when one machine (or several machines, or an entire datacenter) goes down
1. Increase Read Throughput: Scale out the number of machines that can serve read queries on replicas
1. Disconnected operation: Allowing an application to continue working when there is a network interruption

## Important Trade-Offs to Consider with Replication
1. Synchronous or asynchronous replication?
1. How to handle failed leader and/or replicas? 
   * How do you ensure that the new follower has an accurate copy of the leader’s data?

## Comparisons of Various Replications

Item | Single-Leader | Multi-Leader | Leaderless
--- | --- | --- | ---
Stale Data | reads from followers might be stale | 1  |
Conflict Resolution | not needed | 1 |
Synchronous Replication | follower is guaranteed to have consistent data with leader <br /> leader has to block on write  | | 
Asynchronous Replication | most common <br /> faster <br /> no data durability in the case of leader failure during replicating to follower| | 
Complexity | Easiest | 3 | 4
Add New Replica | take snapshot and use replication log to catch up | 3 | 4
Handle  Down Follower | use replication log to catch up | 3 | 4
Handle  Down Leader | failover (consensus): <br /> 1. May lose writes from original leader <br /> 2. Split brain <br /> 3. How to choose timeout  | 3 | 4
Read From Asynchronous Followers | | | 
RDBMS Products| PostgreSQL, MySQL, Oracle Data Guard, SQL Server | 2 | 3
NoSQL Products| MongoDB, RethinkDB, Espresso | 2 | 3
Other Products| Kfaka, RabbitMQ| 2 | 3


## Replication Disadvantages
1. There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
2. Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
3. The more read slaves, the more you have to replicate, which leads to greater *replication lag*.
4. On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
5. Replication adds more hardware and additional complexity.


# Single-Leader (Active/Passive or Leader/follower or Master/Slave)
1. Clients send all writes to the leader node, who first writes the new data to its local storage. It then sends a stream of data change events to the other replicas as part of a *replication log or change stream*. Reads can be performed on any replica, but reads from followers might be stale.
1. Each follower takes the log from the leader and updates its local copy of the database accordingly, by applying all writes in the same order as they were processed on the leader.
1. When a client wants to read from the database, it can query either the leader or any of the followers.
1. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned. 
1. Advantages:
   * Easy to understand
   * No conflict resolution to worry about
6. Disadvantages:
   * Additional logic is needed to promote a slave to a master.
7. Products
   * Relational DB: PostgreSQL, MySQL, Oracle Data Guard, SQL Server
   * Nonrelational DB: MongoDB, RethinkDB, Espresso
   * Distributed Message Brokers: Kafka, RabbitMQ highly available queues
   * Some network filesystems and replicated block devices such as DRBD

## Synchronous Versus Asynchronous Replication
### Synchronous Replication
1. The advantage of synchronous replication is that the follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader.
2. The disadvantage is that if the synchronous follower doesn’t respond (crash, network fault etc.), the write cannot be processed. The leader must block all writes and wait until the synchronous replica is available again.
3. For that reason, it is impractical for all followers to be synchronous: any one node outage would cause the whole system to grind to a halt.
4. *Semi-Synchronous Configuration*: In practice, if you enable synchronous replication on a database, it usually means that one of the followers is synchronous, and the others are asynchronous. If the synchronous follower becomes unavailable or slow, one of the asynchronous followers is made synchronous. This guarantees that you have an up-to-date copy of the data on at least two nodes: the leader and one synchronous follower.

### Asynchronous Replication
1. leader-based replication is often configured to be completely asynchronous.
1. If the leader fails and is not recoverable, any writes that have not yet been replicated to followers are lost. *This means that a write is not guaranteed to be durable, even if it has been confirmed to the client*. However, a fully asynchronous configuration has the advantage that the leader can continue processing writes, even if all of its followers have fallen behind.
2. Weakening durability may sound like a bad trade-off, but asynchronous replication is nevertheless widely used, especially if there are many followers or if they are geographically distributed.

## Adding New Followers
1. Simply copying data files from one node to another is typically not sufficient: clients are constantly writing to the database. so a standard file copy would see different parts of the database at different points in time.
2. You could make the files on disk consistent by locking the database (making it unavailable for writes), but that would go against our goal of high availability.
3. Solution
   * Take a consistent snapshot of the leader’s database at some point in time—if possible, without taking a lock on the entire database.
   * Copy the snapshot to the new follower node.
   * The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken. This requires that the snapshot is associated with an exact position in the leader’s replication log.
      * log sequence number in PostgreSQL
      * binlog coordinates in MySQL
   * When the follower has processed the backlog of data changes since the snapshot, we say it has *caught up*. It can now continue to process data changes from the leader as they happen.

## Handling Node Outages
### Follower Failure: Catch-up Recovery
1. On its local disk, each follower keeps a log of the data changes it has received from the leader.
2. From its log, it knows the last transaction that was processed before the fault occurred. Thus, the follower can connect to the leader and request all the data changes that occurred during the time when the follower was disconnected.

### Leader Failure: Failover
1. Failover Process: one of the followers needs to be promoted to be the new leader, clients need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader. 
1. Failover Process Steps
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


## Implementation of Replication Logs
### Statement-based Replication
1. The leader logs every write request (statement) that it executes and sends that statement log to its followers
2. Problems
   * Any statement that calls a *nondeterministic function*, such as NOW() to get the current date and time or RAND() to get a random number, is likely to generate a different value on each replica.
   * If statements use an *autoincrementing column, or if they depend on the existing data in the database*, they must be executed in exactly the same order on each replica, or else they may have a different effect. This can be limiting when there are multiple concurrently executing transactions.
   * Statements that have *side effects* (e.g., triggers, stored procedures, user-defined functions) may result in different side effects occurring on each replica, unless the side effects are absolutely deterministic.
1. by default MySQL now switches to row-based replication if there is any nondeterminism in a statement.

### Write-Ahead Log (WAL) Shipping
1. The log is an append-only sequence of bytes containing all writes to the database. We can use the exact same log to build a replica on another node
  * PostgreSQL and Oracle use this method
1. The main disadvantage is that the log describes the data on a very low level: a WAL contains details of which bytes were changed in which disk blocks. 
1. This makes replication closely coupled to the storage engine. If the database changes its *storage format* from one version to another, it is typically not possible to run different versions of the database software on the leader and the followers. This may cause downtime durinig upgrades if not handled.

### Logic (row-based) log Replication
1. Logical log: Use different log formats for replication and for the storage engine, which allows the replication log to be decoupled from the storage engine internals.
2. A logical log for a relational database is usually a sequence of records describing writes to database tables at the granularity of a row
   * MySQL's binlog
3. Since a logical log is decoupled from the storage engine internals, it can more easily be kept backward compatible, allowing the leader and the follower to run different versions of the database software, or even different storage engines.
4. Change Data Capture: A logical log format is also easier for external applications to parse. This aspect is useful if you want to send the contents of a database to an external system, such as a data warehouse for offline analysis, or for building custom indexes and caches.

### Trigger-based Replication
1. Use features that are available in many relational databases: triggers and stored procedures.
2. A trigger lets you register custom application code that is automatically executed when a data change (write transaction) occurs in a database system. The trigger has the opportunity to log this change into a separate table, from which it can be read by an external process.
   * Databus for Oracle, Bucardo for Postgres

## Problems with Replication Lag
1. Read-Scaling architecture: For workloads that consist of mostly reads and only a small percentage of writes (a common pattern on the web), there is an attractive option: create many followers, and distribute the read requests across those followers.
2. However, this approach only realistically works with asynchronous replication—if you tried to synchronously replicate to all followers, a single node failure or network outage would make the entire system unavailable for writing. And the more nodes you have, the likelier it is that one will be down, so a fully synchronous configuration would be very unreliable.
4. Unfortunately, if an application reads from an asynchronous follower, it may see outdated information if the follower has fallen behind. This leads to apparent inconsistencies in the database
5. Eventual consistency: This inconsistency is just a temporary state—if you stop writing to the database and wait a while, the followers will eventually catch up and become consistent with the leader
6. Replication lag:the delay between a write happening on the leader and being reflected on a follower
   * if the system is operating near capacity or if there is a problem in the network, the lag can easily increase to several seconds or even minutes.

### Problems with reading from asynchronous followers

#### Read Your Own Writes
1. Read-after-write/Read-Your-Writes Consistency: This is a guarantee that if the user reloads the page, they will always see any updates they submitted themselves.
2. Cross-device read-after-write consistency
   * Approaches that require remembering the timestamp of the user’s last update become more difficult, because the code running on one device doesn’t know what updates have happened on the other device.
   * If your replicas are distributed across different datacenters, there is no guarantee that connections from different devices will be routed to the same datacenter. If your approach requires reading from the leader, you may first need to route requests from all of a user’s devices to the same datacenter.

#### Monotonic Reads
1. Moving backward in time: This can happen if a user makes several reads from different replicas.
   * Example: first to a follower with little lag, then to a follower with greater lag. (This scenario is quite likely if the user refreshes a web page, and each request is routed to a random server.)
1. Monotonic reads is a guarantee that After users have seen the data at one point in time, they shouldn’t later see the data from some earlier point in time. 
   * When you read data, you may see an old value; monotonic reads only means that if one user makes several reads in sequence, they will not see time go backward— i.e., they will not read older data after having previously read newer data.
1. One way of achieving monotonic reads is to make sure that each user always makes their reads from the same replica.  (e.g. based on hash)

#### Consistent Prefix Reads
1. Consistent prefix reads: This guarantee says that if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order so not to violate causality.
2. This is a particular problem in partitioned (sharded) databases.
   * In many distributed databases, different partitions operate independently, so there is no global ordering of writes: when a user reads from the database, they may see some parts of the database in an older state and some in a newer state.
1. One solution is to make sure that any writes that are causally related to each other are written to the same partition—but in some applications that cannot be done efficiently.
2. There are also algorithms that explicitly keep track of causal dependencies

#### Solutions for Replication Lag
1. When working with an eventually consistent system, it is worth thinking about how the application behaves if the replication lag increases to several minutes or even hours.
2. It would be better if application developers didn’t have to worry about subtle replication issues and could just trust their databases to “do the right thing.”
3. This is why **transactions** exist: they are a way for a database to provide stronger guarantees so that the application can be simpler.


# Multi-Leader (Leader-Leader or Master-Master)
1. Clients send each write to one of several leader nodes, any of which can accept writes. The leaders send streams of data change events to each other and to any follower nodes.
1. Leader-based replication has one major downside: there is only one leader, and all writes must go through it. If the database is partitioned, each partition has one leader. Different partitions may have their leaders on different nodes, but each partition must nevertheless have one leader node.
1. Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.
2. In this setup, each leader simultaneously acts as a follower to the other leaders.
3. Multiple leaders to remove the single point of failur from one leader (after its failure and before a follower is promoted to leader)  
4. Advantages:
   * More robust in the presense of faulty nodes, network interruptions, and latency spikes
6. Disadvantages:
   * You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
   * Most master-master systems provide either very weak  consistent (violating ACID) or have increased write latency due to synchronization.
   * Conflict resolution comes more into play as more write nodes are added and as latency increases.
1. Multi-leader replication is often a good choice for multi-datacenter replication

## Use Cases for Multi-Leader Replication
1. It rarely makes sense to use a multi-leader setup within a single datacenter, because the benefits rarely outweigh the added complexity.

### Multi-datacenter Operation
1. Within each datacenter, regular leader–follower replication is used; between datacenters, each datacenter’s leader replicates its changes to the leaders in other datacenters. 

#### Multi-datacenter deployment
1. Performance
   * In a multi-leader configuration, every write can be processed in the local datacenter and is replicated asynchronously to the other datacenters.
   * Better than single-leader configuration
4. Tolerance of datacenter outages
   * More tolerant of the leader failure than single-leader configuration
5. Tolerance of network problems
   * A single-leader configuration is very sensitive to problems in this inter-datacenter link, because writes are made synchronously over this link. A multi-leader configuration with asynchronous replication can usually tolerate network problems better: a temporary network interruption does not prevent writes being processed.

1. Big Downside of multi-leader Configuration
   * Conflict Resolution: The same data may be concurrently modified in two different datacenters, and those write conflicts must be resolved
1. multi-leader replication is often considered dangerous territory that should be avoided if possible 

### Clients with offline Operation
1. If you have an application that needs to continue to work while it is disconnected from the internet
2. Every device has a local database that acts as a leader (it accepts write requests), and there is an asynchronous multi-leader replication process (sync) between the replicas of your calendar on all of your devices. The replication lag may be hours or even days, depending on when you have internet access available.

### Collaborative Editing
1. Google Doc/Wiki: When one user edits a document, the changes are instantly applied to their local replica (the state of the document in their web browser or client application) and asynchronously replicated to the server and any other users who are editing the same document.

## Handling Write Conflicts
1. A write conflict can be caused by two leaders concurrently updating the same record. This problem does not occur in a single-leader database.

### Synchronous versus asynchronous conflict detection
1. In a multi-leader setup, both writes are successful, and the conflict is only detected asynchronously at some later point in time
3. If you want synchronous conflict detection, you might as well just use single-leader replication.

### Conflict Avoidance
1.The simplest strategy for dealing with conflicts is to avoid them: if the application can ensure that all writes for a particular record go through the same leader, then conflicts cannot occur. 

### Converging toward a consistent state
1. In a multi-leader configuration, there is no defined ordering of writes, so it’s not clear what the final value should be.
2. If each replica simply applied writes in the order that it saw the writes, the database would end up in an inconsistent state
3. The database must resolve the conflict in a *convergent* way, which means that all replicas must arrive at the same final value when all changes have been replicated.
   * Give each write a unique ID, pick the write with the highest ID as the winner, and throw away the other writes. If a timestamp is used, this technique is known as **last write wins (LWW)**. It's prone to data loss.
   * Give each replica a unique ID, and let writes that originated at a higher- numbered replica always take precedence over writes that originated at a lower- numbered replica. This approach also implies data loss.
   * Somehow merge the values together—e.g., order them alphabetically and then concatenate them
   * Record the conflict in an explicit data structure that preserves all information, and write application code that resolves the conflict at some later time

### Custom conflict resolution logic
1. Note that conflict resolution usually applies at the level of an individual row or document, not for an entire transaction.
   * Execute On Write
   * Execute On Read
1. Automatic Conflict Resolution is hard

## Multi-Leader Replication Topologies
1. A replication topology describes the communication paths along which writes are propagated from one node to another.
   * Circular Topology: MySQL
   * Star Topology
   * All-to-All topology
1. A problem with circular and star topologies is that if just one node fails, it can interrupt the flow of replication messages between other nodes, causing them to be unable to communicate until the node is fixed.
2. all-to-all topologies can have issues too: some replication messages may “overtake” others. This is a problem of causality: Simply attaching a timestamp to every write is not sufficient, because clocks cannot be trusted to be sufficiently in sync to correctly order these events at leader 2
3. To order these events correctly, a technique called **version vectors** can be used

# Leaderless Replication (Buddy Replication/Dynamo-Style)
1. Clients send each write to several nodes, and read from several nodes in parallel in order to detect and correct nodes with stale data.
1. Amazon Dynamo, Riak, Cassandra, Voldemort
1. In some leaderless implementations, the client directly sends its writes to several replicas, while in others, a coordinator node does this on behalf of the client. However, *unlike a leader database, that coordinator does not enforce a particular ordering of writes* (As compared with leader based configuration, A leader determines the order in which writes should be processed, and followers apply the leader’s writes in the same orde)
1. Advantages:
   * More robust in the presense of faulty nodes, network interruptions, and latency spikes


## Writing to the Database When a Node Is Down
1. In a leaderless configuration, failover does not exist
2. When a client reads from the database, it doesn’t just send its request to one replica: read requests are also sent to several nodes in parallel. Version numbers are used to determine which value is newer


### Read Repair and Anti-Entropy

1. The replication scheme should ensure that eventually all the data is copied to every replica. After an unavailable node comes back online, how does it catch up on the writes that it missed?
   * Read Repair: This approach works well for values that are frequently read.
   * Anti-Entropy: a background process that constantly looks for differences in the data between replicas and copies any missing data from one replica to another

### Quorums for reading and writing
1. If there are n replicas, every write must be confirmed by w nodes to be considered successful, and we must query at least r nodes for each read.
2. As long as w + r > n, we expect to get an up-to-date value when reading, because at least one of the r nodes we’re reading from must be up to date. Reads and writes that obey these r and w values are called **(strict) quorum reads and writes**
3. You can think of r and w as the minimum number of votes required for the read or write to be valid
4. A common choice is to make n an odd number (typically 3 or 5) and to set w = r = (n + 1) / 2 (rounded up).

## Limitations of Quorum Consistency
1. Often, r and w are chosen to be a majority (more than n/2) of nodes, because that ensures w + r > n while still tolerating up to n/2 node failures.
1. But quorums are not necessarily majorities—it only matters that the sets of nodes used by the read and write operations overlap in at least one node.
2. Only after the number of reachable replicas falls below w or r does the database become unavailable for writing or reading, respectively.
4. However, even with w + r > n, there are likely to be edge cases where *stale values are returned*.
5. Dynamo-style databases are generally optimized for use cases that can tolerate eventual consistency. The parameters w and r allow you to adjust the probability of stale values being read, but it’s wise to not take them as absolute guarantees.
6. In particular, you usually do not get the guarantees (reading your writes, monotonic reads, or consistent prefix reads), so the previously mentioned anomalies can occur in applications.

### Monitoring staleness
1. However, in systems with leaderless replication, there is no fixed order in which writes are applied, which makes monitoring more difficult.
2. Eventual consistency is a deliberately vague guarantee, but for operability it’s important to be able to quantify “eventual.”

## Sloppy Quorums and Hinted Handoff

1. Databases with appropriately configured quorums can tolerate the failure of individual nodes without the need for failover.
2. They can also tolerate individual nodes going slow, because requests don’t have to wait for all n nodes to respond—they can return when w or r nodes have responded.
3. These characteristics make databases with leaderless replication appealing for use cases that require high availability and low latency, and that can tolerate occasional stale reads.
4. In a large cluster (with significantly more than n nodes) it’s likely that the client can connect to some database nodes during the network interruption, just not to the nodes that it needs to assemble a quorum for a particular value. So?
   * Should we accept writes anyway, and write them to some nodes that are reachable but aren’t among the n nodes on which the value usually lives?
   * If so, this is known as *sloppy quorum*: writes and reads still require w and r successful responses, but those may include nodes that are not among the designated n “home” nodes for a value.
   * Hinted Handoff: Once the network interruption is fixed, any writes that one node temporarily accepted on behalf of another node are sent to the appropriate “home” nodes.

1. Sloppy quorums are particularly useful for increasing *write availability*: as long as any w nodes are available, the database can accept writes. However, you cannot be sure to read the latest value for a key, because the latest value may have been temporarily written to some nodes outside of n.
1. in Cassandra and Voldemort they are disabled by default

## Multi-Datacenter operation

1. Leaderless replication is also suitable for multi-datacenter operation, since it is designed to tolerate conflicting concurrent writes, network interruptions, and latency spikes.
1. Cassandra and Voldemort implement their multi-datacenter support within the normal leaderless model: the number of replicas n includes nodes in all datacenters, and in the configuration you can specify how many of the n replicas you want to have in each datacenter.



## Detecting Concurrent Writes in Multi-Leader/Leader-Less Configuration
1. The problem is that events may arrive in a different order at different nodes, due to variable network delays and partial failures.
2. We want to achieve eventually consistency: the replicas should converge toward the same value

### Last write wins or LWW (discarding concurrent writes)
1. Even though the writes don’t have a natural ordering, we can force an arbitrary order on them. Each replica need only store the most “recent” value and allow “older” values to be overwritten and discarded. 
1. LWW is the only supported conflict resolution method in Cassandra
2. LWW achieves the goal of eventual convergence, *but at the cost of durability*: if there are several concurrent writes to the same key, even if they were all reported as successful to the client (because they were written to w replicas), only one of the writes will survive and the others will be silently discarded. Moreover, LWW may even drop writes that are not concurrent
3. If losing data is not acceptable, LWW is a poor choice for conflict resolution
4. The only safe way of using a database with LWW is to ensure that a key is only written once and thereafter treated as immutable, thus avoiding any concurrent updates to the same key. For example, a recommended way of using Cassandra is to use a UUID as the key, thus giving each write operation a unique key

### The “happens-before” relationship and concurrency
1. How do we decide whether two operations are concurrent or not?
2. An operation A happens before another operation B if B knows about A, or depends on A, or builds upon A in some way.
3. We can simply say that two operations are concurrent if neither happens before the other (i.e., neither knows about the other).
4. Thus, whenever you have two operations A and B, there are three possibilities: either A happened before B, or B happened before A, or A and B are concurrent
5. For defining concurrency, exact time doesn’t matter: we simply call two operations concurrent if they are both unaware of each other, regardless of the physical time at which they occurred.

### Capturing the happens-before relationship
1. ???
2. The server can determine whether two operations are concurrent by looking at the version numbers—it does not need to interpret the value itself
3. When a write includes the version number from a prior read, that tells us which previous state the write is based on. If you make a write without including a version number, it is concurrent with all other writes, so it will not overwrite anything—it will just be returned as one of the values on subsequent reads.

### Merging concurrently written values
1. if several operations happen concurrently, clients have to clean up afterward by merging the concurrently written values.

### Version Vectors
1. we need to use a version number per replica as well as per key. Each replica increments its own version number when processing a write, and also keeps track of the version numbers it has seen from each of the other replicas. This information indicates which values to overwrite and which values to keep as siblings.
2. The collection of version numbers from all the replicas is called a version vector
3. version vectors are sent from the database replicas to clients when values are read, and need to be sent back to the database when a value is subsequently written. (Riak encodes the version vector as a string that it calls causal context.) The version vector allows the database to distinguish between overwrites and concurrent writes.

# Active-Passive Replication
## Push: Active Replication
## Pull: Passive Replication
1. Data not available, read from peers, then store locally
2. Works well with time-out caches

# More Applications
1. We can have replication of network swtiches as well
1. Have the system in a different Data Centers
   * Need global load balancer (DNS) to direct the traffic to different DCs
   * Need to make sure you get sticky session 
