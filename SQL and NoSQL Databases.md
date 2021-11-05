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
1. Document databases target use cases where data comes in self-contained docu‐ ments and relationships between one document and another are rare.
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

#### Which data model leads to simpler application code?
1. If application has a document-like structure (a tree of one-to-many): document model
   * Better for apps with mostly one-to-many or no relationships between records
   * Limitation of document model: can't refer directly to a nested item within a document but need to use postion
   * Poor support of join: may or may not be a problem, depends on whether your app use many-to-many relationships
      * Ex. event recording doesn't need many-to-many relationships
      * It’s possible to reduce the need for joins by denormalizing, but then the application code needs to do additional work to keep the denormalized data consistent. 
      * Joins can be emulated in application code by making multiple requests to the database, but that also moves complexity into the application and is usually slower than a join performed by specialized code inside the database.
   * For highly interconnected data, the document model is awkward, the relational model is accept‐ able, and graph models are the most natural.
#### Schema flexibility
1. Document is schema-on-read (the structure of the data is implicit, and only interpreted when the data is read), in contrast with schema-on-write (the traditional approach of relational databases, where the schema is explicit and the database ensures all written data con‐ forms to it)
2. The difference between the approaches is particularly noticeable in situations where an application wants to change the format of its data.
   * It usually requires Schema change on relational database, while document database just change the app code
      * Most relational DB execute ALTER TABLE in a few milliseconds, except MySQL where it copies entire table on this
      * UPDATE takes time as well 
3. The schema-on-read approach is advantageous if the items in the collection don’t all have the same structure for some reason (i.e., the data is heterogeneous)
#### Data Locality for queries
1. A document is usually stored as a single continuous string, encoded as JSON, XML, or a binary variant thereof (such as MongoDB’s BSON). If your application often needs to access the entire document (for example, to render it on a web page), there is a performance advantage to this storage locality.
2. The locality advantage only applies if you need large parts of the document at the same time. If only part of document is needed, it's wasteful.
3. On updates to a document, entire needs to be rewritten
   * So it's usually recommended to keep failry small size documents and avoid writes that increase size of a document.
4. N.B. locality is not limited to document model, relational model can also have this (e.g. The column-family concept in Bigtable data model used in Cassandra and HBase)

#### Convergence of document and relational databases
1. Most relational DBs ahve supported XML(except MySQL) and similarly for JSON: local modification, index and query inside XML documents
2. MongoDB performs client-side join (less optimized)

### Fault-Tolerance Properties
### Concurrency Handling

## Graph-Like Data Models
1. Natural for many-to-many relationships
2. An equally powerful use of graphs is to provide a consis‐ tent way of storing completely different types of objects in a single datastore.
3. Graph databases go in the opposite direction, targeting use cases where anything is potentially related to everything

### Property Graph Model
1. You can think of a graph store as consisting of two relational tables, one for vertices and one for edges
2. It has great flexibility for data modeling: 
   * There is no schema that restricts which kinds of things can or cannot be associated
1. Graphs are good for evolvability: as you add features to your application, a graph can easily be extended to accommodate changes in your application’s data structures.
1. Products: Neo4j, Titan, InfiniteGraph

### Triple-Store Model
1. The triple-store model is mostly equivalent to the property graph model, using differ‐ ent words to describe the same ideas.
2. In a triple-store, all information is stored in the form of very simple three-part state‐ ments: (subject, predicate, object). For
3. Products: Datomic, AllegroGraph

#### The RDF Data Model
1. The semantic web is fundamentally a simple and reasonable idea: websites already publish information as text and pictures for humans to read, so why don’t they also publish information as machine-readable data for computers to read? 
2. The Resource Description Framework (RDF) [41] was intended as a mechanism for different web‐ sites to publish data in a consistent format, allowing data from different websites to be automatically combined into a web of data—a kind of internet-wide “database of everything.”
3. Triples can be a good internal data model for applications
### Datalog's Data Model
1. Datalog’s data model is similar to the triple-store model, generalized a bit. Instead of writing a triple as (subject, predicate, object), we write it as predicate(subject, object)

# Query Languages for Data
## Imperative language
   * Many programming languages are imperative
   * An imperative language tells the computer to perform certain operations in a certain order.
   * Hard to parallel across multi-cores, multi-machines because it specifies instructions that must be per‐ formed in a particular order. Declarative
   * Javascript
### imperative languages for Graph
1. Gremlin

## Declarative query language
   * you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal.
   * It is up to the database system’s query optimizer to decide which indexes and which join methods to use, and in which order to execute various parts of the query.
   * concise, easier to work with
   * hides implementation details so data engine can optimize performance underneath
   * limited in functionality: 
   * It often lend themselves to parallel execution
   * example: SQL or relational algebra 
   * CSS and XSL are declarative languages for specifying the style of a document
### Declarative Languages for Graphs
#### Cypher Query Language
1. Cypher is a declarative query language for property graphs, created for the Neo4j graph database
2. graph data can be represented in a relational database. But if we put graph data in a relational structure, can we also query it using SQL?
   * The answer is yes, but with some difficulty. The number of joins is not fixed in advance
   * Since SQL:1999, this idea of variable-length traversal paths in a query can be expressed using something called recursive common table expressions (the WITH RECURSIVE syntax).
#### The SPARQL Query Language
1. SPARQL is a query language for triple-stores using the RDF data model

#### The Foundation: Datalog
1. Datalog is a much older language than SPARQL or Cypher, it provides the foundation that later query languages build upon
   * Cascalog is a Datalog implementation for querying large datasets in Hadoop
1. it’s a very powerful approach, because rules can be combined and reused in different queries. It’s less convenient for simple one-off queries, but it can cope better if your data is complex.

## MapReduce Querying
1. MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google
   * A limited form of MapReduce is supported by some NoSQL DBs, such as MongoDB, CouchDB to perform read-only queries across many documents
1. MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. It is based on the map (also known as collect) and reduce (also known as fold or inject) functions that exist in many functional programming languages.
1. MapReduce is a fairly low-level programming model for distributed execution on a cluster of machines. Higher-level query languages like SQL can be implemented as a pipeline of MapReduce operations. Note there is nothing in SQL that constrains it to running on a single machine, and MapReduce doesn’t have a monopoly on distributed query execution.
1. A usability problem with MapReduce is that you have to write two carefully coordi‐ nated JavaScript functions, which is often harder than writing a single query

# Data Storage and Retrieval 
1. how we can store the data that we’re given, and how we can find it again when we’re asked for it.

## Log-Structured Storage Engines
1. Many databases internally use a log, which is an append-only data file.
   * Concurrency control
   * Reclaim disk space
   * Handling errors and partially written records

## Page-Oriented Storage Engines
1. Ex. B-Trees


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

## RDBMS (Relational Database Management System)
1. If you denormalize your data and include no more Joins in any database query, You can stay with MySQL, and use it like a NoSQL database.
2. Scale Out RDBMS
   * Replication
      * Master-Slave
      * Master-Master
   * Federation
   * Denormalization
   * SQL Tuning
   * Sharding
      * Partitioninig
      * Replication
3. Anti-Pattern: ORM (Object–relational mapping) + Rich Domain Model
   * One attempts to read an object from DB, but results with whole database 

### ACID
1. ACID is a set of properties of relational database transactions.
   * Atomicity - Each transaction is all or nothing
   * Consistency - Any transaction will bring the database from one valid state to another
   * Isolation - Executing transactions concurrently has the same results as if the transactions were executed serially
   * Durability - Once a transaction has been committed, it will remain so

4. Think about Your Data, Think again!
   * When do you need ACID
   * When is evental consistency a better fit?
   * Different kinds of data have different needs?
5. When RDBMS is not Good Enough
   * Scaling reads to a RDBMS is hard
   * Scaling writes to a RDBMS is impossible

### Federation
1. Federation (or functional partitioning) splits up databases by function. For example, instead of a single, monolithic database, you could have three databases: forums, users, and products, resulting in less read and write traffic to each database and therefore less replication lag. Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality. With no single central master serializing writes you can write in parallel, increasing throughput.
1. Disadvantage(s): federation
   * Federation is not effective if your schema requires huge functions or tables.
   * You'll need to update your application logic to determine which database to read and write.
   * Joining data from two databases is more complex with a server link.
   * Federation adds more hardware and additional complexity.


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
4. One thing that document and graph databases have in common is that they typically don’t enforce a schema for the data they store, which can make it easier to adapt applications to changing requirements
   * However, your application most likely still assumes that data has a certain structure; it’s just a question of whether the schema is explicit (enforced on write) or implicit (handled on read)

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
