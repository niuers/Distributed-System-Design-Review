# General Advices:
1. Interviewee should drive the conversation and move forward


# Step 1: Outline use cases, constraints, and assumptions
1. Gather requirements and scope the problem. Ask questions to clarify use cases and constraints. Discuss assumptions.
2. Think about data, what data, how it's get in and how it's get out of the system. Don't worry about time too much.

## Requirement Clarification
1. Users/Consumers: 
   * Who is going to use it?
   * How are they going to use it?
      * Real time
      * In-frequent usage
   * What does the system do?
   * What are the inputs and outputs of the system?
2. Scale (Read and Write):
   * How many users are there?
   * How many read requests per second do we expect?
   * How much data is queried per request?
   * What is the expected read to write ratio?
   * How many videos view are processed per second?
   * Can there be spikes in the traffic and how big are they?
   * How much data do we expect to handle?   
3. Performance
   * How fast our system must be? 
   * What is the expected write-to-read delay?
   * What is the expected 99% latency for read queires?
   * How fast data must be retried from the system?
4. Cost
   * What's the budget constraint?
   * Should the design to minimize development cost?
      * Use open source
   * Should the design to minimize the cost of maintenance? 
      * Consider public cloud service

## Write Down Functional and Non-Functional Requirements

### Functional Requirements: System behaviors or APIs
1. Write down a sentence for each functional requirement
2. Write down an API by following the sentence
    * Try generalize the APIs instead of just for a specific input, or type
       * count --> statistics
       * view --> EventType 
    * Create class for your use case
    * Identify proper input parameters
3. Apply the same technique to data input/output process

### Non-Functional Requirements
1. System qualities
   * Scalable
      * Tens of thousands of views per second
   * High Performant
      * Few tens of milliseconds to return total views count for a video
   * High Available(Fault Tolerant)
      * Survives hardware/network faults, no single point of failure 
   * Consistency
   * Cost Minimization: hardware, development, maintenance
   * Secure
1. Write down non-functional requirements
1. Focus on Scalability, Performance, Availability
1. Need find trade-offs between different qualities 
1. CAP Theorem
   * It tells me that I should be choosing between Availability and consistency, we choose availability, i.e. prefer to show stale data over no data at all.


# Step 2: Create a high level design
1. Outline a high level design with all important components.
2. Sketch the main components and connections
3. Justify your ideas

## Process
1. Start with a Database to store the data
2. Have a web service to get input data from outside
3. Have a web service to output data to outside
4. We start with something simple and construct the frame with the outside pieces, i.e. define data model
   * The outside piece is data
   * What data we need to store and how to do it
      * Individual event (per click) vs. aggregate data (per minute) in real-time
   * Individual event
      * Fast writes
      * Can slice and dice data however we want
      * Can recalculate numbers if needed
      * Slow reads if asked for counts
      * High cost to store all events
   * Aggregate data
      * Fast reads
      * Data is ready for real-time decision making
      * Can only query data the way it's aggregated
      * Requires data aggregation (in memory) pipeline
      * Hard or even impossible to fix errors, e.g. bug in aggregation
   * We need interviewer to help decide which one to choose
      * What's the expected data delay (time between when event happened and when it's processed)
         * Several minutes (aggregate) vs. Several hourse (individual event)
         * This is stream data process vs. batch data process
         * Combining both approaches makes a lot of sense for many systems
            * Store several days of individual events and aggregate data at the same time
            * Price: The system becomes more complex and more expensive
6. Where to store the data?
   * The interviewer wants to know the name of the database and why we make the choice? 
      * Both SQL and NoSQL databases can scale and perform well
      * Evaluate database against the non-functional requirements we wrote down before
   * How to scale reads?
   * How to scale writes?
   * How to make both reads and writes fast?
   * How not to lose data in case of hardware faults and network partitions?
   * How to achieve strong consistency? What are the tradeoffs?
   * How to recover data in case of outage? 
   * How to ensure data security? 
   * How to make it entensible for data model changes in the future? 
   * Where to run (cloud vs. on-premises data centers)? 
   * How much money will it all cost? 
8. SQL Databases (MySQL)
   * Sharding or horizontal partitioning: when data is large we split them into multiple database machines
   * Services need to know how many machines exist, which one to pick and store the data.
      * Option 1: Make service to call every database machine directly 
      * Option 2: Introduce 'Cluster Proxy', which knows about all DB machines, and routes traffic to the correct shard
   * Configuration service (ZooKeeper)
      * Proxy has to know when some shard dies or become unavailable due to network, new shard awareness
      * Configuration service maintains a health check connection to all shards
   * To avoid calling  DB instance directly, we introduct shard proxy, so 'cluster proxy' calls 'shard proxy'
      * It caches query results
      * Monitor DB instance health, 
      * publish metrics
      * Terminate queries that takes too long to run
   * To increase availability, we need replicate data
      * Master/Leader shard with read replica (which only allow reads), the replicas can be in a different data center than the master shard
10. NoSQL Databases (Apache Cassandra)
   * We also split data into chunks (shards) or nodes.
      * Each shard is equal, no leader and followers
      * No configuration service needed, Shards talks to each other and exchange information
      * Gossip Protocol: To reduce network load, shards only talk to less than 3 other shards every second
      * No cluster proxy needed, Every node knows about each other, and can forward request to corresponding node
   * We can use round robin to choose inital node to process request, or choose the node with the shortest distance to client
      * The initial node is called: coordinator node. The coordinator decides which node to process the input data
         * We can use **consistent hashing** algorithm to pick the node
         * N.B. **consisten hasing** is also used to design distributed cache
      * It uses **quorum writes** to write to replicas (asynchronously, since synchronous writing is slow)
      * Similarly, there's **quorum reads** approach
      * Cassandra uses version  number to determine staleness of data
   * Consistency
      * Eventual consistency: in SQL DB, although some followers may lag behind leader DB in terms of data staleness, they'll eventuall become the same.
      * Cassandra offers **tunable consistency**: 
   * Four types of NoSQL DBs
      * Column: 
         * Cassandra is a wide column DB that supports asynchronous masterless replication
         * HBase has master-based replication
      * Document Oriented: MongoDB uses leader based replication
      * Key-Value
      * Graph
   * Cassandra
      * Fault tolerant: support multi-data center replication
      * Scalable: both read and write throughput increases linearly as new machines are added
      * Works well with time series data
   * We need to know the advantages/disadvantages of different databases, and when to use what
11. How we store the data?
   * SQL
      * We usually start by defining noun in the system. 
      * We then convert nouns into tables, and use foreign keys to reference related data
      * Data is normalized: we minimize data duplication across different table
   * NoSQL
      * Think in terms of queries to be executed on the system
      * Denormalization is perfectly normal
         * Instead of adding rows in relational database, we add columns for every hour
12. Data Processing Service        
   * Start with the written requirements: Scalable, reliable, fast
     * How to scale?
     * How to achieve high throughput?  
     * How not to lose data when processing node crashes?
     * What to do when database is unavailable or slow? 
   * Scalable: partitioning
   * Reliable: Replication and checkpointing
   * Fast: in-memory, minimize disk reads
   * Data Aggregation Basics
      * Write count to DB each time we see an event
      * Save current count in memory and write to DB every other second (better for large scale system)
   * PUSH or PULL
      * push means that some other services sends events  synchronously to the processing service
      * pull means the processing service pulls event from some temporary storage (e.g. a queue)
      * In this case, pull has more advantages
         * better fault-tolerant support
         * easier to scale
   * Checkpointing
      * After we have processed several events in the temporary storage (e.g. a queue) and successfully stored them in a DB, we write checkpoint to some persistent storage. 
      * If processing service machine fails, the new machine will resume processing where the failed machine left off. 
   * Partitioning
      * Instead of putting all events into a single queue, let's have several independent queues.
      * Every queue physically lives on its own machine, and stores a subset of all events
      * This allows us to parallize event processing, 
   * Processing service reads events from the queue , counts events in memory, and wirtes to DB periodically, 
      * Components to read events: partition consumer establishes and maintains TCP connection with the event partition to fetch data
         * Think of it as an infinite loop that polls data from the partition
         * It deserializes the event from byte array to actual event object
         * Consumer is usually a single threaded component
         * Consumer helps to eliminte duplicate events through a distributed "deduplication cache" which stores unique event's identifiers for, say last 10 minutes
      * Aggrator then does the in-memory counting, e.g. hash table
      * The counts were sent to an internal queue, which decouples consumption and processing, thus it's possible to increase the processing throughput
      * Then DB writer (single or multiple threaded) writes the data to DB
         * With multi-threaded, it's harder to use checkpoint though, even it increases throughput    
      * The Dead-Letter Queue 
         * The queue to which messages are sent if they cannot be routed to their correct destination
         * This is to protect us from from database performance of availability issues
         * There's another service that reads the dead-letter queue and write to DB
         * Widely used when you need perserve data in case of downstream services degradation
         * Another viable option is to store undelivered messages on a local disk of the processing service machine, 
      * Data Enrichment
         * Embeded Database: This DB lives on the same machine as the processing service
         * All additional attributes (e.g. extra things you want to display) should be retrieved from this DB really quickly
      * State Management
         * Every time we keep anything in memory, we need to understand what to do when machine fails and this in-memory state is lost
         * Solution is to periodically save the entire in-memory data to a durable storage. New machine just reloads  this state int its memory when started.

14. Partitioner Service
   * Distribute data across partitions
15. Load Balancer
   * To evenly distribute events across partitioner service machines
16. API Gateway
   * Represents a single-entry point into a video content delivery system.
   * It routes client requests to to backend services
17. Partitioner Service Client
   * 
18. 






# Step 3: Design core components
Dive into details for each core component. For example, if you were asked to design a url shortening service, discuss:

  * Generating and storing a hash of the full url
  * MD5 and Base62
  * Hash collisions
  * SQL or NoSQL
  * Database schema
  * Translating a hashed url to the full url
  * Database lookup
  * API and object-oriented design

# Step 4: Scale the design
Identify and address bottlenecks, given the constraints. For example, do you need the following to address scalability issues?

Load balancer
Horizontal scaling
Caching
Database sharding
Discuss potential solutions and trade-offs. Everything is a trade-off. Address bottlenecks using principles of scalable system design.

Back-of-the-envelope calculations
You might be asked to do some estimates by hand. Refer to the Appendix for the following resources:

Use back of the envelope calculations
Powers of two table
Latency numbers every programmer should know


# References
1. [How to approach a system design interview question](https://github.com/donnemartin/system-design-primer#how-to-approach-a-system-design-interview-question)
2. [System Design Interview – Step By Step Guide](https://youtu.be/bUHFg8CZFws)
