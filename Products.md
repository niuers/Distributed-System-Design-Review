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
2. Messages are persistent 
3. Everything is distributed - producers, brokers, consumers, the queue itself 
4. Consumers maintain their own state (i.e., "dumb" brokers) 
5. Throughput is key

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
1. Partitions of a topic are replicated. 
2. One broker is the "leader" of a partition. 
3. All writes and reads must go to the leader. 
4. Replicas exist for fault-tolerance, not scalability. 
5. When writing, messages can be synchronously written to N replicas (depending on the producer's ACKiness)

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
