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
2. ZooKeeper is much more than a distributed lock server
3. ZooKeeper is an open source, high-performance coordination service for distributed applications. 
4. Exposes common services in simple interface: 
   * naming 
   * configuration management 
   * locks & synchronization 
   * group services

## Use Cases
1. Configuration Management 
   * Cluster member nodes bootstrapping configuration from a centralized source in unattended way 
   * Easier, simpler deployment/provisioning 
1. Distributed Cluster Management 
   * Node join / leave 
   * Node statuses in real time 
1. Naming service – e.g. DNS 
2. Distributed synchronization - locks, barriers, queues 
3. Leader election in a distributed system. 
4. Centralized and highly reliable (simple) data registry
## ZooKeeper Service
1. ZooKeeper Service is replicated over a set of machines 
2. All machines store a copy of the data (in memory) 
3. A leader is elected on service startup 
4. Clients only connect to a single ZooKeeper server & maintains a TCP connection. 
5. Client can read from any Zookeeper server, writes go through the leader & needs majority consensus

## Data Model
1. ZooKeeper has a hierarchal name space. 
2. Each node in the namespace is called as a ZNode. 
3. Every ZNode has data (given as byte[]) and can optionally have children. 
5. ZNode paths: 
   * canonical, absolute, slash-separated 
   * no relative references. 
   * names can have Unicode characters
7. Znode
   * Maintain a stat structure with version numbers for data changes, ACL (access control list) changes and timestamps. 
   * Version numbers increases with changes 
   * Data is read and written in its entirety
## Persistent Node
1. Persistent Nodes exists till explicitly deleted 
2. Ephemeral Nodes exists as long as the session is active, they can’t have children 
3. Sequence Nodes (Unique Naming) 
   * append a monotonically increasing counter to the end of path 
   * applies to both persistent & ephemeral nodes

1. Clients can set watches on znodes
   * NodeChildrenChanged
   * NodeCreated
   * NodeDataChanged 
   * Node Deleted 
1. Changes to a znode trigger the watch and ZooKeeper sends the client a notification. 
2. Watches are one time triggers. 
3. Watches are always ordered. 
4. Client sees watched event before new znode data. 
5. Client should handle cases of latency between getting the event and sending a new request to get a watch
6. API methods are sync as well as async Sync

## Read/Write
1. Read requests are processed locally at the ZooKeeper server to which the client is currently connected 
2. Write requests are forwarded to the leader and go through majority consensus before a response is generated

## Consistency
1. Sequential Consistency: Updates are applied in order 
2. Atomicity: Updates either succeed or fail 
3. Single System Image: A client sees the same view of the service regardless of the ZK server it connects to. 
4. Reliability: Updates persists once applied, till overwritten by some clients. 
5. Timeliness: The clients’ view of the system is guaranteed to be up-to-date within a certain time bound. (Eventual Consistency)

## Applications
### Cluster Management
1. Each Client Host i, i:=1 .. N 
   * Watch on /members 
   * Create /members/host-${i} as ephemeral nodes 
   * Node Join/Leave generates alert 
   * Keep updating /members/host-${i} periodically for node status changes (load, memory, CPU etc.)
### Leader Election
1. A znode, say “/svc/election-path" 
1. All participants of the election process create an ephemeral-sequential node on the same election path. 
2. The node with the smallest sequence number is the leader. 
3. Each “follower” node listens to the node with the next lower seq. number 
4. Upon leader removal go to election-path and find a new leader, or become the leader if it has the lowest sequence number. 
5. Upon session expiration check the election state and go to election if needed 
6. http://techblog.outbrain.com/2011/07/leader-election-with-zookeeper/

### Distributed Exclusive Lock
1. Assuming there are N clients trying to acquire a lock 
2. Clients creates an ephemeral, sequential znode under the path /Cluster/_locknode_ 
3. Clients requests a list of children for the lock znode (i.e. _locknode_)  
4. The client with the least ID according to natural ordering will hold the lock. 
5. Other clients sets watches on the znode with id immediately preceding  its own id 
6. Periodically checks for the lock in case of notification. 
7. The client wishing to release a lock deletes the node, which triggering the next client in line to acquire the lock
## Lesson Learned
1. Watches are one time triggers 
2. Continuous watching on znodes requires reset of watches after every events / triggers 
3. Too many watches on a single znode creates the “herd effect” - causing bursts of traffic and limiting scalability 
4. If a znode changes multiple times between getting the event and setting the watch again, carefully handle it!
5. Keep session time-outs long enough to handle long garbage-collection pauses in applications. 
6. Set Java max heap size correctly to avoid swapping. Dedicated disk for ZooKeeper transaction log

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
## Normal Cache
1. The anatomy
   * simple key/value storage
   * simple operations: save/ get/ delete

1. Terminology 
   * storage cost
   * retrieval cost (network load / algorithm load)
   * invalidation (keeping data up to date / removing irrelevant data)
   * replacement policy (FIFO/LFU/LRU/MRU/RANDOM vs. Belady’s algorithm)
   * cold cache / warm cache
   * cache hit and cache miss
   * typical stats:
      * hit ratio (hits / hits + misses)
      * miss ratio (1 - hit ratio) 
      * 45 cache hits and 10 cache misses: 45/(45+10) = 82% hit ratio, 18% miss ratio
1. When to cache?
   * caches are only efficient when the beneﬁts of faster access outweigh the overhead of checking and keeping your cache up to date
   * more cache hits than cache misses
3. Caches in the web stack
   * Browser cache
   * DNS cache
   * Content Delivery Networks (CDN)
   * Proxy servers
   * Application level
      * full output caching plugin) (eg. Wordpress WP-Cache • ...
      * opcode cache (APC)
      * query cache (MySQL)
      * storing denormalized results in the database
      * object cache
      * storing values in php objects/classes
1. Efficiency of caching?
   * the earlier in the process, the closer to the original request(er), the faster
      * browser cache will be faster then cache on a proxy 
   * but probably also the harder to get it right: the closer to the requester the more parameters the cache depends on
1. What to cache on the server-side? As PHP backend developer, what to cache?
   * expensive operations: operations that work with slower resources • database access • reading ﬁles(in fact, any ﬁlesystem access) • API calls • Heavy computations • XML
1. Where to cache on the server-side? As PHP backend developer, where to store cache results? 
   * in database (computed values, generated html) 
      * you’ll still need to access your database
   * in static ﬁles (generated html or serialized php values) 
      * you’ll still need to access your ﬁle system
   * in memory
## About Memcached
1. About memcached 
    * Free & open source, high-performance, distributed memory object caching system 
    * Generic in nature, intended for use in speeding up dynamic web applications by alleviating database load. 
    * key/value dictionary
1. Technically
   * It’s a server 
   * Client access over TCP or UDP 
   * Servers can run in pools
      * eg. 3 servers with 64GB mem each give you a single pool of 192GB storage for caching 
      * Servers are independent, clients manage the pool
1. What to store in memcache? 
   * high demand (used often) 
   * expensive (hard to compute) 
   * common (shared accross users) 
   * Best? All three
1. Typically stored in memcache
   * user sessions (often) 
   * user data (often, shared) 
   * homepage data (eg. often, shared, expensive)
### Memcached principles
1. Fast network access (memcached servers close to other application servers) 
2. No persistency (if you server goes down, data in memcached is gone)
3. No redundancy / fail-over
4. No replication (single item in cache lives on one server only) 
5. No authentication (not in shared environments)
1. 1 key is maximum 1MB 
2. keys are strings of 250 characters (in application typically MD5 of user readable string) 
3. No enumeration of keys (thus no list of valid keys in cache at certain moment, list of keys beginnen with “user_”, ...) 
4. No active clean-up (only clean up when more space needed, LRU)

### Output Caching vs. Data Caching
1. Output caching 
   * Pages with high load / expensive to generate 
   * Very easy 
   * Very fast 
   * But: all the dependencies ... • language, css, template, logged in user’s details

1. Data caching 
   * on a lower level 
   * easier to ﬁnd all dependencies 
   * ideal solution for offloading database queries 
   * the database is almost always the biggest bottleneck in backend performance problems

1. “There are only two hard things in Computer Science: cache invalidation and naming things.” Phil Karlton

### Cache Invalidation
1. Caching for a certain amount of time 
   * eg. 10 minutes 
   * don’t delete caches 
   * thus: You can’t trust that data coming from cache is correct
   * Use: Great for summaries 
      * Overview 
      * Pages where it’s not that big a problem if data is a little bit out of date (eg. search results) 
   * Good for quick and dirty optimizations
1. Store forever, and expire on certain events 
   * the userdata example 
      * store userdata for ever 
      * when user changes any of his preferences, throw cache away
   * Use: 
      * data that is fetched more then it’s updated 
      * where it’s critical the data is correct 
      * Improvement: instead of delete on event, update cache on event. (Mind: race conditions. Cache invalidation always as close to original change as possible!)
1. Query Caching
   * queries with JOIN and WHERE statements are harder to cache 
      * often not easy to ﬁnd the cache key on update/change events 
      * solution: JOIN in PHP
### Multi-Get Optimisations 
1. We reduced database access 
2. Memcached is faster, but access to memcache still has it’s price 
3. Solution: multiget 
   * fetch multiple keys from memcached in one single call 
   * result is array of items

### Consistent Hashing 
1. client is responsible for managing pool 
   * hashes a certain key to a certain server 
3. clients can be naïve: distribute keys on size of pool 
   * if one server goes down, all keys will now be queried on other servers > cold cache 
   * use a client with consistent hashing algorithms, so if server goes down, only data on that server gets lost
### memcached isn’t the only caching solution 
1. memcachedb (persistent memcached) 
2. opcode caching 
  3. APC (php compiled code cache, usable for other purposes too) 
  4. xCache 
  5. eAccelerator 
  6. Zend optimizer

### Last thought
1. main bottleneck in php backends is database 
   * adding php servers is easier then scaling databases 
3. A complete caching layer before your database layer solves a lot of performance and scalability issues 
   * But being able to scale takes more then memcached 
   * performance tuning, beginning with identifying the slowest and most used parts
# Redis
1. Redis is: 
   * Memcache-ish in-memory key/value store, 
   * But it's also persistent! 
   * And it also has very cool value types: 
   * ○ lists ○ sets ○ sorted sets ○ hash tables ○ appendable buffers

## Key Features
1. All data is in memory (almost) 
2. All data is eventually persistent (But can be immediately) 
3. Handles huge workloads easily 
4. Mostly O(1) behavior 
5. Ideal for write-heavy workloads 
6. Support for atomic operations 
7. Supports for transactions 
8. Has pub/sub functionality 
9. Tons of client libraries for all major languages 
10. Single threaded, uses aync. IO 
11. Internal scripting with LUA

## Scaling it up 
1. Master-slave replication out of the box 
2. Slaves can be made masters on the fly 
3. Currently does not support "real" clustered mode.... 
4. ... But Redis-Cluster to be released soon 
5. You can manually shard it client side 
6. Single threaded - run num_cores/2 instances on the same machine

## Persistence 
1. All data is synchronized to disk - eventually or immediately 
2. Pick your risk level Vs. performance 
3. Data is either dumped in a forked process, or written as a append-only change-log (AOF) 
4. Append-only mode supports transactional disk writes so you can lose no data (cost: 99% speed loss :) ) 
5. AOF files get huge, but redis can minimize them on the fly. 
6. You can save the state explicitly, background or blocking 
7. Default configuration: 
   * Save after 900 sec (15 min) if at least 1 key changed 
   * Save after 300 sec (5 min) if at least 10 keys changed 
   * Save after 60 sec if at least 10000 keys changed

## Virtual Memory 
1. If your database is too big - redis can handle swapping on its own. 
2. Keys remain in memory and least used values are swapped to disk. 
3. Swapping IO happens in separate threads 
4. But if you need this - don't use redis, or get a bigger machine

## Features
1. Get/Set/Incr - strings/numbers
   * Keys are strings, anything goes - just quote spaces
   * You can atomically increment numbers
   * Keys are lazily expired: Be careful with EXPIRE - re-setting a value without re-expiring it will remove the expiration
1. Atomic Operations
   * Atomic Operations GETSET puts a different value inside a key, retriving the old one
   * SETNX sets a value only if it does not exist
   * SETNX + Timestamp => Named Locks
1. Binary-safe strings means that values are essentially byte strings, and can contain any byte (including the null byte). This is possible because the Redis protocol uses pascal-style strings, prefixing any string with its length in bytes.
3. Lists 
   * Lists are your ordinary linked lists. 
   * You can push and pop at both sides, extract range, resize, etc. 
   * Random access and ranges at O(N)
   * BLPOP: Blocking POP - wait until a list has elements and pop them. Useful for realtime stuff.
5. Sets
   * sets of unique values w/ push, pop, etc. 
   * Sets can be intersected/diffed /union'ed server side. 
   * Can be useful as keys when building complex schemata
7. Sorted Sets
   * Same as sets, but with score per element 
   * Ranked ranges, aggregation of scores on INTERSECT 
   * Can be used as ordered keys in complex schemata 
   * Think timestamps, inverted index, geohashing, ip ranges
9. Hash Tables 
   * Hash tables as values 
   * Think of an object store with atomic access to object members
11. PubSub
   * Clients can subscribe to channels or patterns and receive notifications when messages are sent to channels. 
   * Subscribing is O(1), posting messages is O(n) 
   * Think chats, Comet applications: real-time analytics, twitter 
13. SORT 
   * Keep in mind that it's blocking and redis is single threaded. Maybe put a slave aside if you have big SORTs
15. Transactions 
   * MULTI, ...., EXEC: Easy because of the single thread. 
   * All commands are executed after EXEC, block and return values for the commands as a list. 
   * Transactions can be discarded with DISCARD. 
   * WATCH allows you to lock keys while you are queuing your transaction, and avoid race conditions
1. Lessons Learned 
   * emory fragmentation can be a problem with some usage patterns. Alternative allocators (jemalloc, tcmalloc) ease that. 
   * 64 bit instances consume much much more RAM. 
   * Master/Slave sync far from perfect

# HDFS

# Apache Kafka
1. Apache Kafka is publish-subscribe messaging rethought as a distributed commit log
   * Kafka is a persistent, distributed, replicated pub/sub messaging system. 
   * Publishers send messages to a cluster of brokers. 
   * The brokers persist the messages to disk. 
   * Consumers then request a range of messages using an (offset, length) style API. 
   * Use of NIO (Java New IO) FileChannel allows for very fast transfer of data in and out of the system.
## Features
1. Use ZooKeeper for Broker coordination 
2. Configurable Producer acks 
3. Consumer Groups 
4. TTL persistence 
5. Sync/Async producer API 
6. Durable 
7. Scalable 
8. Fast

## Motivation
1. Activity stream processing (user interactions) 
2. Batch latency was too high 
3. Existing queues handle large volumes of (unconsumed) data poorly - durability is expensive 
4. Need something fast and durable

## Key design choices 
1. Pub/sub messaging pattern 
2. Messages are persistent and thus can be used for batched consumption such as ETL, in addition to real time applications
3. Everything is distributed - producers, brokers, consumers, the queue itself that's easy to scale out
4. Consumers maintain their own state (i.e., "dumb" brokers) 
5. Throughput is key
6. It supports multi-subscribers and automatically balances the consumers during failure.


## Architecture
### Brokers 
1. Receive messages from Producers (push), deliver messages to Consumers (pull) 
2. Responsible for persisting the messages for some time 
3. Relatively lightweight - mostly just handling TCP connections and keeping open file handles to the queue files

### Log-based queue 
1. Messages are persisted to append-only log files by the broker. 
2. Producers are appending to these log files (sequential write), and consumers are reading a range of these files (sequential reads).

### Topics 
1. Topics are queues. 
2. They are logical collections of partitions (the physical files). A broker contains some of the partitions for a topic

### Replication 
1. Benefits
   * A producer can continue to publish messages during failure and it can choose between latency and durability, depending on the application.
   * A consumer continues to receive the correct messages in real time, even when there is failure.
1. Application of CAP Theorem 
   * Our goal was to support replication in a Kafka cluster within a single datacenter, where network partitioning is rare, so our design focuses on maintaining highly available and strongly consistent replicas. 
   * Strong consistency means that all replicas are byte-to-byte identical, which simplifies the job of an application developer.
3. [Partitions of a topic are replicated](https://engineering.linkedin.com/kafka/intra-cluster-replication-apache-kafka)
   * In Kafka, a message stream is defined by a topic, divided into one or more partitions. Replication happens at the partition level and each partition has one or more replicas.
   * The replicas are assigned evenly to different servers (called brokers) in a Kafka cluster. Each replica maintains a log on disk. Published messages are appended sequentially in the log and each message is identified by a monotonically increasing offset within the log.
   * The offset is logical concept within a partition. Given an offset, the same message can be identified in each replica of the partition. When a consumer subscribes to a topic, it keeps track of an offset in each partition for consumption and uses it to issue fetch requests to the broker.
   * When a producer publishes a message to a partition in a topic, the message is first forwarded to the leader replica of the partition and is appended to its log. The follower replicas keep pulling new messages from the leader. Once enough replicas have received the message, the leader commits it.
   * One subtle issue is how the leader decides what's enough. The leader can't always wait for writes to complete on all replicas. This is because any follower replica can fail and the leader can't wait indefinitely.
   * To address this problem, for each partition of a topic, we maintain an in-sync replica set (ISR). This is the set of replicas that are alive and have fully caught up with the leader (note that the leader is always in ISR). When a partition is created initially, every replica is in the ISR. When a new message is published, the leader waits until it reaches all replicas in the ISR before committing the message. If a follower replica fails, it will be dropped out of the ISR and the leader then continues to commit new messages with fewer replicas in the ISR. Notice that now, the system is running in an under replicated mode.
   * The leader also maintains a high watermark (HW), which is the offset of the last committed message in a partition. The HW is continuously propagated to the followers and is checkpointed to disk in each broker periodically for recovery.
   * When a failed replica is restarted, it first recovers the latest HW from disk and truncates its log to the HW. This is necessary since messages after the HW are not guaranteed to be committed and may need to be thrown away. Then, the replica becomes a follower and starts fetching messages after the HW from the leader. Once it has fully caught up, the replica is added back to the ISR and the system is back to the fully replicated mode.











5. One broker is the "leader" of a partition. 
6. All writes and reads must go to the leader. 
7. Replicas exist for fault-tolerance, not scalability. 
8. When writing, messages can be synchronously written to N replicas (depending on the producer's ACKiness)

#### Strongly consistent replicas
1. In the literature, there are two typical approaches of maintaining strongly consistent replicas. Both require one of the replicas to be designated as the leader, to which all writes are issued. The leader is responsible for ordering all incoming writes, and for propagating those writes to other replicas (followers), in the same order.
   * The first approach is quorum-based. The leader waits until a majority of replicas have received the data before it is considered safe (i.e., committed). On leader failure, a new leader is elected through the coordination of a majority of the followers. This approach is used in Apache Zookeeper and Google's Spanner.
   * The second approach is for the leader to wait for "all" (to be clarified later) replicas to receive the data. When the leader fails, any other replica can then take over as the new leader.
1. We selected the second approach for Kafka replication for two primary reasons:
   * The second approach can tolerate more failures with the same number of replicas. That is, it can tolerate f failures with f+1 replicas, while the first approach often only tolerates f failures with 2f +1 replicas. For example, if there are only 2 replicas, the first approach can't tolerate any failures.
   * While the first approach generally has better latency, as it hides the delay from a slow replica, our replication is designed for a cluster within the same datacenter, so variance due to network delay is small.

#### Handling Failures
1. We rely on Zookeeper for detecting broker failures. 
2. Similar to Helix, we use a controller (embedded in one of the brokers) to receive all Zookeeper notifications about the failure and to elect new leaders. If a leader fails, the controller selects one of the replicas in the ISR as the new leader and informs the followers about the new leader.
3. By design, committed messages are always preserved during leadership change whereas some uncommitted data could be lost. The leader and the ISR for each partition are also stored in Zookeeper and are used during the failover of the controller. Both the leader and the ISR are expected to change infrequently since failures are rare.
4. For clients, a broker only exposes committed messages to the consumers. Since committed data is always preserved during broker failures, a consumer can automatically fetch messages from another replica, using the same offset.
5. A producer can choose when to receive the acknowledgement from the broker after publishing a message. For example, it can wait until the message is committed by the leader (i.e, it's received by all replicas in the ISR). Alternatively, it may choose to receive an acknowledgement as soon as the message is appended to the log in the leader replica, but may not be committed yet. In the former case, the producer has to wait a bit longer, but all acknowledged messages are guaranteed to be kept by the brokers. In the latter case, the producer has lower latency, but a smaller number of acknowledged messages could be lost when a broker fails.

## Design
1. [Kafka Design](https://kafka.apache.org/documentation/#design)


### Producers 
1. Producers are responsible for load balancing messages to the various brokers. 
2. They can discover all brokers from any one broker. 
1. In 0.8, there are 3 ack levels: 
   * No ack (0) 
   * Ack from N replicas (1..N) 
   * Ack from all replicas (-1)
1. Message routing 
   * Producers can be configured with a custom routing function (implementing the Partitioner interface) 
   * Default is hash-mod 
   * One side effect of routing is that there is no total ordering for a topic, but there is within a partition (this is actually really useful)
1. (Partially) Ordered Messages Consider a system that is processing updates. If you partition the messages based on the primary key you guarantee all messages for a given key end up in the same partition. Since Kafka guarantees ordering of the messages at the partition level, your updates will be processed in the correct sequence.

### Consumers 
1. Consumers request a range of messages from a Broker. 
2. They are responsible for their own state 
3. Default implementation uses ZooKeeper to manage state. 
4. In 0.8.1, Brokers will expose an API for offset management to remove direct communication between consumers and ZooKeeper (a good thing).
5. [Zero-copy ](https://medium.com/swlh/linux-zero-copy-using-sendfile-75d2eb56b39b)
   * Reading data out of Kafka is super fast thanks to java.nio.channels.FileChannel#transferTo. 
   * This method uses the "sendfile" system call which allows for very efficient transfer of data from a file to another file (including sockets). 
      * sendfile() claims to make data transfer happening under kernel space only — i.e data transferred from kernel system cache to NIC buffer (or traversed through kernel system cache if local copy), thus doesnt require context switches as in read+write combination. sendfile() has now been widely used as a supported data transfering technique especially under nginx and kafka.
   * This optimization is what allows us to pull ~100MB/s out of a single broker
   * KafkaStream is an iterable 
   * Blocking or non-blocking behavior 
   * Auto-commit offset, or manual commit 
   * Participate in a "consumer group"

1. Consumer Groups 
   * Multiple high-level consumers can participate in a single "consumer group". A consumer group is coordinated using ZooKeeper, so it can span multiple machines. In a group, each partition will be consumed by exactly one consumer (i. e., KafkaStream). 
   * This allows for broadcast or pub/sub type of messaging pattern

#### Zero Copy
1. [IBM Zero Copy](https://developer.ibm.com/articles/j-zerocopy/?mhsrc=ibmsearch_a&mhq=zero%20copy)
2. 


### Result? 
1. Brokers keep very little state, mostly just open file pointers and connections 
2. Can scale to thousands of producers and consumers* 
3. Stable performance, good scalability 
4. In 0.7, 50MB/s producer throughput, 100MB/s consumer throughput

### High-Level API
1. The Producer class determines where a message is sent based on the routing key. If a null key is given, the message is sent to a random partition
  
### Persistent Messages 
1. MessageSets received by the broker and flushed to append-only log files 
   * To maximize throughput, the same binary format for messages is used throughout the system (producer API, consumer API, and log file format) 
   * Broker only decodes far enough to read the checksum and validate it, then writes it to the disk
   * Compressed message sets or, message batching 
   * The value of a Message can be a compressed 
   * Useful for increasing throughput (especially if you are buffering messages in the producer) 
   * Can also be used as a way to atomically write multiple messages
3. Simple log format 
4. Zero-copy (i.e., sendfile) for file to socket transfer 
5. Log files are not kept forever

## Caveats 
1. Not designed for large payloads. 
2. Decoding is done for a whole message (no streaming decoding). 
3. Rebalancing can screw things up if you're doing any aggregation in the consumer by the partition key 
4. Number of partitions cannot be easily changed (chose wisely) 
5. Lots of topics can hurt I/O performance

## Hadoop InputFormat 
1. InputFormat/OutputFormat implementations 
2. Uses low-level consumer, no offsets in ZK (they are stored on HDFS instead) 
3. Useful for long term storage of messages 

## Applications
1. Log Aggregation 
   * Many applications running on many machines. You want centralized logging, but don't want to install an agent (Flume, etc). 
   * Kafka includes a log4j appender

1. Notifications 
   * Store data into data store A, need to sync it to data store B. 
   * Suppose you can send a message to Kafka when an update happens in A 

1. Stream Processing 
   * Using a stream processing tool like Storm or Apache Camel, Kafka provides an excellent backbone

## Stats Data
1. LinkedIn stats
   * Peak writes per second: 460k 
   * Average writes per day: 28 billion (324K per second)
   * Average reads per second: 2.3 million 
   * ~700 topics 
   * Thousands of producers 
   * ~1000 consumers

# Google File System (GFS)
1. a scalable distributed file system for large distributed data-intensive applications
2. It provides fault tolerance while running on inexpensive commodity hardware, and it delivers high aggregate performance to a large number of clients
## Their Design Requirements
1. First, component failures are the norm rather than the exception.  Therefore, constant monitoring, error detection, fault tolerance, and automatic recovery must be integral to the system.
2. Second, files are huge by traditional standards. it is unwieldy to manage billions of approximately KB-sized files even when the file system could support it. As a result, design assumptions and parameters such as I/O operation and blocksizes have to be revisited.
3. Third, most files are mutated by appending new data rather than overwriting existing data. Random writes within a file are practically non-existent.  Once written, the files are only read, and often only sequentially. Given this access pattern on huge files, appending becomes the focus of performance optimization and atomicity guarantees, while caching data blocks in the client loses its appeal.
4. Fourth, co-designing the applications and the file system API benefits the overall system by increasing our flexibility.
## DESIGN OVERVIEW
### Assumptions
1. The system is built from many inexpensive commodity components that often fail. It must constantly monitor itself and detect, tolerate, and recover promptly from component failures on a routine basis.
2. The system stores a modest number of large files. We expect a few million files, each typically 100 MB or larger in size. Multi-GB files are the common case and should be managed efficiently. Small files must be supported, but we need not optimize for them.
3. The workloads primarily consist of two kinds of reads: large streaming reads and small random reads. 
   * In large streaming reads, individual operations typically read hundreds of KBs, more commonly 1 MB or more. Successive operations from the same client often read through a contiguous region of a file. 
   * A small random read typically reads a few KBs at some arbitrary offset. Performance-conscious applications often batch and sort their small reads to advance steadily through the file rather than go backand forth.
1. The workloads also have many large, sequential writes that append data to files. Typical operation sizes are similar to those for reads. Once written, files are seldom modified again. Small writes at arbitrary positions in a file are supported but do not have to be efficient. 
1. The system must efficiently implement well-defined semantics for multiple clients that concurrently append to the same file. Our files are often used as producer consumer queues or for many-way merging. Hundreds of producers, running one per machine, will concurrently append to a file. Atomicity with minimal synchronization overhead is essential. The file may be read later, or a consumer may be reading through the file simultaneously.
1. High sustained bandwidth is more important than low latency. Most of our target applications place a premium on processing data in bulkat a high rate, while few have stringent response time requirements for an individual read or write.

### Interfaces
1. We support the usual operations to create, delete, open, close, read, and write files. 
2. Moreover, GFS has snapshot and record append operations.
3. Record append allows multiple clients to append data to the same file concurrently while guaranteeing the atomicity of each individual client’s append. It is useful for implementing multi-way merge results and producerconsumer queues that many clients can simultaneously append to without additional locking.

### Architecture
1. A GFS cluster consists of a single master and multiple chunkservers and is accessed by multiple clients
2. Files are divided into fixed-size chunks. Each chunkis identified by an immutable and globally unique 64 bit chunk handle assigned by the master at the time of chunkcreation.
3. Chunkservers store chunks on local disks as Linux files and read or write chunkdata specified by a chunkhandle and byte range. 
4. For reliability, each chunk is replicated on multiple chunkservers. By default, we store three replicas, though users can designate different replication levels for different regions of the file namespace.
5. The master maintains all file system metadata. 
   * This includes the namespace, access control information, the mapping from files to chunks, and the current locations of chunks. 
   * It also controls system-wide activities such as chunk lease management, garbage collection of orphaned chunks, and chunk migration between chunkservers. 
   * The master periodically communicates with each chunkserver in HeartBeat messages to give it instructions and collect its state
1. Clients interact with the master for metadata operations, but all data-bearing communication goes directly to the chunkservers.
2. Neither the client nor the chunkserver caches file data. Client caches offer little benefit because most applications stream through huge files or have working sets too large  to be cached. Not having them simplifies the client and the overall system by eliminating cache coherence issues. (Clients do cache metadata, however.)    
3. Chunkservers need not cache file data because chunks are stored as local files and so Linux’s buffer cache already keeps frequently accessed data in memory

### Single Master
1. Having a single master vastly simplifies our design and enables the master to make sophisticated chunk placement and replication decisions using global knowledge
3. However, we must minimize its involvement in reads and writes so that it does not become a bottleneck.
   * Clients never read and write file data through the master. Instead, a client asks the master which chunkservers it should contact. It caches this information for a limited time and interacts with the chunkservers directly for many subsequent operations.

### Chunk Size
1. We have chosen 64 MB, which is much larger than typical file system blocksizes. 
2. Each chunk replica is stored as a plain Linux file on a chunkserver and is extended only as needed.
3. Lazy space allocation avoids wasting space due to internal fragmentation, perhaps the greatest objection against such a large chunksize
4. A large chunksize offers several important advantages.
   *  it reduces clients’ need to interact with the master because reads and writes on the same chunkrequire only one initial request to the master for chunklocation information.
   *   since on a large chunk, a client is more likely to perform many operations on a given chunk, it can reduce network overhead by keeping a persistent TCP connection to the chunkserver over an extended period of time. 
   *   Third, it reduces the size of the metadata stored on the master. This allows us to keep the metadata in memory, which in turn brings other advantages

1. Disadvantages. 
   * A small file consists of a small number of chunks, perhaps just one. 
   * The chunkservers storing those chunks may become hot spots if many clients are accessing the same file. 

### Meta Data
1. The types: the file and chunknamespaces, the mapping from files to chunks, and the locations of each chunk’s replicas. 
2. All metadata is kept in the master’s memory. The first two types (namespaces and file-to-chunkmapping) are also kept persistent by logging mutations to an operation log stored on the master’s local disk and replicated on remote machines
3. The master does not store chunklocation information persistently. Instead, it asks each chunkserver about its chunks at master startup and whenever a chunkserver joins the cluster.

#### In-Memory Data Structures
1. Since metadata is stored in memory, master operations are fast. 
2. Furthermore, it is easy and efficient for the master to periodically scan through its entire state in the background. This periodic scanning is used to implement 
   * chunkgarbage collection, 
   * re-replication in the presence of chunkserver failures, and 
   * chunkmigration to balance load and diskspace usage across chunkservers
1. One potential concern for this memory-only approach is that the number of chunks and hence the capacity of the whole system is limited by how much memory the master
has. 
   * This is not a serious limitation in practice. The master maintains less than 64 bytes of metadata for each 64 MB chunk. 
   * Most chunks are full because most files contain many chunks, only the last of which may be partially filled. 
   * Similarly, the file namespace data typically requires less then 64 bytes per file because it stores file names compactly using prefix compression.
#### Chunk Locations
1. We initially attempted to keep chunk location information persistently at the master, but we decided that it was much simpler to request the data from chunkservers at startup, and periodically thereafter. This eliminated the problem of keeping the master and chunkservers in sync as chunkservers join and leave the cluster, change names, fail, restart, and so on. In a cluster with hundreds of servers, these events happen all too often.
2. Another way to understand this design decision is to realize that a chunkserver has the final word over what chunks it does or does not have on its own disks. There is no point in trying to maintain a consistent view of this information on the master because errors on a chunkserver may cause chunks to vanish spontaneously (e.g., a disk may go bad and be disabled) or an operator may rename a chunkserver.
#### Operation Log
1. The operation log contains a historical record of critical metadata changes. It is central to GFS. Not only is it the only persistent record of metadata, but it also serves as a logical time line that defines the order of concurrent operations. 
2. Files and chunks, as well as their versions, are all uniquely and eternally identified by the logical times at which they were created.
3. Since the operation log is critical, we must store it reliably and not make changes visible to clients until metadata changes are made persistent. Otherwise, we effectively lose the whole file system or recent client operations even if the chunks themselves survive. 
4. Therefore, we replicate it on multiple remote machines and respond to a client operation only after flushing the corresponding log record to disk both locally and remotely.
5. The master batches several log records together before flushing thereby reducing the impact of flushing and replication on overall system throughput.
6. The master recovers its file system state by replaying the operation log. To minimize startup time, we must keep the log small. The master checkpoints its state whenever the log grows beyond a certain size so that it can recover by loading the latest checkpoint from local disk and replaying only the limited number of log records after that. The checkpoint is in a compact B-tree like form that can be directly mapped into memory and used for namespace lookup without extra parsing. This further speeds up recovery and improves availability
7. Because building a checkpoint can take a while, the master’s internal state is structured in such a way that a new checkpoint can be created without delaying incoming mutations. The master switches to a new log file and creates the new checkpoint in a separate thread. The new checkpoint includes all mutations before the switch. It can be created in a minute or so for a cluster with a few million files. When completed, it is written to diskboth locally and remotely.
8. Recovery needs only the latest complete checkpoint and subsequent log files. Older checkpoints and log files can be freely deleted, though we keep a few around to guard
against catastrophes. A failure during checkpointing does not affect correctness because the recovery code detects and skips incomplete checkpoints.





