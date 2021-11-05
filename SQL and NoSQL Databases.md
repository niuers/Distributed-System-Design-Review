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
   * 

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

### Sharding
1. In general if your service has lots of rapidly changing data (i.e. lots of writes) or is sporadically queried by lots of users in a way which causes your working set not to fit in memory (i.e. lots of reads leading to lots of page faults and disk seeks) then your primary bottleneck will likely be I/O. This is typically the case with social media sites like Facebook, LinkedIn
2. Traditionally it hasn’t been read performance that caused people to shard anyway. It has been write performance.
   * But with the rise of SSD, like Fusion-IO’s ioDrive that can do 120K IOPS, which improves on the writes, thus reduces the need for Sharding
4. Sharding distributes data across different databases such that each database can only manage a subset of the data. Taking a users database as an example, as the number of users increases, more shards are added to the cluster.
5. Similar to the advantages of federation, sharding results in 
   * Performant: 
      * less read and write traffic, 
      * less replication, and more cache hits. 
      * Index size is also reduced, which generally improves performance with faster queries. 
   * High Availability: If one shard goes down, the other shards are still operational, although you'll want to add some form of replication to avoid data loss. 
   * High Throughtput: Like federation, there is no single central master serializing writes, allowing you to write in parallel with increased throughput.
6. Common ways to shard a table of users is either through the user's last name initial or the user's geographic location.
7. Sharding Architecture
   * Data are denormalized: You store together data that are used together. Ex, the user profile data would be stored and retrieved as a whole. You just get a blob and store a blob. No joins are needed and it can be written with one disk write.
   * Data are parallelized across many physical instances: 
   * Data are kept small
   * Data are more highly available
   * It doesn't use replication: Replicating data from a master server to slave servers is a traditional approach to scaling. Obviously the master becomes the write bottleneck and a single point of failure. Sharding cleanly and elegantly solves the problems with replication.

3. Disadvantage(s): sharding
   * You'll need to update your application logic to work with shards, which could result in complex SQL queries.
   * Data distribution can become lopsided in a shard. For example, a set of power users on a shard could result in increased load to that shard compared to others.
   * How do you partition your data in shards? What data do you put in which shard? Where do comments go? Should all user data really go together, or just their profile data? Should a user's media, IMs, friends lists, etc go somewhere else?
   * Rebalancing adds additional complexity. A sharding function based on consistent hashing can reduce the amount of transferred data.
      * What happens when a shard outgrows your storage and needs to be split? moving data from shard to shard required a lot of downtime.
      * Rebalancing has to be built in from the start. Google's shards automatically rebalance. For this to work data references must go through some sort of naming service so they can be relocated.
      * rebalance means the partitioning scheme changed AND all existing data moved to new locations. Doing this without incurring down time is extremely difficult and not supported by any off-the-shelf today.
   * Joining data from multiple shards is more complex.
      * it is often not feasible to perform joins that span database shards due to performance constraints since data has to be compiled from multiple servers and the additional complexity of performing such cross-server. 
      * A common workaround is to denormalize the database so that queries that previously required joins can be performed from a single table.
      * Of course, the service now has to deal with all the perils of denormalization such as data inconsistency
      * Thankfully, because of caching and fast networks this process is usually fast enough that your page load times can be excellent.
   * Sharding adds more hardware and additional complexity.
      * The application developer has to write more code to be able to handle sharding logic (this is actually lessened with projects such as HiveDB.)
      * Operational issues become more difficult (backing up, adding indexes, changing schema).
   * Referential integrity – As you can imagine if there's a bad story around performing cross-shard queries it is even worse trying to enforce data integrity constraints such as foreign keys in a sharded database. Most relational database management systems do not support foreign keys across databases on different database servers. This means that applications that require referential integrity often have to enforce it in application code and run regular SQL jobs to clean up dangling references once they move to using database shards. Dealing with data inconsistency issues due to denormalization and lack of referential integrity can become a significant development cost to the service.
   * 



#### You don’t want to shard.
Optimize everything else first, and then if performance still isn’t good enough, it’s time to take a very bitter medicine. The reason you need to shard basically comes down to one of these two reasons:
1. Very large working set – The amount of memory you require to keep your frequently accessed data loaded exceeds what you can (economically) fit in a commodity machine. 5 years ago this was 4GB, today it is 128GB or even 256GB. Defining “working set” is always an interesting concept here, since with good schema and indexing it normally doesn’t need to be the same size as your entire database.
1. Too many writes – Either the IO system, or a slave can’t keep up with the amount of writes being sent to the server. While the IO system can be improved with a RAID 10 controller w/battery backed write cache, the slave delay problem is actually very hard to solve. 

#### Federation
1. Another form of sharding
1. Federation (or functional partitioning) splits up databases by function. For example, instead of a single, monolithic database, you could have three databases: forums, users, and products, resulting in less read and write traffic to each database and therefore less replication lag. Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality. With no single central master serializing writes you can write in parallel, increasing throughput.
2. This is usually the best way to fix any of the problems mentioned above. What you do is pick a few very busy tables, and move them onto their own MySQL server. Partition-by-function keeps the architecture still simple, and should work for most cases unless you have a single table which by itself can’t fit into the above constraints.
3. Disadvantage(s): federation
   * Federation is not effective if your schema requires huge functions or tables.
   * You'll need to update your application logic to determine which database to read and write.
   * Joining data from two databases is more complex with a server link.
   * Federation adds more hardware and additional complexity.
### Sharding by range
#### Sharding by Hashing
1. This approach should ensure a uniform allocation of data to each server. The key problem with this approach is that it effectively fixes your number of database servers since adding new servers means changing the hash function which without downtime is like being asked to change the tires on a moving car.
#### Sharding by look-up service
1. It’s a highly scalable architecture, and once you write scripts to be able to migrate users to/from shards you can tweak and rebalanced to make sure that all your hardware is utilized efficiently.Â  The only problem with this method is what I stated at the start: it’s complicated.
2. It can be a single point of failure




### Denormalization
1. Denormalization attempts to improve read performance at the expense of some write performance. Redundant copies of the data are written in multiple tables to avoid expensive joins. Some RDBMS such as PostgreSQL and Oracle support materialized views which handle the work of storing redundant information and keeping redundant copies consistent.
1. Once data becomes distributed with techniques such as federation and sharding, managing joins across data centers further increases complexity. Denormalization might circumvent the need for such complex joins.
1. In most systems, reads can heavily outnumber writes 100:1 or even 1000:1. A read resulting in a complex database join can be very expensive, spending a significant amount of time on disk operations.

#### Disadvantage(s): denormalization
1. Data is duplicated.
1. Constraints can help redundant copies of information stay in sync, which increases complexity of the database design.
1. A denormalized database under heavy write load might perform worse than its normalized counterpart.

### SQL Tuning
1. It's important to benchmark and profile to simulate and uncover bottlenecks.
   * Benchmark - Simulate high-load situations with tools such as ab.      
      * You’re going to need numbers if you want to make a good decision. What queries are the worst? Where are the bottlenecks? Under what circumstances am I generating bad queries? Benchmarking is will let you simulate high-stress situations and, with the aid of profiling tools, expose the cracks in your database configuration. 
   * Profile - Enable tools such as the slow query log to help track performance issues.
      * The rule in any situation where you want to opimize some code is that you first profile it and then find the bottlenecks.
      * Profiling enables you to find the bottlenecks in your configuration, whether they be in memory, CPU, network, disk I/O, or, what is more likely, some combination of all of them.
   * Benchmarking and profiling might point you to the following optimizations.

1. Tighten up the schema
   * MySQL dumps to disk in contiguous blocks for fast access.
   * Use CHAR instead of VARCHAR for fixed-length fields.
      * CHAR effectively allows for fast, random access, whereas with VARCHAR, you must find the end of a string before moving onto the next one.
   * Use TEXT for large blocks of text such as blog posts. TEXT also allows for boolean searches. Using a TEXT field results in storing a pointer on disk that is used to locate the text block.
   * Use INT for larger numbers up to 2^32 or 4 billion.
   * Use DECIMAL for currency to avoid floating point representation errors.
   * Avoid storing large BLOBS, store the location of where to get the object instead.
   * VARCHAR(255) is the largest number of characters that can be counted in an 8 bit number, often maximizing the use of a byte in some RDBMS.
   * Set the NOT NULL constraint where applicable to improve search performance.
      * In Oracle, NULL values are not indexed, so if you have NULLs in the column, a select might have to do full table scan
1. Use good indices
   * Columns that you are querying (SELECT, GROUP BY, ORDER BY, JOIN) could be faster with indices.
   * Indices are usually represented as self-balancing B-tree that keeps data sorted and allows searches, sequential access, insertions, and deletions in logarithmic time.
   * Placing an index can keep the data in memory, requiring more space. Obviously each index requires space proportional to the number of rows in your table, so too many indices winds up taking more memory.
   * Writes could also be slower since the index also needs to be updated.
   * When loading large amounts of data, it might be faster to disable indices, load the data, then rebuild the indices.
1. Avoid expensive joins
   * Denormalize where performance demands it.
1. Partition tables
   * Break up a table by putting hot spots in a separate table to help keep it in memory.
   * Frequently accessed data is kept in one table while infrequently accessed data is kept in another. Since the data is now partitioned the infrequently access data takes up less memory. 
   * You can also optimize for writing: frequently changed data can be kept in one table, while infrequently changed data can be kept in another. This allows more efficient caching since MySQL no longer needs to expire the cache for data which probably hasn’t changed.
1. Don’t Overuse Artificial Primary Keys
   * Artificial primary keys are nice because they can make the schema less volatile.
3. Tune the query cache
   * In some cases, the query cache (deprecated from MySQL 8) could lead to performance issues.
   * The query cache stores the text of a SELECT statement together with the corresponding result that was sent to the client. If an identical statement is received later, the server retrieves the results from the query cache rather than parsing and executing the statement again. The query cache is shared among sessions, so a result set generated by one client can be sent in response to the same query issued by another client.


## NoSQL Databases
1. NoSQL is a collection of data items represented in a key-value store, document store, wide column store, or a graph database. Data is denormalized, and joins are generally done in the application code. Most NoSQL stores lack true ACID transactions and favor eventual consistency.
### BASE
1. BASE is often used to describe the properties of NoSQL databases. In comparison with the CAP Theorem, BASE chooses availability over consistency.
   * Basically available - the system guarantees availability.
   * Soft state - the state of the system may change over time, even without input.
   * Eventual consistency - the system will become consistent over a period of time, given that the system doesn't receive input during that period.

#### Who's BASE
1. Distributed Databases: Cassandra, Riak, Dynomite, SimpleDB

### MongoDB or CouchDB
1. Joins will now need to be done in your application code.
1. Need use cache to improve the performance.
1. We need to know the advantages/disadvantages of different databases, and when to use what
 
### NoSQL: Nonrelational Model
1. Greater scalability for very large datasets or very high write throughput
2. Specialized query operations that are not well supported by the relational model
3. Frustration with the restrictiveness of relational schemas
4. One thing that document and graph databases have in common is that they typically don’t enforce a schema for the data they store, which can make it easier to adapt applications to changing requirements
   * However, your application most likely still assumes that data has a certain structure; it’s just a question of whether the schema is explicit (enforced on write) or implicit (handled on read)

### Key-value store
1. Abstraction: hash table
1. A key-value store generally allows for `O(1)` reads and writes and is often backed by memory or SSD. Data stores can maintain keys in lexicographic order, allowing efficient retrieval of key ranges. Key-value stores can allow for storing of metadata with a value.
1. Key-value stores provide high performance and are often used for simple data models or for rapidly-changing data, such as an in-memory cache layer. Since they offer only a limited set of operations, complexity is shifted to the application layer if additional operations are needed.
1. A key-value store is the basis for more complex systems such as a document store, and in some cases, a graph database.
1. Disadvantages
   * No way to make a column mandatory (equivalent of NOT NULL).
   * No way to use SQL data types to validate entries.
   * No way to ensure that attribute names are spelled consistently.
   * No way to put a foreign key on the values of any given attribute, e.g. for a lookup table.
   * Fetching results in a conventional tabular layout is complex and expensive, because to get attributes from multiple rows you need to do JOIN for each attribute.

3. Products
   * Dynamo, memcached, Redis
#### Redis
1. Redis is a in-memory, key-value data store.
2. Primary memory is limited(much lesser size and expensive than secondary) therefore Redis cannot store large files or binary data. It can only store those small textual information which needs to be accessed, modified and inserted at a very fast rate. If we try to write more data than available memory then we will receive errors.
1. Redis is an in-memory but persistent on disk database, so it represents a different trade off where very high write and read speed is achieved with the limitation of data sets that can't be larger than memory. Another advantage of in memory databases is that the memory representation of complex data structures is much simpler to manipulate compared to the same data structures on disk, so Redis can do a lot, with little internal complexity. At the same time the two on-disk storage formats (RDB and AOF) don't need to be suitable for random access, so they are compact and always generated in an append-only fashion (Even the AOF log rotation is an append-only operation, since the new version is generated from the copy of data in memory). However this design also involves different challenges compared to traditional on-disk stores. Being the main data representation on memory, Redis operations must be carefully handled to make sure there is always an updated version of the data set on disk.
1. we are very happy if we can do one thing well: data served from memory, disk used for storage.
1. Redis server is responsible for storing data in memory. It handles all kinds of management and forms the major part of architecture. Redis client can be Redis console client or any other programming language’s Redis API.

1. As we saw that Redis stores everything in primary memory. Therefore we need a way for datastore persistance.
   * RDB: RDB makes a copy of all the data in memory and stores them in secondary storage(permanent storage). This happens in a specified interval. So there is chance that you will loose data that are set after RDB’s last snapshot. 
   * AOF: AOF logs all the write operations received by the server. Therefore everything is persistance. The problem with using AOF is that it writes to disk for every operation and it is a expensive task and also size of AOF file is large than RDB file.
   * SAVE: You can force redis server to create a RDB snapshot anytime using the redis console client SAVE command.

1. Backup And Recovery Of Redis DataStore: If you are using Redis in a replicated environment then there is no need for backup.
1. Redis Replication
   * All the slaves contain exactly same data as master. There can be as many as slaves per master server. When a new slave is inserted to the environment, the master automatically syncs all data to the slave.
   * All the queries are redirected to master server, master server then executes the operations. When a write operation occurs, master replicates the newly written data to all slaves. When a large number sort or read operation are made, master distributes them to the slaves so that a large number of read and sort operations can be executed at a time.
   * If a slave fails, then also the environment continues working. when the slave again starts working, the master sends updated data to the slave.
   * If there is a crash in master server and it looses all data then you should convert a slave to master instead of bringing a new computer as a master. If we make a new computer as master then all data in the environemt will be lost because new master will have no data and will makes the slaves also to have zero data(new master does resync ).  If master fails but data is persistent(disk not crashed) then starting up the same master server again will bring up the whole environment to running mode.
   * Replication helped us from disk failures and other kinds of hardware failures. It also helped to execute multiple read/sort queries at a time.
1. Persistance In Redis Replication
   * We saw how persistance can tackle unexpected failures and keep our backend strong. But one way whole data can be lost is if the whole replicated environment goes down due to power failure. This happens because all data is stored in primary memory. So we need persistance here also.
   * We can configure master or any one slave to store data in secondary storage using any method(AOF and RDB). Now when the whole environment is again started, make the persistent server as the master server.
   * Using persistance and replication together all our data is completely safe and protected from unexpected failures.
1. Clustering In Redis
   * Clustering is a technique by which data can be sharded(divided) into many computers. The main advantage is that more data can be stored in a cluster because its a combination of computers.
   * Suppose we have one redis server with 64GB of memory i.e., we can have only 64GB of data. Now if we have 10 clustered computers with each 64GB of RAM then we can store 640GB of data.
   * If one node fails then the whole cluster stops working.
1. Clustering And Replication Together to tolerate node failure.
1. Is using Redis together with an on-disk database a good idea?
Yes, a common design pattern involves taking very write-heavy small data in Redis (and data you need the Redis data structures to model your problem in an efficient way), and big blobs of data into an SQL or eventually consistent on-disk database. Similarly sometimes Redis is used in order to take in memory another copy of a subset of the same data stored in the on-disk database. This may look similar to caching, but actually is a more advanced model since normally the Redis dataset is updated together with the on-disk DB dataset, and not refreshed on cache misses.

##### What's the Redis memory footprint?
To give you a few examples (all obtained using 64-bit instances):

An empty instance uses ~ 3MB of memory.
1 Million small Keys -> String Value pairs use ~ 85MB of memory.
1 Million Keys -> Hash value, representing an object with 5 fields, use ~ 160 MB of memory.


### Document store
1. Abstraction: key-value store with documents stored as values
1. A document store is centered around documents (XML, JSON, binary, etc), where a document stores all information for a given object. Document stores provide APIs or a query language to query based on the internal structure of the document itself. Note, many key-value stores include features for working with a value's metadata, blurring the lines between these two storage types.
1. Based on the underlying implementation, documents are organized by collections, tags, metadata, or directories. Although documents can be organized or grouped together, documents may have fields that are completely different from each other.
1. Some document stores like MongoDB and CouchDB also provide a SQL-like language to perform complex queries. DynamoDB supports both key-values and documents.
1. Document stores provide high flexibility and are often used for working with occasionally changing data.
1. Products: MongoDB, CouchDB, ElasticSearch

#### ElasticSearch
1. inverted index
   * The inverted index maps terms to documents (and possibly positions in the documents) containing the term. Since the terms in the dictionary are sorted, we can quickly find a term, and subsequently its occurrences in the postings-structure. This is contrary to a "forward index", which lists terms related to a specific document.
   * Consequently, an index term is the unit of search. The terms we generate dictate what types of searches we can (and cannot) efficiently do.
   * In terms of complexity, looking up terms by their prefix is O(log(n)), while finding terms by an arbitrary substring is O(n).
   * we have highlighted why it is important to be meticulous about index term generation: to get searches that can be performed efficiently.
1. Building Indexes
   * When building inverted indexes, there's a few things we need to prioritize: search speed, index compactness, indexing speed and the time it takes for new changes to become visible.
   * Search speed and index compactness are related: when searching over a smaller index, less data needs to be processed, and more of it will fit in memory. Both, particularly compactness, come at the cost of indexing speed, as we'll see.
   * To minimize index sizes, various compression techniques are used. 
   * Keeping the data structures small and compact means sacrificing the possibility to efficiently update them.
      * The exception is deletions. When you delete a document from an index, the document is marked as such in a special deletion file, which is actually just a bitmap which is cheap to update. The index structures themselves are not updated.
   * When new documents are added (perhaps via an update), the index changes are first buffered in memory. Eventually, the index files in their entirety, are flushed to disk. The written files make up an index segment.

1. Index Segment
   * A Lucene index is made up of one or more immutable index segments, which essentially is a "mini-index". When you do a search, Lucene does the search on every segment, filters out any deletions, and merges the results from all the segments. Obviously, this gets more and more tedious as the number of segments grows. To keep the number of segments manageable, Lucene occasionally merges segments according to some merge policy as new segments are added.
   * Index segments are immutable. Deleted documents are marked as such.
   * As new segments are created (either due to a flush or a merge), they also cause certain caches to be invalidated, which can negatively impact search performance. Caches like the field and filter caches are per segment. 
   * The most common cause for flushes with Elasticsearch is probably the continuous index refreshing, which by default happens once every second. As new segments are flushed, they become available for searching, enabling (near) real-time search. While a flush is not as expensive as a commit (as it does not need to wait for a confirmed write), it does cause a new segment to be created, invalidating some caches, and possibly triggering a merge.
   * When indexing throughput is important, e.g. when batch (re-)indexing, it is not very productive to spend a lot of time flushing and merging small segments. Therefore, in these cases it is usually a good idea to temporarily increase the refresh_interval-setting, or even disable automatic refreshing altogether. One can always refresh manually, and/or when indexing is done.

1. Elasticsearch indexes
   * An Elasticsearch index is made up of one or more shards, which can have zero or more replicas. These are all individual Lucene indexes. That is, an Elasticsearch index is made up of many Lucene indexes, which in turn is made up of index segments. 
   * A "shard" is the basic scaling unit for Elasticsearch. As documents are added to the index, it is routed to a shard. By default, this is done in a round-robin fashion, based on the hash of the document's id.
   * It is important to know, however, that the number of shards is specified at index creation time, and cannot be changed later on.
   * A search is done on every segment, with the results merged.
   * Segments are occasionally merged.
   * Field and filter caches are per segment.
1. Transactions
   * Elasticsearch does not have transactions.
   * While Lucene has a concept of transactions, Elasticsearch does not. All operations in Elasticsearch add to the same timeline, which is not necessarily entirely consistent across nodes, as the flushing is reliant on timing. 
   * Managing the isolation and visibility of different segments, caches and so on across indexes across nodes in a distributed system is very hard. Instead of trying to do this, it prioritizes being fast.
   * Elasticsearch has a "transaction log" where documents to be indexed are appended. Appending to a log file is a lot cheaper than building segments, so Elasticsearch can write the documents to index somewhere durable - in addition to the in-memory buffer, which is lost on crashes. You can also specify the consistency level required when you index. For example, you can require every replica to have indexed the document before the index operation returns.











### Wide column store
1. Abstraction: nested map ColumnFamily<RowKey, Columns<ColKey, Value, Timestamp>>
1. A wide column store's basic unit of data is a column (name/value pair). A column can be grouped in column families (analogous to a SQL table). Super column families further group column families. You can access each column independently with a row key, and columns with the same row key form a row. Each value contains a timestamp for versioning and for conflict resolution.
1. Google introduced Bigtable as the first wide column store, which influenced the open-source HBase often-used in the Hadoop ecosystem, and Cassandra from Facebook. Stores such as BigTable, HBase, and Cassandra maintain keys in lexicographic order, allowing efficient retrieval of selective key ranges.
1. Wide column stores offer high availability and high scalability. They are often used for very large data sets.
2. Products: BigTable, HBase, Cassandra

### Graph Database
1. Abstraction: graph
1. In a graph database, each node is a record and each arc is a relationship between two nodes. Graph databases are optimized to represent complex relationships with many foreign keys or many-to-many relationships.
1. Graphs databases offer high performance for data models with complex relationships, such as a social network. They are relatively new and are not yet widely-used; it might be more difficult to find development tools and resources. Many graphs can only be accessed with REST APIs.

1. Products: Neo4j, FlockDB, 



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

## SQL or NoSQL
### Reasons for SQL:

1. Structured data
1. Strict schema
1. Relational data
1. Need for complex joins
1. Transactions
1. Clear patterns for scaling
1. More established: developers, community, code, tools, etc
1. Lookups by index are very fast

### Reasons for NoSQL:

1. Semi-structured data
1. Dynamic or flexible schema
1. Non-relational data
1. No need for complex joins
1. Store many TB (or PB) of data
1. Very data intensive workload
1. Very high throughput for IOPS

### Sample data well-suited for NoSQL:

1. Rapid ingest of clickstream and log data
1. Leaderboard or scoring data
1. Temporary data, such as a shopping cart
1. Frequently accessed ('hot') tables
1. Metadata/lookup tables



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
1. We also data into chunks (shards) or nodes.
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
   * Migrating a database schema in MySQL on a huge table takes forever and a day. That’s a very real problem if you want to avoid an enterprisey schema full of kludges put in place to avoid adding, renaming, or dropping columns on big tables. Or avoid long scheduled maintenance windows.
3. Object DBs: Gemstone, db4o
4. Clustering Products: Coherence, Terracotta
5. Most Caching Products: ehcache


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

## Consistent Hashing
### Mechanism
1. We select N random integers (where N is around 100 or 200) for each server and sort those values into an array of N * server.size values. To look up the server for a key, we find the closest value >= the key hash and use the associated server. The values form a virtual circle; the key hash maps to a point on that circle and then we find the server clockwise from that point.


### Implications
#### Easier to Avoid Hotspots
1. When you put data on nodes based on a random result, which is what the hash function calculates, a value that’s a lot more random than the key it’s based on, it’s easier to avoid hotspots.
2. when you determine the location in the cluster based solely on the hash of the key, chances are much higher that two keys lexicographically close to each other end up on different nodes. Thus, the load is shared more evenly. The disadvantage is that you lose the order of keys.
3. There are partitioning schemes that can work around this, even with a range-based key location. HBase (and Google’s BigTable, for that matter) stores ranges of data in separate tablets. As tablets grow beyond their maximum size, they’re split up and the remaining parts re-distributed. The advantage of this is that the original range is kept, even as you scale up.
#### Consistent Hashing Enables Partitioning
1. When you have a consistent hash, everything looks like a partition. The idea is simple. Consistent hashing forms a keyspace, which is also called continuum. As a node joins the cluster, it picks a random number, and that number determines the data it’s going to be responsible for. Everything between this number and one that’s next in the ring and that has been picked by a different node previously, is now belong to this node. The resulting partition could be of any size theoretically. It could be a tiny slice, or a large one.
2. First implementations of consistent hashing still had the problem that a node picking a random range of keys resulted in one node potentially carrying a larger keyspace than others, therefore still creating hotspots.
3. But the improvement was as simple as it was ingenious. A hash function has a maximum result set, a SHA-1 function has a bit space of 2^160. You do the math. Instead of picking a random key, a node could choose from a fixed set of partitions, like equally size pizza slices. But instead of picking the one with the most cheese on, everyone gets an equally large slice. The number of partitions is picked up front, and practically never changes over the lifetime of the cluster.
#### Partitioning Makes Scaling Up and Down More Predictable
1. With a fixed number of partitions of the same size, adding new nodes becomes even less of a burden than with just consistent hashing. With the former, it was still unpredictable how much data had to be moved around to transfer ownership of all the data in the range of the new node. One thing’s for sure, it already involved a lot less work than previous methods of sharding data.
2. With partitioning, a node simply claims partitions, and either explicitly or implicitly asks the current owners to hand off the data to them. As a partition can only contain so many keys, and randomness ensures a somewhat even spread of data, there’s a lot less unpredictability about the data that needs to be transferred.
1. If that partitions just so happens to carry the largest object by far in you whole cluster, that’s something even consistent hashing can’t solve. It only cares for keys.
1. Going back to HBase, it cares for keys and the size of the tablet the data is stored in, as it breaks up tablets once they reach a threshold. Breaking up and reassigning a tablet requires coordination, which is not an easy thing to do in a distributed system.

#### Consistent Hashing and Partitioning Enable Replication
1. Consistent hashing made one thing a lot easier: replicating data across several nodes. The primary means for replication is to ensure data survives single or multiple machine failures. The more replicas you have, the more likely is your data to survive one or more hardware crashes. With three replicas, you can afford to lose two nodes and still serve the data.
1. With a fixed set of partitions, a new node can just pick the ones it’s responsible for, and another stack of partitions it’s going to be a replica for. When you really think about it, both processes are actually the same. The beauty of consistent hashing is that there doesn’t need to be master for any piece of data. Every node is simply a replica of a number of partitions.

But replication has another purpose besides ensuring data availability.

#### Replication Reduces Hotspots (Even More!!!)
1. Having more than one replica of a single piece of data means you can spread out the request load even more. With three replicas of that data, residing on three different nodes, you can now load-balance between them. Neat!
1. With that, consistent hashing enables a pretty linear increase in capacity as you add more nodes to a cluster.

#### Consistent Hashing Enables Scalability and Availability
1. Consistent hashing allows you to scale up and down easier, and makes ensuring availability easier. Easier ways to replicate data allows for better availability and fault-tolerance. Easier ways to reshuffle data when nodes come and go means simpler ways to scale up and down.

1. It’s an ingenious invention, one that has had a great impact. Look at the likes of Memcached, Amazon’s Dynamo, Cassandra, or Riak. They all adopted consistent hashing in one way or the other to ensure scalability and availability.










