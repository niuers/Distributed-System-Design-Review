# Data Models
1. They are perhaps the most important part on how we *think about the problem* 
2. Most applications are built by layering one data model on top of another. For each layer, the key question is: *how is it represented in terms of the next-lower layer?* Each layer hides the complexity of the layers below it by providing a clean data model.
   * Application layer: objects or data structures often specific to application
   * To Store/Query those data structures: express them in general-purpose data model, e.g. JSON, XML documents, tables, graph
   * Database(Data storage) layer: how bytes are arranged in memory, disk or on a network for query, search, processing.
   * Hardware layer: how bytes are represented in electrical currents, pulses of light, etc.

## Relational Model
### Definition
1. Data is organized into relations (called tables in SQL), where each relation is an unordered collection of tuples (rows in SQL)
1. A relation (table) is simply a collection of tuples (rows), and that’s it.
2. A key insight of the relational model was this: you only need to build a query optimizer once, and then all applications that use the database can benefit from it.

### The Object-Relational Mismatch
1. A common criticism of the SQL data model (Impedance Mismatch): if data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns. 
   * Object-Relational Mapping (ORM) framework reduces the work but not completely

## Document Model
### JSON representation
1. Although the JSON model might reduce the impedance mismatch between the application code and the storage layer there are also problems with JSON as a data encoding format.
1. The JSON representation has better *locality* than the multi-table schema
1. JSON representation makes **one-to-many relationships** tree structure (e.g. user has multiple jobs) explicit. 
1. The lack of a schema
   * It can be advantageous for scaling
   * This flexibility facilitates the mapping of documents to an entity or an object.

### Many-to-One and Many-to-Many Relationships
#### Many-to-One Relationship
1. Example: Many users can live in one city
2. It's usually better store an ID instead of a text string to avoid duplication, write overheads and risk inconsistencies 
   * Removing such duplication is the key idea behind normalization in databases
   * As a rule of thumb, if you’re duplicating values that could be stored in just one place, the schema is not normalized.
1. Unfortunately, normalizing this data requires many-to-one relationships, which doesn't fit nicely with document model.
   * In relational databases, it's normal to refer to rows in other tables by ID since joins are easy
   * In document databases, joins are not needed for one-to-many tree structures, and support for joins is often weak.
   * You have to emulate a join in application code
#### Many-to-Many Relationship
1. Data has a tendency of becoming more interconnected as features are added to applications.
   * Example, Company name is not just a string, but a link to a company entity
1. This requires many-to-many relationships

### Document Databases
1. A document database (also known as a document-oriented database or a document store) is a database that stores information in documents.
1. Document databases offer a variety of advantages, including:
   * An intuitive data model that is fast and easy for developers to work with.
   * A flexible schema that allows for the data model to evolve as application needs change.
   * The ability to horizontally scale out.
1. Document databases are considered to be non-relational (or NoSQL) databases. 
1. MongoDB
   * MongoDB stores data records as BSON documents. BSON is a binary representation of JSON documents. The maximum BSON document size is 16 megabytes.
   * A MongoDB collection is a grouping of MongoDB documents, it's the equivalent of an RDBMS table. A collection exists within a single database. **Collections do not enforce a schema as in SQL databases** (where you must determine and declare a table's schema before inserting data, ). Documents within a collection can have different fields. Typically, all documents in a collection have a similar or related purpose.   

3. RethinkDB
4. CouchDB
5. Espresso

## Relational Database vs. Document Database

### Data Model Comparison
1. The main arguments in favor of the document data model are 
   * schema flexibility
   * better performance due to locality
      * Document databases store nested records (one-to-many relationships) within their parent record rather than in a separate table.
   * For some applications it is closer to the data structures used by the application. 
1. The relational model counters by providing better support for
   * joins
   * many-to-one and many-to-many relationships.
      * When it comes to representing many-to-one and many-to-many relationships, relational and document databases are not fundamentally different: in both cases, the related item is referenced by a unique identifier, which is called a foreign key in the relational model and a document reference in the document model. That identifier is resolved at read time by using a join or follow-up queries.

### Fault-Tolerance Properties
### Concurrency Handling

# SQL and NoSQL Databases
## Database Scaling
Most web apps are majority reads, around 95% +
### Basic Scaling Techniques
1. Indexes
   * based on columns
   * Speed up read performance
   * Writes and updates become slightly slower
   * More storage required for index
3. Denormalization
   * Add redundant data to tables to reduce joins
   * Boosts read performance
   * Slows down writes
   * Risk inconsistent data across tables
   * Code is harder to write
5. Connection pooling
   * Allow multiple application threads to use same DB connection
   * Saves on overhead of independent DB connections
7. Caching
   * One of the most important way to scale the DB
   * Can't cache everything
      * Dynamic data, e.g. real-time driver location
   * Redis/Memcached
9. Vertical Scaling
### Replications and Partitioning

#### Replications
1. Read Replicas
   * Master server dedicated only to writes
   * Consistency : Have to handle making sure new data reaches replicas
   * Side effect: fault-tolerance
1. 
#### Partitioning
1. Sharding: 
   * Horizontal partitioning
   * Schema of table stays the same, but split across multiple DBs
   * Downside- Hot Keys, no joins across shards
      * InstantGram: famous people, Justin Bieber
2. Vertical Partition
   * Divide schema of database into separate tables
   * Generally divid by functionality
   * Best when most data in row isn't need for most queries

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

### NoSQL: Nonrelational Model
1. Greater scalability for very large datasets or very high write throughput
2. Specialized query operations that are not well supported by the relational model
3. Frustration with the restrictiveness of relational schemas

### When to Consider NoSQL
1. If you do transactions or banking, you want consistency
2. But for Google or FB, no strict consistency, we can trade it off for scale

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

## Consistency
### Client-Side Consistency
1. Strong  Consistency
2. Weak Consistency
   * Eventual Consistency
   * Never Consistency
3. Eventual Consistency Levels
   * Casual Consistency
   * Read-your-writes consistency (important
   * Session Consistency
   * Monotonic read Consistency
   * Monotonic write Consistency

### Server-Side Consistency
1. N: The number of nodes that store replicas of data
2. W: The number of replicas that need to acknowledge the receipt of the update before the update completes
3. R: The number of replicas that are contacted when a data object is accessed through a read operation
4. W+R > N: Strong consistency
5. W+R <=N: Eventual consistency
