
# Replication

## Replication Types 
### Leader-follower or Master-Slave
1. The master serves reads and writes, replicating writes to one or more slaves, which serve only reads. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned. 
   * Read on followers and write on leader
   * Improve on redundancy
   * DB Load Balancer to improve read performance: For read heavy service, the followers can handle read requests, the leader can handle write requests.
1. Disadvantage(s): master-slave replication
   * Additional logic is needed to promote a slave to a master.

### Leader-Leader or Master-Master
1. Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.
1. Multiple leaders to remove the single point of failur from one leader (after its failure and before a follower is promoted to leader)  
2. Disadvantage(s): master-master replication
   * You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
   * Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
   * Conflict resolution comes more into play as more write nodes are added and as latency increases.

### Tree-Replication
### Buddy Replication

## Replication Disadvantages
1. There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
2. Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
3. The more read slaves, the more you have to replicate, which leads to greater replication lag.
4. On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
5. Replication adds more hardware and additional complexity.


## Active-Passiv Replication
### Push: Active Replication

### Pull: Passive Replication
1. Data not available, read from peers, then store locally
2. Works well with time-out caches


## More Applications
3. We can have replication of network swtiches as well
4. Have the system in a different Data Centers
   * Need global load balancer (DNS) to direct the traffic to different DCs
   * Need to make sure you get sticky session 



