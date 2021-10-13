# SQL and NoSQL Databases

## SQL Databases
1. If you denormalize your data and include no more Joins in any database query, You can stay with MySQL, and use it like a NoSQL database.
2. Scale Out RDBMS
   * Sharding
      * Partitioninig
      * Replication
3. Anti-Pattern: ORM (Object–relational mapping) + Rich Domain Model
   * One attempts to read an object from DB, but results with whole database 
4. Think about Your Data, Think again!
   * When do you need ACID
   * When is evental consistency a better fit?
   * Different kinds of data have different needs?

5. When RDBMS is not Good Enough
   * Scaling reads to a RDBMS is hard
   * Scaling writes to a RDBMS is impossible

### Denormalization
### SQL Tuning

## NoSQL Databases
MongoDB or CouchDB
1. Joins will now need to be done in your application code.
1. Need use cache to improve the performance.
1. We need to know the advantages/disadvantages of different databases, and when to use what

### Types of NoSQL DBs
1. Column: 
   * Cassandra is a wide column DB that supports asynchronous masterless replication
   * HBase has master-based replication
1. Document Oriented: 
   * MongoDB uses leader based replication
   * CouchDB
3. Key-Value: Dynamite, Voldemort
4. Graph
   * Neo4j
6. Datastructure Databases
   * Redis
### Open Source NoSQL DBs

1. Google: BigTable
2. Amazon: Dynamo, SimpleDB
3. Yahoo: HBase, a clone of BigTable
4. Facebook: Cassandra
5. LinkedIn: Voldemort

### Chord & Pastry
1. Distributed Hash Table (DHT)
2. Scalable
3. Partitioned
4. Fault-Tolerant
5. Decentralized
6. Peer to Peer
7. Popularized
   * Node Ring
   * Consistency Hashing

#### Consistency Hashing
1. Node ring with consistency hashing: find data with `log(N)` jumps


3. We also data into chunks (shards) or nodes.
   * Each shard is equal, no leader and followers
   * No configuration service needed, Shards talks to each other and exchange information
   * Gossip Protocol: To reduce network load, shards only talk to less than 3 other shards every second
   * No cluster proxy needed, Every node knows about each other, and can forward request to corresponding node
4. We can use round robin to choose inital node to process request, or choose the node with the shortest distance to client
   * The initial node is called: coordinator node. The coordinator decides which node to process the input data
      * We can use **consistent hashing** algorithm to pick the node
      * N.B. **consisten hasing** is also used to design distributed cache
   * It uses **quorum writes** to write to replicas (asynchronously, since synchronous writing is slow)
   * Similarly, there's **quorum reads** approach
   * Cassandra uses version  number to determine staleness of data
5. Consistency
   * Eventual consistency: in SQL DB, although some followers may lag behind leader DB in terms of data staleness, they'll eventuall become the same.
   * Cassandra offers **tunable consistency**: 
7. Apache Cassandra
   * Fault tolerant: support multi-data center replication
   * Scalable: both read and write throughput increases linearly as new machines are added
   * Works well with time series data

## Who's ACID
1. Relational DBs (MySQL, Orcale, Postgres)
2. Object DBs: Gemstone, db4o
3. Clustering Products: Coherence, Terracotta
4. Most Caching Products: ehcache

## Who's BASE
1. Distributed Databases: Cassandra, Riak, Dynomite, SimpleDB

