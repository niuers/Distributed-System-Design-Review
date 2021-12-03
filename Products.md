# Storm
1. Hadoop is stateful


# BigTable
1. Bigtable: Bigtable is a distributed storage system for managing structured data that is designed to scale to a very large size
2. 

# ZooKeeper
1. Coordination Service
   * Group membership
   * Locking
   * Publisher/Subscriber
   * Leader Election
   * Synchronization

1. Zookeeper allows distributed processes to coordinate with each other to through a shared hierarchial namespace of data registers.
2. 

# HBase
1. Non-relational distributed column-oriented database modeled after BigTable
2. Think of it as a sparse, consistent, distributed, multi-dimensional, sorted map
   * labeled table of rows
   * row consists of key-value cells: (row key, column family, column, timestamp) -> value
1. What is HBase Not?
   * Not an SQL DB
   * Not relational
   * No joins
   * No fancy query language and no sophisticated query engine
   * No transactions out of the box
   * No secondary index out of the box
## Features
1. Linear scalability: capable of storing hundreds of terabytes of data
2. Automatic and configurable sharding of tables
   * Tables partitioned into Regions 
   * Region defined by start & end row keys 
   * Regions are the “atoms” of distribution 
   * Regions are assigned to RegionServers (HBase cluster slaves)
4. Automatic failover support
   * DataNode failures handled by HDFS (replication)
   * RegionServers failures (incl. caused by whole server failure) handled automatically: Master re-assignes Regions to available RSs 
   * HMaster failover: automatic with multiple HMasters
6. strictly consistent reads and writes
7. Provide real-time random read/write access to data stored in HDFS
8. Integrates nicely with Hadoop MapReduce (both as source and destination) 
9. Easy Java API for client access 
10. Thrift gateway and REST APIs 
11. Bulk import of large amount of data 
12. Replication across clusters & backup options Block cache and Bloom filters for real-time queries

### Column Oriented
1. [Although HBase is known to be a column oriented database (where the column data stay together)](http://www.thecloudavenue.com/2012/01/is-hbase-really-column-oriented.html), the data in HBase for a particular row stay together and the column data is spread and not together. So it's more like 'column-family oriented'.
2. In HBase, the cell data in a table is stored as a key/value pair in the HFile and the HFile is stored in HDFS.
## Data
1. Row keys are uninterpreted byte arrays
2. Cell is uniterpreted byte array and a timestamp, it can have multiple versions
3. Rows are sorted and accessed by row keys
4. Data can be sparse
### Write
1. Row updates are atomic
2. Updates across multiple rows are not atomic, no transaction support out of box
3. HBase stores N versions of a cell (default 3) 
4. Tables are usually “sparse”, not all columns populated in a row

### Read
1. Reader will always read the last written (and committed) values 
2. Reading single row: Get 
3. Reading multiple rows: Scan (very fast) 
   * Scan usually defines start key and stop key 
   * Rows are ordered, easy to do partial key scan Row Key 
   * Example Data: ‘login_2012-03-01.00:09:17’ d:{‘user’:‘alex’} ... ... ‘login_2012-03-01.23:59:35’ d:{‘user’:‘otis’} ‘login_2012-03-02.00:00:21’ d:{‘user’:‘david’} 
   * Query predicate pushed down via server-side Filters
### Integration with MapReduce
1. MapReduce Integration Out of the box integration with Hadoop MapReduce
2. Data from HBase table can be source for MR job 
3. MR job can write data into HBase 
4. MR job can write data into HDFS directly and then output files can be very quickly loaded into HBase via “Bulk Loading” functionality


## Comparison
1. One thing which was very clear is that Cassandra is way simpler to setup than HBase, since Cassandra is self contained. HBase depends on HDFS for storage, which is still evolving a bit complex. 
2. Another thing is that Cassandra is decentralized and there is no SPOF (Single Point Of Failure), while in HBase the HDFS Name Node and HBase Master are SPOF.
3. What HBase is good at
   * Serving large amount of data: built to scale from the get-go 
   * fast random access to the data 
   * Write-heavy applications: clients should handle the loss of HTable client-side buffer
   * Append-style writing (inserting/ overwriting new data) rather than heavy read-modify-write operations** 
   * ** see https://github.com/sematext/HBaseHUT
1. HBase vs ... 
   * Favors consistency over availability 
   * Part of a Hadoop ecosystem 
   * Great community; adopted by tech giants like Facebook, Twitter, Yahoo!, Adobe, etc.
1. Use-cases 
   * Audit logging systems 
      * track user actions 
      * answer questions/queries like: 
         * what are the last 10 actions made by user? row key: userId_timestamp 
         * which users logged into system yesterday? row key: action_timestamp_userId
   * Real-time analytics, OLAP 
      * real-time counters 
      * interactive reports showing trends, breakdowns, etc 
      * time-series databases
   * Messages-centered systems 
      * twitter-like messages/statuses 
   * Content management systems: serving content out of HBase 
   * Canonical use-case: webtable (pages stored during crawling the web)
## [HDFS vs. HBase](https://www.geeksforgeeks.org/difference-between-hdfs-and-hbase/)
1. HDFS: Hadoop Distributed File System is a distributed file system designed to store and run on multiple machines that are connected to each other as nodes and provide data reliability. It consists of clusters, each of which is accessed through a single NameNode software tool installed on a separate machine to monitor and manage the that cluster’s file system and user access mechanism.
2. HBase: HBase is a top-level Apache project written in java which fulfills the need to read and write data in real-time. It provides a simple interface to the distributed data. It can be accessed by Apache Hive, Apache Pig, MapReduce, and store information in HDFS.
3. Differences
   * HDFS is a java based file distribution system,	Hbase is hadoop database that runs on top of HDFS
   * HDFS is highly fault-tolerant and cost-effective,	HBase is partially tolerant and highly consistent
   * HDFS Provides only sequential read/write operation,	HBase Random access is possible due to hash table
   * HDFS is based on write once read many times,	HBase supports random read and write operation into filesystem
   * HDFS has a rigid architecture,	HBase support dynamic changes
   * HDFS is prefereable for offline batch processing,	HBase is preferable for real time processing
   * HDFS provides high latency for access operations.	HBase provides low latency access to small amount of data

# Cassandra
1. Definition: Apache Cassandra is an open source, distributed, decentralized, elastically scalable, highly available, fault-tolerant, tuneably consistent, column-oriented key-value database that bases its distribution design on Amazon’s Dynamo and its data model on Google’s Bigtable. Created at Facebook, it is now used at some of the most popular sites on the Web
## Features
1. Distributed and Decentralized 
   * Distributed: Capable of running on multiple machines
   * Decentralized: No single point of failure: No master-slave issues due to peer-to-peer architecture (protocol "gossip") 
   * Single Cassandra cluster may run across geographically dispersed data centers: Read- and write requests to any node
1. Elastic Scalability
   * Cassandra scales horizontally, adding more machines that have all or some of the data on 
   * Adding of nodes increase performance throughput linearly 
   * De-/ and increasing the nodecount happen seamlessly: Linearly scales to terabytes and petabytes of data
1. High Availability and Fault Tolerance
   * High Availability? 
      * Multiple networked computers operating in a cluster 
      * Facility for recognizing node failures 
      * Forward failing over requests to another part of the system 
   * Cassandra has High Availability No single point of failure due to the peer-to-peer architecture
1. Tunable Consistency 
   * Choose between strong and eventual consistency 
   * Adjustable for read-and write operations separately 
   * Conflicts are solved during reads, as focus lies on write-performance 
   * Use case dependent level of consistency

1. When do we have strong consistency? 
   * Simple Formula: (nodes_written + nodes_read) > replication_factor 
   * Ensures that a read always reflects the most recent write 
   * If not: Weak consistency
      * Eventually consistent

1. Column-oriented Key-Value Store 
   * Data is stored in sparse multidimensional hash tables 
   * A row can have multiple columns – not necessarily the same amount of columns for each row 
   * Each row has a unique key, which also determines partitioning 
   * No relations! Stored sorted by row key 
   * Stored sorted by column key/value Map<RowKey, SortedMap<ColumnKey, ColumnValue>> 
   * Row keys (partition keys) should be hashed, in order to distribute data across the cluster evenly

1. CQL – An SQL-like query interface
   * “CQL 3 is the default and primary interface into the Cassandra DBMS” 
   * Familiar SQL-like syntax that maps to Cassandras storage engine and simplifies data modelling
   * “SQL-like” but NOT relational SQL

1. High Performance 
   * Optimized from the ground up for high throughput 
   * All disk writes are sequential, append only operations 
   * No reading before writing 
   * Cassandra`s threading-concept is optimized for running on multiprocessor/ multicore machines 
   * Optimized for writing, but fast reads are possible as well

# MongoDB
1. An open source,  high performance,   document oriented  database
2. NoSQL:
   * no  joins + no  complex  transactions:
      *  Horizontally Scalable Architectures
      *  New data models
1. Data Models
   * Key  /  Value: memcached,  Dynamo 
   * Tabular: BigTable 
   * Document  Oriented: MongoDB,  CouchDB,  JSON  stores
## Features
1. JSON style documents represented as BSON
2. Flexible schema
3. Dynamic queries
4. Atomic Update modifiers
5. Focus on Performance
6. Replication
7. Auto-sharding: 
8. Best Use Case: scaling out, caching, high volume
9. Less good at: 
   * highly transactional
   * ad-hoc  business  intelligence 
   * problems  that  require  SQL
## Special Key
1. A Quick Aside _id special  key present  in  all  documents unique  across  a  Collection any  type  you  want
2. In MongoDB, each document stored in a collection requires a unique _id field that acts as a primary key. If an inserted document omits the _id field, the MongoDB driver automatically generates an ObjectId for the _id field.

This also applies to documents inserted through update operations with upsert: true.

The _id field has the following behavior and constraints:

By default, MongoDB creates a unique index on the _id field during the creation of a collection.
The _id field is always the first field in the documents. If the server receives a document that does not have the _id field first, then the server will move the field to the beginning.

# Memcached
1. 
