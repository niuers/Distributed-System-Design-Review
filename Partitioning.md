
# Overview
## Purpose of Partitioning

1. Scalability: a large dataset can be distributed across many disks, and the query load can be distributed across many processors
   * Our goal with partitioning is to spread the data and the query load evenly across nodes.
3. 

## Problems to Consider
1. How do you decide which records to store on which nodes?
1. How to route requests to the right partitions and execute queries?
1. 1. How to rebalance data when add or remove nodes?

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
   * Assign records to nodes randomly. Bt has a big disadvantage: when you’re trying to read a particular item, you have no way of knowing which node it is on, so you have to query all nodes in parallel.
   * Assume Key-Value data model
      * Partitioning by Key Range: 
      * 

## Partitioning by Key Range
1. Assign a continuous range of keys (from some minimum to some maximum) to each partition. Within each partition, we can keep keys in sorted order
   * Can easily find the node for a key
   * range scans are easy if keys are sorted in each partition: SSTables and LSM-trees
   * certain access patterns can lead to hot spots
      * Modify the key, adds complexity for query
1. Products: Bigtable, HBase, RethinkDB, and MongoDB before version 2.4
## Partitioning by Hash Key
1. A good hash function takes skewed data and makes it uniformly distributed
   * Programming languages' hash function may return different hash value in different processes for the same key, e.g. Java's Object.hashCode(). So not good.
1. Each partition handles a range of hashes
   * This technique is good at distributing keys fairly among the partitions. The partition boundaries can be evenly spaced, or they can be chosen pseudorandomly (in which case the technique is sometimes known as consistent hashing)
   * Can not do efficient range queries: has to query all partitions
3. Products
   * Cassandra and MongoDB use MD5, and Voldemort uses the Fowler– Noll–Vo function
   * Cassandra achieves a compromise: 
      * A table in Cassandra can be declared with a compound primary key consisting of several columns. Only the first part of that key is hashed to determine the partition, but the other columns are used as a concatenated index for sorting the data in Cassandra’s SSTables
      * A query with fixed value of first column can perform an efficient range scan over the other columns of the key
      * The concatenated index approach enables an elegant data model for one-to-many relationships
### Consistent Hashing
1. It uses randomly chosen partition boundaries to avoid the need for central control or distributed consensus. Note
2. The consistent here describes a particular approach to rebalancing

## Skewed Workloads and Relieving Hot Spot
1. Partitioning by hash keys can’t avoid hotspot entirely: in the extreme case where all reads and writes are for the same key, you still end up with all requests being routed to the same partition
   * Ex. popular user
   * if one key is known to be very hot, a simple technique is to add a random number to the beginning or end of the key.
      * Any reads now have to do addi‐ tional work, as they have to read the data from all 100 keys and combine it.
      * It requires additional bookkeeping: need keeping track of which keys are being split

# Partitioning and Secondary Indexes
1. A secondary index usually doesn’t identify a record uniquely but rather is a way of searching for occurrences of a particular value
2. The problem with secondary indexes is that they don’t map neatly to partitions.
   * HBase and Voldemort have avoided secondary indexes
   * 
## Partitioning Secondary Indexes by Document
1. A document-partitioned index is also known as a local index
   * In this indexing approach, each partition is completely separate: each partition main‐ tains its own secondary indexes, covering only the documents in that partition. It doesn’t care what data is stored in other partitions. Whenever you need to write to the database—to add, remove, or update a document—you only need to deal with the partition that contains the document ID that you are writing.
   * requires expensive scatter/gather read: reading from a document-partitioned index requires care: you need to send the query to all partitions, and combine all the results you get back
      * Prone to tail latency amplification
      * MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud, and VoltDB all use document-partitioned secondary indexes

## Partitioning Secondary Indexes by Term
1. we can construct a global index that covers data in all partitions. However, we can’t just store that index on one node, since it would likely become a bottleneck and defeat the pur‐ pose of partitioning. A global index must also be partitioned, but it can be partitioned differently from the primary key index
2. It's called term-partitioned index, because the term we’re looking for deter‐ mines the partition of the index
   * it can make reads more efficient
   * writes are slower and more complicated
   * updates to global secondary indexes are often asynchronous (that is, if you read the index shortly after a write, the change you just made may not yet be reflected in the index

# Rebalancing Partitions
1. The process of moving load from one node in the cluster to another is called rebalancing
2. rebalancing is usually expected to meet some minimum requirements:
   * After rebalancing, the load (data storage, read and write requests) should be shared fairly between the nodes in the cluster.
   * Whille rebalancing is happening, the database should continue accepting reads and writes.
   * No more data than necessary should be moved between nodes, to make rebalanc‐ ing fast and to minimize the network and disk I/O load

# Request Routing
