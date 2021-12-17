
# Overview
## Purpose of Partitioning

1. Scalability: a large dataset can be distributed across many disks, and the query load can be distributed across many processors
   * Our goal with partitioning is to spread the data and the query load evenly across multiple machines, avoiding hot spots. 
      * It requires appropriate partitioning schema
      * It requires rebalancing the parititions when nodes are added/removed from the cluster

1. By design, every partition operates mostly independently—that’s what allows a partitioned database to scale to multiple machines. However, operations that need to write to several partitions can be difficult to reason about: for example, what happens if the write to one partition succeeds, but another fails?


## Problems to Consider
1. How do you decide which records to store on which nodes?
1. How to route requests to the right partitions and execute queries?
1. How to rebalance data when add or remove nodes?

## Comparison
Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
Products | MongoDB, Elasticsearch, SolrCloud, HBase, Bigtable, Cassandra, Riak, CouchBase | 3


# Partitioning and Replication
1. Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodes.
2. The choice of partitioning scheme is mostly independent of the choice of replication scheme.

# Partitioning of Key-Value Data
1. The presence of skew makes partitioning much less effective. A partition with disproportionately high load is called a **hot spot**
2. Avoid hot spot
   * Assign records to nodes randomly. But this has a big disadvantage: when you’re trying to read a particular item, you have no way of knowing which node it is on, so you have to query all nodes in parallel.
   * Assume Key-Value data model
      * Partitioning by Key Range: 
      * 

## Partitioning by Key Range
1. Assign a continuous range of keys (from some minimum to some maximum) to each partition. Within each partition, we can keep keys in sorted order
   * Can easily find the node for a key
   * range scans are easy if keys are sorted in each partition: SSTables and LSM-trees
   * certain access patterns can lead to hot spots
      * Modify the key, adds complexity for query
   * Partitions are typically rebalanced dynamically by splitting the range into two subranges when a partition gets too big.
1. Products: Bigtable, HBase, RethinkDB, and MongoDB before version 2.4
## Partitioning by Hash Key
1. A good hash function takes skewed data and makes it uniformly distributed
   * Programming languages' hash function may return different hash value in different processes for the same key, e.g. Java's Object.hashCode(). So not good.

1. Each partition handles a range of hashes
   * This technique is good at distributing keys fairly among the partitions. The partition boundaries can be evenly spaced, or they can be chosen pseudorandomly (in which case the technique is sometimes known as consistent hashing)
   * it destroys the ordering of keys, Can not do efficient range queries: has to query all partitions
   * Common to create a fixed number of partitions in advance, to assign several partitions to each node, and to move entire partitions from one node to another when nodes are added or removed. dynamic partitioning can also be used.
3. Products
   * Cassandra and MongoDB use MD5, and Voldemort uses the Fowler– Noll–Vo function
   * Cassandra achieves a compromise: 
      * A table in Cassandra can be declared with a compound primary key consisting of several columns. Only the first part of that key is hashed to determine the partition, but the other columns are used as a concatenated index for sorting the data in Cassandra’s SSTables
      * A query with fixed value of first column can perform an efficient range scan over the other columns of the key
      * The concatenated index approach enables an elegant data model for one-to-many relationships
### Consistent Hashing
1. It uses randomly chosen partition boundaries to avoid the need for central control or distributed consensus. Note
2. The consistent here describes a particular approach to rebalancing

### Skewed Workloads and Relieving Hot Spot
1. Partitioning by hash keys can’t avoid hotspot entirely: in the extreme case where all reads and writes are for the same key, you still end up with all requests being routed to the same partition
   * Ex. popular user
   * if one key is known to be very hot, a simple technique is to add a random number to the beginning or end of the key.
      * Any reads now have to do additional work, as they have to read the data from all 100 keys and combine it.
      * It requires additional bookkeeping: need keeping track of which keys are being split
## Hybrid Partitioning
1. Hybrid approaches are also possible, for example with a compound key: using one part of the key to identify the partition and another part for the sort order.


## Partitioning and Secondary Indexes
1. A secondary index usually doesn’t identify a record uniquely but rather is a way of searching for occurrences of a particular value
3. The problem with secondary indexes is that they don’t map neatly to partitions.
   * HBase and Voldemort have avoided secondary indexes
2. A secondary index also needs to be partitioned. 

### Partitioning Secondary Indexes by Document
1. A document-partitioned index is also known as a *local index*
   * In this indexing approach, each partition is completely separate: each partition maintains its own secondary indexes, covering only the documents in that partition. It doesn’t care what data is stored in other partitions. Whenever you need to write to the database—to add, remove, or update a document—you only need to deal with the partition that contains the document ID that you are writing.
   * Expensive scatter/gather read: reading from a document-partitioned index requires care: you need to send the query to all partitions, and combine all the results you get back
      * Prone to tail latency amplification
      * MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud, and VoltDB all use document-partitioned secondary indexes

### Partitioning Secondary Indexes by Term
1. we can construct a global index that covers data in all partitions. However, we can’t just store that index on one node, since it would likely become a bottleneck and defeat the pur‐ pose of partitioning. A global index must also be partitioned, but it can be partitioned differently from the primary key index
2. It's called term-partitioned index, because the term we’re looking for deter‐ mines the partition of the index
   * it can make reads more efficient
   * writes are slower and more complicated
   * updates to global secondary indexes are often asynchronous (that is, if you read the index shortly after a write, the change you just made may not yet be reflected in the index

# Rebalancing Partitions
1. The need of rebalancing
   * The query throughput increases, so you want to add more CPUs to handle the load.
   * The dataset size increases, so you want to add more disks and RAM to store it.
   * A machine fails, and other machines need to take over the failed machine’s responsibilities
3. The process of moving data and requests from one node in the cluster to another is called rebalancing
4. rebalancing is usually expected to meet some minimum requirements:
   * After rebalancing, the load (data storage, read and write requests) should be shared fairly between the nodes in the cluster.
   * Whille rebalancing is happening, the database should continue accepting reads and writes.
   * No more data than necessary should be moved between nodes, to make rebalanc‐ ing fast and to minimize the network and disk I/O load

## Strategies for Rebalancing
We need an approach that doesn’t move data around more than necessary.
### How not to do it: hash mod N
1. If the number of nodes N changes, most of the keys will need to be moved from one node to another
### Fixed number of partitions
1. Create many more partitions than there are nodes, and assign several partitions to each node. 
2. Now, if a node is added to the cluster, the new node can steal a few partitions from every existing node until partitions are fairly distributed once again. 
3. Only entire partitions are moved between nodes. The number of partitions does not change, nor does the assignment of keys to partitions. The only thing that changes is the assignment of partitions to nodes. 
4. This change of assignment is not immediate— it takes some time to transfer a large amount of data over the network—so the old assignment of partitions is used for any reads and writes that happen while the transfer is in progress.
5. This approach to rebalancing is used in Riak [15], Elasticsearch [24], Couchbase [10], and Voldemort
6. Choosing the right number of partitions is difficult if the total size of the dataset is highly variable. If partitions are very large, rebalancing and recovery from node failures become expensive. But if partitions are too small, they incur too much overhead.

### Dynamic Partitioning
1. For databases that use key range partitioning, a fixed number of partitions with fixed boundaries would be very inconvenient: if you got the boundaries wrong, you could end up with all of the data in one partition and all of the other partitions empty.
2. For that reason, key range–partitioned databases such as HBase and RethinkDB create partitions dynamically
3. When a partition grows to exceed a configured size (on HBase, the default is 10 GB), it is split into two partitions so that approximately half of the data ends up on each side of the split. Conversely, if lots of data is deleted and a partition shrinks below some threshold, it can be merged with an adjacent partition. This process is similar to what happens at the top level of a B-tree.
4. An advantage of dynamic partitioning is that the number of partitions adapts to the total data volume.
5. However, a caveat is that an empty database starts off with a single partition, since there is no a priori information about where to draw the partition boundaries. While the dataset is small—until it hits the point at which the first partition is split—all writes have to be processed by a single node while the other nodes sit idle.
6. Dynamic partitioning is not only suitable for key range–partitioned data, but can equally well be used with hash-partitioned data (MongoDB)

### Partitioning proportionally to nodes
1. In both of above strategies, the number of partitions is independent of the number of nodes.
2. make the number of partitions proportional to the number of nodes—in other words, to have a fixed number of partitions per node
3. When a new node joins the cluster, it randomly chooses a fixed number of existing partitions to split, and then takes ownership of one half of each of those split partitions while leaving the other half of each partition in place.
4. Picking partition boundaries randomly requires that hash-based partitioning is used (so the boundaries can be picked from the range of numbers produced by the hash function). Indeed, this approach corresponds most closely to the original definition of consistent hashing [7]
5. Cassandra (256 partitions per node by default) and Ketama

## Operations: Automatic or Manual Rebalancing
1. Rebalancing is an expensive operation, because it requires rerouting requests and moving a large amount of data from one node to another. If it is not done carefully, this process can overload the network or the nodes and harm the performance of other requests while the rebalancing is in progress.
2. Fully automation rebalancing can be dangerous in combination with automatic failure detection. For example, say one node is overloaded and is temporarily slow to respond to requests. The other nodes conclude that the overloaded node is dead, and automatically rebalance the cluster to move load away from it. This puts additional load on the overloaded node, other nodes, and the network—making the situation worse and potentially causing a cascading failure.
3. It's good to have human in the loop for rebalancing.

# Request Routing
1. When a client wants to make a request, how does it know which node to connect to?
## Service Discovery
### Approaches
1. Allow clients to contact any node (e.g., via a round-robin load balancer). If the partition is not in that node, forward the request to the appropriate node
2. Send all requests from clients to a routing tier first, which determines the node that should handle each request and forwards it accordingly. The routing tier only acts as a partition-aware load balancer
3. Require that clients be aware of the partitioning and the assignment of partitions to nodes. 

### How to Keep Updated on Partition Changes
1. In all cases, the key problem is: how does the component making the routing decision (which may be one of the nodes, or the routing tier, or the client) learn about changes in the assignment of partitions to nodes?
2. Consensus problem: it is important that all participants agree— otherwise requests would be sent to the wrong nodes and not handled correctly
3. Many distributed data systems rely on a separate coordination service (from routing tier, etc.) such as ZooKeeper to keep track of this cluster metadata
4. Each node registers itself in ZooKeeper, and ZooKeeper maintains the authoritative mapping of partitions to nodes. Other actors, such as the routing tier or the partitioning-aware client, can subscribe to this information in ZooKeeper. Whenever a partition changes ownership, or a node is added or removed, ZooKeeper notifies the routing tier so that it can keep its routing information up to date
5. HBase, SolrCloud, and Kafka also use ZooKeeper to track partition assignment. MongoDB has a similar architecture, but it relies on its own config server implemen‐ tation and mongos daemons as the routing tier
6. Cassandra and Riak take a different approach: they use a *gossip protocol* among the nodes to disseminate any changes in cluster state. 
   * Requests can be sent to any node, and that node forwards them to the appropriate node for the requested partition
7. When using a routing tier or when sending requests to a random node, clients still need to find the IP addresses to connect to. These are not as fast-changing as the assignment of partitions to nodes, so it is often sufficient to use DNS for this purpose.

## Parallel Query Execution
1. **Massively parallel processing (MPP)** relational database products, often used for analytics, are much more sophisticated in the types of queries they support. A typical data warehouse query contains several join, filtering, grouping, and aggregation operations. 
2. The MPP query optimizer breaks this complex query into a number of execution stages and partitions, many of which can be executed in parallel on different nodes of the database cluster.

