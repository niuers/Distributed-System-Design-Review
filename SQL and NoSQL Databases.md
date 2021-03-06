https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/relational-vs-nosql-data

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
   * Good for write oriented applications

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
1. How we can store the data that we’re given, and how we can find it again when we’re asked for it?

## Log-Structured Storage Engines
1. Many databases internally use a log: an append-only sequence of records. You need deal with following:
   * Concurrency control
   * Reclaim disk space
   * Handling errors and partially written records
1. The search operation is bad though, you have to scan the entire file: O(n). We need **index**

### Advantages of Append-Only Log
1. Appending and segment merging are sequential write operations, which are much faster than random writes, especially on magnetic spinning-disk hard drives. To some extent sequential writes are also preferable on flash-based SSD.
2. Concurrency and crash recovery are much simpler. 
   * No worry about crash while a value was being overwritten, leaving you with half old, half new value
3. Merging old segments avoid the problem of data files getting fragmented over time.

## Indexes
1. Index: The general idea behind them is to keep some additional metadata on the side, which acts as a signpost and helps you to locate the data you want. An index is an additional structure that is derived from the primary data. 
   * This is an important trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes
1. Key-Value Indexes
   * They like a primary key index in the relational model: unique
   * Hash Table Indexes with Append-Only Log Records
   * SSTable and LSM-Tree Structured Segment
   * B-trees
2. Secondary indexes
   * They are often crucial for performing joins efficiently
   * They can easily be constructed from a key-value index. The keys are not unique.
   * Both B-trees and log-structured indexes can be used

### Hash Indexes
1. We keep an in-memory hash map where every key is mapped to a byte offset in the data file—the location at which the value can be found by file seek. It's suited where the value for each key is updated frequently (e.g. count of a page view) but not too many distinct keys (so it's feasible to keep all keys in memory)

### How to Avoid Running Out of Disk Space
1. Break the log into segments of a certain size. 
2. Then perform *compaction* on the segments: throw away duplicate keys in the log, and keeping only the most recent update for each key.
3. We can also merge the segments. Delete the old segment files once new segment is generated
4. Each segment now has its own in-memory hash table, mapping keys to file offsets. So to find a key we need look at each segment's hashmap (from most recent to least recent). Merging helps reduce number of segments.

#### Important Considerations
1. File Format for log
   * Binary format: faster, simpler to first encode the length of a string in bytes, followed by the raw string
1. Deleting records
   * Tombstone: append a special deletion record to the data file, the key is deleted when merge
2. Crash Recovery
   * In-memory hash maps are lost if the DB is restarted. To avoid reading entire segment file from beginning to end, we can save a snapshot of each segment's hash map on disk, which can be loaded into memory more quickly.
3. Partially written records
   * The DB might crash halfway through appending a record to the log
   * We can use checksums to detect and ignore the corrupted records
4. Concurrency control
   * Use one writer thread
   * Can be read concurrently


#### Disadvantages of Hash Table Index
1. The hash table must fit in memory. 
   * It's difficult to make an on-disk hash map perform well. It requires a lot of random access I/O
   * It is expensive to grow when it becomes full, and hash collisions require fiddly logic.
1. Range queries are not efficient:
   * You have to look up each key individually in the hash maps

### SSTables Segments and LSM-Tree Algorithm
1. Sorted String Table or SSTable Format: 
   * Here we require that in the segment files, the sequence of key-value pairs is sorted by key.
   * Also each key only appears once within each merged segment file
1. Advantages over log segments with hash indexes
   * Merging segments is simple and efficient, even if the files are bigger than the memory (like mergesort)
   * You only need sparse index in memory to find the location of a key: No longer need to keep an index of all the keys in memory. 
      * One key for every few kilobytes of segment file is sufficient because a few kilobytes can be scanned very quickly.
   * Reduced disk space and I/O: since read requests need to scan over several key-value pairs in the requested range anyway, we can group those records into a block and compress it before writing it to disk.

#### Constructing and Maintaining SSTables
1. It's possible to use B-Trees to maintain a sorted structure on disk. But it's easier to maintain it in memory such as red-black trees or AVL trees
2. Steps for our storage engine
   * When a write comes in, add it to an in-memory balanced tree (memtable)
   * Write the memtable to disk as an SSTable file when its size is large. 
   * When a read comes in, first try to find the key in the memtable, then in the most recent on-disk segment, then the next-older, etc.
   * From time to time, run merge and compaction process
3. The problem
   * If the DB crashes, the most recent writes (in memtable but not in disk) are lost.
   * To avoid this: we can keep a separate log on disk to which every write is immediately appended. 
3. Storage engines that are based on this principle of merging and compacting sorted files are often called LSM (Log-Structured Merge) storage engines.
4. Products
   * Similar storage engines are used in Cassandra and HBase
   * Lucene, an indexing engine for full-text search used by Elasticsearch and Solr, uses a similar method for storing its term dictionary.

#### Performance Optimizations
1. LSM-tree algorithm can be slow when looking up keys that do not exist in the DB: 
   * Use Bloom filters: A Bloom filter is a memory-efficient data structure for approximating the contents of a set. It can tell you if a key does not appear in the database, and thus saves many unnecessary disk reads for nonexistent keys
1. Size-tiered and Leveled Compaction
   * Size-tiered: newer and smaller SSTables are successively merged into older and larger SSTables
   * Leveled: the key range is split up into smaller SSTables and older data is moved into separate "levels"
   * HBase uses size-tiered
   * Cassandra supports both
1. The basic idea of LSM-trees—keeping a cascade of SSTables that are merged in the background—is simple and effective. 
2. Even when the dataset is much bigger than the available memory it continues to work well. 
3. Since data is stored in sorted order, you can efficiently perform range queries (scanning all keys above some minimum and up to some maximum), 
4. Because the disk writes are sequential the LSM-tree can support remarkably high write throughput.


## Page-Oriented Storage Engines

### B-Trees
1. They remain the standard index implementation in almost all relational databases, and many nonrelational databases use them too
2. Like SSTables, B-trees keep key-value pairs sorted by key, which allows efficient key-value lookups and range queries.
3. The log-structured indexes we saw earlier break the database down into variable-size segments, typically several megabytes or more in size, and always write a segment sequentially. 
   * By contrast, B-trees break the database down into **fixed-size blocks or pages**, traditionally **4 KB in size** (sometimes bigger), and **read or write one page at a time**. This design corresponds more closely to the underlying hardware, as disks are also arranged in fixed-size blocks.
1. Each page can be identified using an address or location. One page is designated as the root of the B-tree; whenever you want to look up a key in the index, you start here. Each child is responsible for a continuous range of keys, and the keys between the references indicate where the boundaries between those ranges lie.
2. The number of references to child pages in one page of the B-tree is called the branching factor, it's typically several hundred
3. Update the value for an existing key in a B-Tree: search for the leaf page, update, write back
4. Add new key: if no space is available for new key, the page is split into two half-full pages. This ensures the tree remains *balanced* 
   * Most DBs can fit into a B-tree that is 3 or 4 levels deep. 
   * A four-level tree of 4KB pages with a branching factor of 500 can store up to 256TB.
5. Deleting a key while keeping the tree balanced is more involved .

### Making B-Trees Reliable
1. Orphan page: if DB crashes when you split a page into two and overwrite their parent page to update the references to the two child pages.
2. To make the DB resilient to crashes it's common to include additional data structure on disk: a **write-ahead log** (WAL, or redo log)
   * WAL log: an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself.
3. When updating pages, careful concurrency control is required if multiple threads are going to access B-tree at the same time. 
   * This is typically done by protecting the tree's data structures with *latches* or lightweight locks.

### B-tree Optimizations
1. Instead of WAL, some DBs use copy-on-write scheme, which is also useful for concurrency control.
2. We can save space in (internal) pages by not storing the entire key, but abbreviating it.
3. Layout leaf pages in sequential order on disk to make it efficient scanning a large part of the key range in sorted order. This is hard to maintain as tree grows. In contrast, LSM-trees makes it easier.
4. Have left/right pointers in leaf pages to go to previous/next leaf pages

### Compare B-Trees and LSM-Trees
1. Generally, LSM-trees are typicall faster for writes whereas B-trees are thought to be faster for reads. Because LSM have to check several different data structures and SSTables at different stages of compaction.

#### Advantages of LSM-Trees
1. Write Amplification
   * Write amplification (WA) is an undesirable phenomenon associated with flash memory and solid-state drives (SSDs) where the actual amount of information physically written to the storage media is a multiple of the logical amount intended to be written.
   * Because flash memory must be erased before it can be rewritten, with much coarser granularity of the erase operation when compared to the write operation, the process to perform these operations results in moving (or rewriting) user data and metadata more than once. Thus, rewriting some data requires an already-used-portion of flash to be read, updated, and written to a new location, together with initially erasing the new location if it was previously used at some point in time. Due to the way flash works, much larger portions of flash must be erased and rewritten than actually required by the amount of new data. This multiplying effect increases the number of writes required over the life of the SSD, which shortens the time it can operate reliably.
   * A B-tree index must write every data at least twice: once to WAL, once to the tree page itself. There's overhead from having to write an entire page at a time. 
   * Log-Structured indexes also rewrite data multiple times (compaction, merging of SSTables). 
   * Write amplification is of particular concern of SSDs, which can only overwrite blocks a limited number of times before wearing out.
   * In write-heavy applications, the performance bottleneck might be the rate at which the DB can write to disk.
      * Write amplification reduces write througput
   * LSM-trees are typically able to sustain higher write throughput than B-trees due to lower write amplification, sequentially write compact SSTable files (important on magnetic hard drives)

1. LSM-trees can be compressed better, produce smaller files on disk than B-trees.
   * B-tree leave some disk space unused due to fragmentation: when page split
   * LSM-trees periodically rewrite SSTables to remove fragmentation
   * Reduced data allows more read/write requests within available I/O bandwidth

#### Downsides of LSM-trees
1. The compaction can sometimes interfere with the performance of onging reads and writes.
   * Disk have limited resources
   * B-trees are more predictable
1. At high write throughput: the disk's finite write bandwidth needs to be shared between the initial write (logging and flushing a memtable to disk) and the compaction threads running in the backgroud. 
   * As DB grows larger, the more disk bandwidth is required for compaction
   * It may happen that compaction cannot keep up with the rate of incoming writes. It increases unmerged segments and you run out of disk space.
1. LSM-trees have multiple copies of the same key in different segments, whereas each key exists in one place of B-tree.
   * B-tree offers strong transactional semantics: transaction isolation locks on ranges of keys can be directly applied to B-tree.

### Other Indexing Structures

#### Storing values within the index
1. The value could be the actual row (document) or a reference to the row stored elsewhere (As MyISAM storage engine used in MySQl before v5.6). In the latter case, the place is called *heap file*, and it stores data in no particular order. It is common because it avoids duplicating data when multiple secondary indexes are present. 
2. When updating a value without changing the key, the heap file approach can be quite efficient when the new value size is smaller than the old value.
   * If the new value is larger in size, we may move it to some other place
   * Update all indexes or use a forwarding pointer
1. Clustered index: Sometimes it's desirable to store the value directly within an index
   * MySQL's InnoDB storage engiine: Only primary index does clustered index, other second indexes store the primary index value (along with secondary key) in their leaves. So they require two queries if search on secondary index.
   * They speed up reads, but require additional storage and add overhead on writes
   * Additional support for transanction guarantees on consistency

#### Multi-Column Indexes
1. Concatenated Index: combines several fields into one key
   * Important for geospatial data
   * Standard B-tree or LSM-tree can't answer that kind of query efficiently
#### Full-Text search and fuzzy indexes
1. All above indexes don't allow search for *similar keys*, e.g. misspelled words, called *fuzzy*  querying
2. Levenshtein automaton
#### Keeping Everything in Memory
1. The data structures discussed so far in this chapter have all been answers to the limitations of disks.
   * Disks are durable and lower cost per GB than RAM
3. In-memory Databases
   * Memcached are intended for caching only, data is lost if a machine restarted (Not persistent)
   * Other in-memory DBs aim for durability with battery-powered RAM) by writing a log of changes to disk, periodic snapshots to disk, replicating in-memory state to other machines
      * VoltDB, MemSQL, Oracle TimesTen
      * Redis, Couchbase: provide weak durability by writing to disk asynchronously
   * The disk is merely used as an append-only log for durability, and reads are served entirely from memory
4. The in-memory DBs can be faster because they can avoid the overheads of encoding in-memory data structures in a form that can be written to disk, not because they don't need to read from disk. Even a disk-based storage engine may never read from disk since OS caches recently used disk blocks in memory.
5. In-memory DBs provide data models that are difficult to implement with disk-based indexes
   * Redis offeres priority queues and sets
1. The so-called anti-caching approach works by evicting the least recently used data from memory to disk when there is not enough memory, and loading it back into memory when it is accessed again in the future. This is similar to what operating systems do with virtual memory and swap files, but the database can manage memory more efficiently than the OS, as it can work at the granularity of individual records rather than entire memory pages

## Transaction vs. Analytics
1. The storage engines fall into two broad categories: optimized for transaction processing (OLTP) and for anlytics (OLAP)

### OLTP Access Pattern
1. A transaction needn’t necessarily have ACID properties. Transaction processing just means allowing clients to make low-latency reads and writes—as opposed to **batch processing** jobs, which only run periodically (e.g. once per day). 
1. Typically user-facing, meaning huge volume of requests
1. Main read pattern: small number of records in each query, fetched by key
1. Main write pattern: random-access, low-latency writes from user input. 
   * Disk seek time is often the bottleneck here.
3. The application requests records using some kind of key, and the storage engine uses an index to find the data for the requested key.
4. Dataset size: Gigabytes to terabytes

### OLAP Access Pattern
1. Primiarly used by business analysts.
2. Main read pattern: They handle a much lower volume of queries than OLTP, but each query typically requires many millions of records to be scanned in a short time, only reading a few columns per record and calculates aggregate statistics rather than returning the raw data to the user. 
   * When your queries require sequentially scanning across a large number of rows, indexes are much less relevant. Instead it becomes important to encode data very compactly, to minimize the amount of data that the query needs to read from disk. 
3. Main write pattern: Bulk import (ETL, Extract-Transform-Load) or event stream (from OLTP DBs)
4. Disk bandwidth (not seek time) if often bottleneck
6. Column-oriented storage is a popular solution
7. Dataset size: Terabytes to petabytes
5. Storage Engines on OLTP
   * The log-structured school: SSTables, LSM-trees, Cassandra, HBase, Lucene
      * Key idea is to systematically turn random-access writes into sequential writes on disk, enable higher write throughput
   * The update-in-place, which treats the disk as a set of fixed-size pages that can be overwritten: B-trees being used in all major relational DBs and many nonrelational ones

#### Data Warehouse for OLAP
1. A data warehouse, by contrast, is a separate database that analysts can query to their hearts’ content, without affecting OLTP operations.
2. Products
   * Teradata, Vertica, SAP HANA
   * Amazon RedShift
   * SQL-on-Hadoop: Apache Hive, Spark SQL, Cloudera Impala, FB Presto

#### Storage Engines for OLAP
1. It turns out that the indexing algorithms discussed before work well for OLTP, but are not very good at answering analytic queries
#### Stars and Snowflakes: Schemas for Analytics
1. Many data warehouses are used in a fairly formulaic style, known as a star schema (also known as dimensional modeling
2. At the center of the schema is a so-called fact table. As each row in the fact table represents an event, the dimensions represent the who, what, where, when, how, and why of the event.
3. Other columns in the fact table are foreign key references to other tables, called dimension tables
4. A variation of this template is known as the snowflake schema, where dimensions are further broken down into subdimensions.
5. In a typical data warehouse, **tables are often very wide**: fact tables often have over 100 columns, sometimes several hundred

### Column-Oriented Storage
1. Although fact tables are often over 100 columns wide, a typical data warehouse query only accesses 4 or 5 of them at one time. 
2. In most OLTP databases, storage is laid out in a row-oriented fashion: all the values from one row of a table are stored next to each other. Document databases are simi‐ lar: an entire document is typically stored as one contiguous sequence of bytes.
3. When query by a few columns (even if they are indexed), a row-oriented storage engine still needs to load all of those rows (each consisting of over 100 attributes) from disk into memory, parse them, and filter out those that don’t meet the required conditions. That can take a long time
4. The idea behind column-oriented storage is simple: don’t store all the values from one row together, but store all the values from each column together instead. If each col‐ umn is stored in a separate file, a query only needs to read and parse those columns that are used in that query, which can save a lot of work.
5. Column storage is easiest to understand in a relational data model, but it applies equally to nonrelational data. For example, Parquet is a columnar storage format that supports a document data model.
6. The column-oriented storage layout relies on each column file containing the rows in the same order.
#### Column Compression
1. we can further reduce the demands on disk throughput by compressing data.
2. they often look quite repetitive, which is a good sign for compression. Often, the number of distinct values in a column is small compared to the number of rows
3. One technique that is particu‐ larly effective in data warehouses is **bitmap encoding**
   * We can now take a column with n distinct values and turn it into n separate bitmaps: one bitmap for each distinct value, with one bit for each row. The bit is 1 if the row has that value, and 0 if not.
   * Take a matrix of nxm, where n is the unique number of values, and m is the number of rows. So we only need n bitmaps to represent all data in this column. Each bitmap has a length of m.
1. If n is very small (for example, a country column may have approximately 200 dis‐ tinct values), those bitmaps can be stored with one bit per row. But if n is bigger, there will be a lot of zeros in most of the bitmaps (we say that they are sparse). In that case, the bitmaps can additionally be **run-length encoded (RLE)**.
1. Cassandra and HBase have a concept of column families, which they inherited from Bigtable. However, it is very misleading to call them column-oriented: within each column family, they store all columns from a row together, along with a row key, and they do not use column compression. Thus, the Bigtable model is still mostly row-oriented.
2. Cassandra:
  * Each table requires a unique *primary key*. The first field listed is the partition key, since its hashed value is used to determine the node to store the data. If those fields are wrapped in parentheses then the partition key is composite. Otherwise the first field is the partition key. Any fields listed after the partition key are called clustering columns. These store data in ascending or descending order within the partition for the fast retrieval of similar values. All the fields together are the primary key.


#### Memory bandwidth and vectorized processing
1. For data warehouse queries that need to scan over millions of rows, a big bottleneck is the bandwidth for getting data from disk into memory.
2. Developers of analytical databases also worry about efficiently using the bandwidth from main memory into the CPU cache, avoiding branch mispredictions and bubbles in the CPU instruction processing pipeline, and making use of single-instruction-multi-data (SIMD) instructions in modern CPUs
3. Besides reducing the volume of data that needs to be loaded from disk, column- oriented storage layouts are also good for making efficient use of CPU cycles.
4. The query engine can take a chunk of compressed column data that fits comfortably in the CPU’s L1 cache and iterate through it in a tight loop (that is, with no function calls). A CPU can execute such a loop much faster than code that requires a lot of function calls and conditions for each record that is processed. 
5. Column compression allows more rows from a column to fit in the same amount of L1 cache. Operators, such as the bitwise AND and OR described previously, can be designed to operate on such chunks of compressed column data directly. This technique is known as **vectorized processing**

#### Sort Order in Column Storage
1. In a column store, it doesn’t necessarily matter in which order the rows are stored. It’s easiest to store them in the order in which they were inserted, since then inserting a new row just means appending to each of the column files.
1. Note that it wouldn’t make sense to sort each column independently, because then we would no longer know which items in the columns belong to the same row.
2. Another advantage of sorted order is that it can help with compression of columns. A simple run-length encoding, could compress that column down to a few kilobytes—even if the table has billions of rows.
3. Several Different Sort Orders
   *  Having multiple sort orders in a column-oriented store is a bit similar to having mul‐ tiple secondary indexes in a row-oriented store
   * row- oriented store keeps every row in one place (in the heap file or a clustered index), In a column store, there normally aren’t any pointers to data elsewhere, only columns containing values 

#### Writing to Column-Oriented Storage
1. Writes are difficult for Column-Oriented Storage
2. An update-in-place approach, like B-trees use, is not possible with compressed col‐ umns. If you wanted to insert a row in the middle of a sorted table, you would most likely have to rewrite all the column files.
3. Fortunately, we have already seen a good solution earlier: LSM-trees. All writes first go to an in-memory store, where they are added to a sorted structure and prepared for writing to disk.
4. Queries need to examine both the column data on disk and the recent writes in mem‐ ory, and combine the two. However,

#### Aggregation: Data Cubes and Materialized Views
1. materialized aggregates: used to cache the aggregate queries
2. The difference is that a materialized view is an actual copy of the query results, written to disk, whereas a virtual view is just a shortcut for writ‐ ing queries. 
3. A common special case of a materialized view is known as a data cube or OLAP cube [64]. It is a grid of aggregates grouped by different dimensions.
5. The advantage of a materialized data cube is that certain queries become very fast because they have effectively been precomputed. For
6. The disadvantage is that a data cube doesn’t have the same flexibility as querying the raw data. For

# SQL and NoSQL Databases
1. Many NoSQL DBs are optimized for write. 
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
   * Memcached: 
      * set/list not supported in value
      * both key and values are strings
      * cache单机的，单机的内存有限制，并且只能本机访问，memcache是一个cache db，可以由多台机器组成，并且可以多个server同时访问
   * Redis
      * support set/list in value, can add/append to value
      * a bit lower in performance than Memcached, but still in the order of memory (是CPU高速缓存，分为L1、L2、L3三个等级)
      * Can be used in cache/message queue/DB
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
3. Cassandra
   * Simple query, not complex data structure
   * 

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


### MySQL
1. Response time to request rate increases linearly. Some technologies don’t respond as well when the request rate spikes.

### Why Redis?
1. Provides a variety of data structures.
1. Has persistence and replication, with selectability on how you want them implemented. If you want MySQL style persistence you can have it. If you want no persistence you can have it. Or if you want 3 hour persistence you can have it.
1. Your home feed is on Redis and is saved every 3 hours. There’s no 3 hour replication. They just backup every 3 hours.
1. If the box your data is stored on dies, then they just backup a few hours. It’s not perfectly reliable, but it is simpler. You don’t need complicated persistence and replication. It’s a lot simpler and cheaper architecture.
1. Few failure modes. Has a few subtle failure modes that you need to learn about. This is where maturity comes in. Just learn them and get past it.


### Solr
1. Doesn’t scale past one box. (not so with latest release)
1. Tried Elastic Search, but at their scale it had trouble with lots of tiny documents and lots of queries.

### Clustering Vs Sharding
1. During rapid growth they realized they were going to need to spread their data evenly to handle the ever increasing load.
1. Defined a spectrum to talk about the problem and they came up with a range of options between Clustering and Sharding.

### Clustering - Everything Is Automatic:
1. Examples: Cassandra, MemBase, HBase
1. Verdict: too scary, maybe in the future, but for now it’s too complicated and has way too many failure modes.
1. Properties:
   * Data distributed automatically
   * Data can move
   * Rebalance to distribute capacity
   * Nodes communicate with each other. A lot of crosstalk, gossiping and negotiation.

1. Pros:
   * Automatically scale your datastore. That’s what the white papers say at least.
   * Easy to setup.
   * Spatially distribute and colocate your data. You can have datacenter in different regions and the database takes care of it.
   * High availability
   * Load balancing
   * No single point of failure

1. Cons (from first hand experience):
   * Still fairly young.
   * Fundamentally complicated. A whole bunch nodes have to symmetrical agreement, which is a hard problem to solve in production.
   * Less community support. There’s a split in the community along different product lines so there’s less support in each camp.
   * Fewer engineers with working knowledge. Probably most engineers have not used Cassandra.
   * Difficult and scary upgrade mechanisms. Could be related to they all use an API and talk amongst themselves so you don’t want them to be confused. This leads to complicated upgrade paths.
   * Cluster Management Algorithm is a SPOF. If there’s a bug it impacts every node. This took them down 4 times.
   * Cluster Managers are complex code replicated over all nodes that have the following failure modes:
     * Data rebalance breaks. Bring a new box and data starts replicating and then it gets stuck. What do you do? There aren’t tools to figure out what’s going on. There’s no community to help, so they were stuck. They reverted back to MySQL.
     * Data corruption across all nodes. What if there’s a bug that sprays badness into the write log across all of them and compaction or some other mechanism stops? Your read latencies increase. All your data is screwed and the data is gone.
     * Improper balancing that cannot be easily fixed. Very common. You have 10 nodes and you notice all the node is on one node. There’s a manual process, but it redistributes the load back to one node.
     * Data authority failure. Clustering schemes are very smart. In one case they bring in a new secondary. At about 80% the secondary says it’s primary and the primary goes to secondary and you’ve lost 20% of the data. Losing 20% of the data is worse than losing all of it because you don’t know what you’ve lost.

### Sharding - Everything Is Manual:
1. Verdict: It’s the winner. Note, I think their sharding architecture has a lot in common with Flickr Architecture.
1. Properties:
   * Get rid of all the properties of clustering that you don’t like and you get sharding.
   * Data distributed manually
   * Data does not move. They don’t ever move data, though some people do, which puts them higher on the spectrum.
   * Split data to distribute load.
   * Nodes are not aware of each other. Some master node controls everything.

1. Pros:
   * Can split your database to add more capacity.
   * Spatially distribute and collocate your data
   * High availability
   * Load balancing
   * Algorithm for placing data is very simple. The main reason. Has a SPOF, but it’s half a page of code rather than a very complicated Cluster Manager. After the first day it will work or won’t work.
   * ID generation is simplistic.

1. Cons:
   * Can’t perform most joins.
   * Lost all transaction capabilities. A write to one database may fail when a write to another succeeds.
   * Many constraints must be moved to the application layer.
   * Schema changes require more planning.
   * Reports require running queries on all shards and then perform all the aggregation yourself.
   * Joins are performed in the application layer.
   * Your application must be tolerant to all these issues.

### When To Shard?
1. If your project will have a few TBs of data then you should shard as soon as possible.
1. When the Pin table went to a billion rows the indexes ran out of memory and they were swapping to disk.
1. They picked the largest table and put it in its own database.
1. Then they ran out of space on the single database.
1. Then they had to shard.

### Transition To Sharding
1. Started the transition process with a feature freeze.
1. Then they decided how they wanted to shard. Want to perform the least amount of queries and go to least number of databases to render a single page.
1. Removed all MySQL joins. Since the tables could be loaded into separate partitions joins would not work.
1. Added a ton of cache. Basically every query has to be cached.
1. The steps looked like:
   * 1 DB + Foreign Keys + Joins
   * 1 DB + Denormalized + Cache
   * 1 DB + Read Slaves + Cache
   * Several functionally sharded DBs + Read slaves + Cache
   * ID sharded DBs + Backup slaves + cache
1. Earlier read slaves became a problem because of slave lag. A read would go to the slave and the master hadn’t replicated the record yet, so it looked like a record was missing. Getting around that require cache.
1. They have background scripts that duplicate what the database used to do. Check for integrity constraints, references.
1. User table is unsharded. They just use a big database and have a uniqueness constraint on user name and email. Inserting a User will fail if it isn’t unique. Then they do a lot of writes in their sharded database.

### How To Shard?
1. Looked at Cassandra’s ring model. Looked at Membase. And looked at Twitter’s Gizzard.
1. Determined: the least data you move across your nodes the more stable your architecture.
1. Cassandra has a data balancing and authority problems because the nodes weren’t sure of who owned which part of the data. They decided the application should decide where the data should go so there is never an issue.
1. Projected their growth out for the next five years and presharded with that capacity plan in mind.
1. Initially created a lot of virtual shards. 8 physical servers, each with 512 DBs. All the databases have all the tables.
1. For high availability they always run in multi-master replication mode. Each master is assigned to a different availability zone. On failure the switch to the other master and bring in a new replacement node.

1. When load increasing on a database:
   * Look at code commits to see if a new feature, caching issue, or other problem occurred.
   * If it’s just a load increase they split the database and tell the applications to find the data on a new host.
   * Before splitting the database they start slaves for those masters. Then they swap application code with the new database assignments. In the few minutes during the transition writes are still write to old nodes and be replicated to the new nodes. Then the pipe is cut between the nodes.

### ID Structure
1. 64 bits:
   * shard ID: 16 bits
   * type : 10 bits - Pin, Board, User, or any other object type
   * local ID - rest of the bits for the ID within the table. Uses MySQL auto increment.
1. Twitter uses a mapping table to map IDs to a physical host. Which requires a lookup. Since Pinterest is on AWS and MySQL queries took about 3ms, they decided this extra level of indirection would not work. They build the location into the ID.
1. New users are randomly distributed across shards.
1. All data (pins, boards, etc) for a user is collocated on the same shard. Huge advantage. Rendering a user profile, for example, does not take multiple cross shard queries. It’s fast.
1. Every board is collocated with the user so boards can be rendered from one database.
1. Enough shards IDs for 65536 shards, but they only opened 4096 at first, they’ll expand horizontally. When the user database gets full they’ll open up more shards and allow new users to go to the new shards.

### Lookups
1. If they have 50 lookups, for example, they split the IDs and run all the queries in parallel so the latency is the longest wait.
1. Every application has a configuration file that maps a shard range to a physical host.
   * “sharddb001a”: : (1, 512)
   * “sharddb001b”: : (513, 1024) - backup hot master
1. If you want to look up a User whose ID falls into sharddb003a:
   * Decompose the ID into its parts
   * Perform the lookup in the shard map
   * Connect to the shard, go to the database for the type, and use the local ID to find the right user and return the serialized data.

Objects And Mappings
All data is either an object (pin, board, user, comment) or a mapping (user has boards, pins has likes).

For objects a Local ID maps to a MySQL blob. The blob format started with JSON but is moving to serialized thrift.

For mappings there’s a mapping table.  You can ask for all the boards for a user. The IDs contain a timestamp so you can see the order of events.

There’s a reverse mapping, many to many table, to answer queries of the type give me all the users who like this pin.

Schema naming scheme is noun_verb_noun: user_likes_pins, pins_like_user.

Queries are primary key or index lookups (no joins).

Data doesn’t move across database as it does with clustering. Once a user lands on shard 20, for example, and all the user data is collocated, it will never move. The 64 bit ID has contains the shard ID so it can’t be moved. You can move the physical data to another database, but it’s still associated with the same shard.

All tables exist on all shards. No special shards, not counting the huge user table that is used to detect user name conflicts.

No schema changes required and a new index requires a new table.

Since the value for a key is a blob, you can add fields without destroying the schema. There’s versioning on each blob so applications will detect the version number and change the record to the new format and write it back. All the data doesn’t need to change at once, it will be upgraded on reads.

Huge win because altering a table takes a lock for hours or days.  If you want a new index you just create a new table and start populating it. When you don’t want it anymore just drop it. (no mention of how these updates are transaction safe).

Rendering A User Profile Page
Get the user name from the URL. Go to the single huge database to find the user ID.

Take the user ID and split it into its component parts.

Select the shard and go to that shard.

SELECT body from users WHERE id = <local_user_id>

SELECT board_id FROM user_has_boards WHERE user_id=<user_id>

SELECT body FROM boards WHERE id IN (<boards_ids>)

SELECT pin_id FROM board_has_pins WHERE board_id=<board_id>

SELECT body FROM pins WHERE id IN (pin_ids)

Most of the calls are served from cache (memcache or redis), so it’s not a lot of database queries in practice.

Scripting
When moving to a sharded architecture you have two different infrastructures, one old, the unsharded system, and one new, the sharded system. Scripting is all the code to transfer the old stuff to the new stuff.

They moved 500 million pins and 1.6 billion follower rows, etc

Underestimated this portion of the project. They thought it would take 2 months to put data in the shards, it actually took 4-5 months. And remember, this was during a feature freeze.

Applications must always write to both old and new infrastructures.

Once confident that all your data is in the new infrastructure then point your reads to the new infrastructure and slowly increase the load and test your backends.

Built a scripting farm. Spin up more workers to complete the task faster. They would do tasks like move these tables over to the new infrastructure.

Hacked up a copy of Pyres, a Python interface to Github’s Resque queue, a queue on built on top of redis. Supports priorities and retries. It was so good they replaced Celery and RabbitMQ with Pyres.

A lot of mistakes in the process. Users found errors like missing boards. Had to run the process several times to make sure no transitory errors prevented data from being moved correctly.

Development
Initially tried to give developers a slice of the system. Each having their own MySQL server, etc, but things changed so fast this didn’t work.

Went to Facebook’s model where everyone has access to everything. So you have to be really careful.

Future Directions
Service based architecture.

As they are starting to see a lot of database load they start to spawn a lot of app servers and a bunch of other servers. All these servers connect to MySQL and memcache. This means there are 30K connections on memcache which takes up a couple of gig of RAM and causes the memcache daemons to swap.

As a fix these are moving to a service architecture. There’s a follower service, for example, that will only answer follower queries. This isolates the number of machines to 30 that access the database and cache, thus reducing the number of connections.

Helps isolate functionality. Helps organize teams around around services and support for those services. Helps with security as developer can’t access other services.

Lessons Learned
It will fail. Keep it simple.

Keep it fun. There’s a lot of new people joining the team. If you just give them a huge complex system it won’t be fun. Keeping the architecture simple has been a big win. New engineers have been contributing code from week one.

When you push something to the limit all these technologies fail in their own special way.

Architecture is doing the right thing when growth can be handled by adding more of the same stuff. You want to be able to scale by throwing money at the problem by throwing more boxes at the problem as you need them. If your architecture can do that, then you’re golden.

Cluster Management Algorithm is a SPOF. If there’s a bug it impacts every node. This took them down 4 times.

To handle rapid growth you need to spread data out evenly to handle the ever increasing load.

The least data you move across your nodes the more stable your architecture. This is why they went with sharding over clusters.

A service oriented architecture rules. It isolates functionality, helps reduce connections, organize teams, organize support, and  improves security.

Asked yourself what your really want to be. Drop technologies that match that vision, even if you have to rearchitecture everything.

Don’t freak out about losing a little data. They keep user data in memory and write it out periodically. A loss means only a few hours of data are lost, but the resulting system is much simpler and more robust, and that’s what matters.




# RocksDB
1. Key-Value NoSQL DB
2. set/list not supported in value
3. Many companies use it as the bottom layer of the key-value store










