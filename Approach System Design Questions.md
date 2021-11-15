# General Advices:
1. Interviewee should drive the conversation and move forward
3. We should always know about the trade offs of our options. 

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
15. [Load Balancer](https://github.com/niuers/Distributed-System-Design-Review/blob/main/Load%20Balancer.md)
   * To evenly distribute events across partitioner service machines
16. API Gateway
   * Represents a single-entry point into a video content delivery system.
   * It routes client requests to to backend services
17. Partitioner Service Client
   * 
## Partitioner Service Client
### Blocking vs. Non-Blocking I/O
1. Client initiates request by using sockets. When a client makes a request, the socket on the server side is blocked. The thread handles the connection is blocked as well.
2. To handle new request, we need create a new thread to handle the new connection.
3. When the machine slows down with increasing number of requests and eventually dies, it may bring down the full cluster.
4. This is why we need rate limiting solution to keep system stable during traffic peeks.
5. Non-blocking I/O:
   * A single thread can handle multiple concurrent connections
   * Server queues the requests, and the actual I/O is processed later.
   * It's far less expensive to pile up requests in queue than pile up threads. 
   * This is more efficient and has higher throughput.
   * This however increases the complexity of operations. Very hard to debug.
### Buffering and Batching
1. To process large number of requests, API gateway cluster has to be big in size, e.g. thousands of machines.
2. If we pass individual events to partitioner service, partitioner service cluster has to be big as well.
3. It's more efficient to combine events together and send several of them in a single request.
4. We put events into buffer, and wait a couple of seconds before sending buffer's contents
5. It increases throughput, save on cost, request compression is more effective.
6. It introduces some complexity on both client and server sides. 
   * If some events failed when partitioner server process the batch, what shall we do? re-send whole batch, re-send the failed ones only? 
### Timeout
1. It defines how much time a client is willing to wait for a response from a server.
2. Two types
   * Connection Timeout: defines how much time a client is willing to wait for a connection to establish
      * Usually very small, tens of milliseconds
   * Request Timeout : happens when request processing takes too much time, and a client is not willing to wait any longer.
      * To choose request timeout value, we need analyze latency percentiles.
      * For example, we measure latency of 1% of the slowest requests in the system, and set this value as request timeout. It means that about 1% of the requests will timeout.
### Retries
1. What to do with failed requests?
   * Re-try them: maybe hit a new machine? 
2. Need to be smart when doing re-try
   * If all clients retry at the same time, or do it aggressively, we may create so-called **retry storm event**,  and overload server with too many requests.
### Exponential Backoff and Jitter
1. To prevent **retry storm event**
2. Exponential Backoff:
   * Increases the waiting time between retries up to a maximum backoff time
   * We retry requests several times, but wait a bit longer with every retry attempt
3. Jitter
   * It adds randomness to retry intervals to spread out the load
   * If we don't add jitter, backoff algorithms will retry requests at the same time. 
   * It helps to separate retries.
4. We may still be in the danger of too many retries
   * Ex. when partitioner service is down or degraded., and majority requests are retried.
### Circuit Breaker 
1. The circuit breaker pattern stops a client from repeatedly trying to execute an operation that's likely to fail.
2. We calculate how many requests have failed recently and if error threshold is exceeded, we stop calling a downstream service.
3. Some time later, limited number of requests  from the client are allowed to pass through and invoke the operation.
4. If the failed requests are successful, we restart counting failed requests from scratch.
5. It makes the system more difficult to test. It's also hard to properly set error threshold and timers. 

### DNS (Domain Name System)
1. How does our partitioner service client know about load balancer? 
   * DNS is a phonebook of internet, it maintains a directory of domain name and translate to IP address.
   * We register our partitioner service in DNS, specify domain name, e.g. partitionerservice.domain.com, and associate it with the IP address of load balancer service, so when clients hit domain name, the requests are forwarded to the load balancer device. 
3. How does the load balancer know about the partitioner service machines ? 
   * We need explicitly tell the load balancer the IP address of each machine, 
   * Both software/hardware load balancers provide API to register and unregister servers.  
### Health Checking
   * Load balancer needs to know which server from the registered list are healthy and which are unavailable. So it can ensure the traffic is enrouted to the healthy servers. 
   * Load balancer pings each server periodically and if unhealthy server is identified, load balancer stops to send traffic to it. 
### High Availability
1. How does load balancer guarantee high availability? 
   * It uses the concept of primary and secondary nodes
   * The primary nodes accept connections and serve requests, while the seondary monitors the primary.
   * If the primary is unable to accept connections, secondary takes over. 
   * Primary and secondary live in different data center. 

## Partitioner Service and Partitions
1. It's a web service that gets requests from clients, looks inside each request to retrieve each individual video events (remember batch? ), and routes each event (or message) to some partition. 
### Partitions
1. Partitions are also a web service, that gets messages and stores them on disk in the form of append-only log file. 
2. So we have totally ordered sequence of messages ordered by time.
3. This is not a single very large log file, but a set of log files of the predefined size. 
### Partition Strategy
1. Partitioner service has to use some rule, partition strategy, that defines which partition gets what messages.
2. Calculate hash function based on some key, and choose machine based on this hash
   * This doesn't work well with large scale. It may lead to **hot partitions**
   * We can include event time, e.g. in minutes, into the partition key. 
   * Or We can split the hot partitions into two new partitions. Remember **Consistent Hash Algirhtm**
      * Adding a new node to the consistent hashing ring splits a range of keys into two new ranges. 
   * Or we may allocate dedicated partitions to some hot partitions. 
### Service Discovery
1. To send messages to partitions, partitioner service needs to know every partition
2. Two main service discovery patterns
   * Server-side, e.g. load balancer
      * Clients know load balancer, load balancer knows about server-side instances
      * But we don't need a load balancer between partition service and partitions.
      * Partitioner service itself acts like load balancer by distributing events over partitiosn
      * This is perfect for client side discovery
   * Client-side
      * Every server instance registers itself in some common place, named **service registry** 
      * Service Registry is another highly available web service, which can perform health checks to determine health of each registered isntance.
      * Clients then query service registry and obtain a list of avaiable servers. 
      * Example of such registry service is Zookeeper. 
      * Each partition registers itself in Zookeeper, while every partitioner service instances queries Zookeeper for the list of partitions. 
      * Distributed Cache Design: when cache client needs to pick a cache shard that stores the data, where we can use client-side service discovery as well.
  
3. One more option for service discovery is like Cassandra
   * Cassandra nodes talk to each other
   * So clients only needs to contact one node from the server cluster to figure out the information about the whole cluster.
### Replication
1. When event is persistent in partition, we need to replicate it in the case the partiton machine goes down.
2. Three main approaches in replication
   * Single Leader
      * Each partition has a leader and several followers
      * Leader checks all followers, and keep a list of active followers
      * Related: how to scale the SQL database
      * Related concept: Quorum write
         * When partition service makes a call to a partition, we may send response back as soon as leader partition persisted the message, or only when message was  replicated to a specified number of replicas.
         * When we write only to leader, we may still lose data if the leader goes down before replication really happened.
         * If we wait for the repliation to complete, we increase durability, but increases latency. Plus, if the required number of replications is not available at the moment,availability will suffer.
   * Multiple Leader
      * Mostly used to replicate in several data centers
   * Leaderless Replication 
      * Related: when we discuss how Cassandra works

### Message Formats
1. Textual format
   * XML
   * JSON
      * Every message contains field names, which greatly increase total message size. 
   * CSV 
   * Human readable, well-known, widely supported, and heavily used by many distributed systems. 
3. Binary Format
   * Apache Thrift
      * Use field tags
   * Protocal Buffer
      * Use field tags, numbers, act like alias to field names
   * Avro: good choice for counting system.
   * It provide much benefits for large scale real time processing systems. 
   * Messages are more compact, faster to parse. 
      * They use schema, no longer need field names, 
      * Tags occupy less space when encoded
   * Message producers (clients) need to know the schema to serialize the data 
   * Message consumers (processing service) require the schema to deserialize the data. 
   * So schemas are usually stored in some shared database, 
   * Schema may change  over time
      * 
## Data Retrieval  Path
### Data Roll Up 
1. Minutes level data rolled up to hours level data
2. Older data dont' have to be in DB (hot storage, frequently used data that must be accessed fast)
   * Object Storage, AWS S3  (cold storage, archived and infrequently accessed data)
### Data Query Service
1. Data Federation
   * Retrieve data from several data storage and combine the data together
   * This is an ideal use case for distributed cache (store query results)
      * This improve query performance
      * Scale query service
2.

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
## Identify and address bottlenecks, given the constraints. 
1. For example, do you need the following to address scalability issues?
  Load balancer
  Horizontal scaling
  Caching
  Database sharding
  Discuss potential solutions and trade-offs. Everything is a trade-off. Address bottlenecks using principles of scalable system design.

3. How to identify bottlenect? Performance Testing
   * Load Testing:  Measure system behavior under specifc expected load
      * We want to see the system is indeed scalable and can handle expected load, e.g. 2-3 times increases in traffic
   * Stress Testing: Test beyond normal operation capacity, often to a breaking point
      * Which component will suffer first and what resource would it be: memory, CPU, netowrk, disk IO
   * Soak Testing: Test a system with a typical production load for an extended period of time
      * Leaks in system, e.g. memory leak
   * Generate high load is key
      * Tools: Apache JMeter 

4. How do you make sure the system is running healthy? 
   * Need monitor all components
   * metrics: error count, processing time, 
   * dashboard: summary view of service's core metrics
   * alerts
   * Four Golden Signals of Monitor
      * Latency
      * Traffic
      * Errors
      * Saturation

5. How to make sure the system produce the accurate results?
   * Build an audit system
      * Weak
         * A continuously running end-to-end system, 
         * e.g. generate view events, and query to check if the count is the same as expected
      * Strong
         * It can compute the counts using a different path than our main system, 
         * Lambda Architecture
            * The key idea is to send events to a batch system and a stream processing system in parallel
            * Stich together the results from both systems at query time
         * We should use a batch processing framework like MapReduce if we aren't latency sensitive, and use stream processing framework if we are, but not try to do both at the same time unless we absolutely must. 

6. If we have a hugely popular video, will it become a bottleneck? 
   * Spread events coming from a popular video across several partitions
7. What if the processing service can't keep up with the speed new events arrive?
   *  Batch events and store them in the object storage, every time we persist a batch of events, we send a message to message broker, e.g. SQS, then we have a big cluster of machines, e.g. EC2, that retrieve messages from SQS, read a corresponding batch from S3 and process events.
   *  This is slower than stream processing but faster than batch processing.

Back-of-the-envelope calculations
You might be asked to do some estimates by hand. Refer to the Appendix for the following resources:

Use back of the envelope calculations
Powers of two table
Latency numbers every programmer should know

# Cluster Resources
1. DEV
   * 28 Machines: 16 cores/ 94GB RAM/ 2GB Swap/, Total cores: 448, with 2 Data centers
3. PROD
   * 188 Machines: 16 cores/ 94GB RAM/ 2GB Swap/, Total cores:3008, with 2 Data centers

# Product Problems
1. When there are hundreds of or thousands of machines, machine failures are common
2. RPC can be used in MapReduce where a reduce worker reads the local data from a map worker using RPC

# MapReduce
1. Master-Worker
2. Fault tolerance
   * Worker failure: retry on other worker 
   * Master failure:
3. We rely on atomic commits of map and reduce task outputs to achieve this property.
1. If the same reduce task is executed on multiple machines, multiple rename calls will be executed for the same final output file. We rely on the atomic rename operation provided by the underlying file system to guarantee that the final file system state contains just the data produced by one execution of the reduce task.
2. Task Granuity
   * There are practical bounds on how large M and R can be in our implementation, since the master must make O(M + R) scheduling decisions and keeps O(M ∗ R) state in memory as described above.
3. Handle long run tail task
   * 



# References
1. [How to approach a system design interview question](https://github.com/donnemartin/system-design-primer#how-to-approach-a-system-design-interview-question)
2. [System Design Interview – Step By Step Guide](https://youtu.be/bUHFg8CZFws)
