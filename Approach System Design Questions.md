# General Advices:
1. Interviewee should drive the conversation and move forward
2. We should always know about the trade offs of our options. 
3. [You must change your mentality to build really scalable systems](http://highscalability.com/amazon-architecture). Approach chaos in a probabilistic sense that things will work well. In traditional systems we present a perfect world where nothing goes down and then we build complex algorithms (agreement technologies) on this perfect world. Instead, take it for granted stuff fails, that's reality, embrace it. For example, go more with a fast reboot and fast recover approach. With a decent spread of data and services you might get close to 100%. Create self-healing, self-organizing lights out operations.
4. Create a shared nothing infrastructure. Infrastructure can become a shared resource for development and deployment with the same downsides as shared resources in your logic and data tiers. It can cause locking and blocking and dead lock. A service oriented architecture allows the creation of a parallel and isolated development process that scales feature development to match your growth.
5. Open up you system with APIs and you'll create an ecosystem around your application.
6. Only way to manage as large distributed system is to keep things as simple as possible. Keep things simple by making sure there are no hidden requirements and hidden dependencies in the design. Cut technology to the minimum you need to solve the problem you have. It doesn't help the company to create artificial and unneeded layers of complexity.
7. Organizing around services gives agility. You can do things in parallel is because the output is a service. This allows fast time to market. Create an infrastructure that allows services to be built very fast.
8. There's bound to be problems with anything that produces hype before real implementation
9. Use SLAs internally to manage services.
10. Anyone can very quickly add web services to their product. Just implement one part of your product as a service and start using it.
11. Build your own infrastructure for performance, reliability, and cost control reasons. By building it yourself you never have to say you went down because it was company X's fault. Your software may not be more reliable than others, but you can fix, debug, and deployment much quicker than when working with a 3rd party.
12. Use measurement and objective debate to separate the good from the bad. I've been to several presentations by ex-Amazoners and this is the aspect of Amazon that strikes me as uniquely different and interesting from other companies. Their deep seated ethic is to expose real customers to a choice and see which one works best and to make decisions based on those tests.
13. Avinash Kaushik calls this getting rid of the influence of the HiPPO's, the highest paid people in the room. This is done with techniques like A/B testing and Web Analytics. If you have a question about what you should do code it up, let people use it, and see which alternative gives you the results you want.
14. People's side projects, the one's they follow because they are interested, are often ones where you get the most value and innovation. Never underestimate the power of wandering where you are most interested.
15. Create a staging site where you can run thorough tests before releasing into the wild.
16. A robust, clustered, replicated, distributed file system is perfect for read-only data used by the web servers.
17. Have a way to rollback if an update doesn't work. Write the tools if necessary.
18. Switch to a deep services-based architecture (http://webservices.sys-con.com/read/262024.htm).
19. Look for three things in interviews: enthusiasm, creativity, competence. The single biggest predictor of success at Amazon.com was enthusiasm.
20. Hire a Bob. Someone who knows their stuff, has incredible debugging skills and system knowledge, and most importantly, has the stones to tackle the worst high pressure problems imaginable by just leaping in.
21. Innovation can only come from the bottom. Those closest to the problem are in the best position to solve it. any organization that depends on innovation must embrace chaos. Loyalty and obedience are not your tools.
22. Creativity must flow from everywhere.
23. Everyone must be able to experiment, learn, and iterate. Position, obedience, and tradition should hold no power. For innovation to flourish, measurement must rule.
24. Embrace innovation. In front of the whole company, Jeff Bezos would give an old Nike shoe as "Just do it" award to those who innovated.
25. Don't pay for performance. Give good perks and high pay, but keep it flat. Recognize exceptional work in other ways. Merit pay sounds good but is almost impossible to do fairly in large organizations. Use non-monetary awards, like an old shoe. It's a way of saying thank you, somebody cared.
26. Get big fast. The big guys like Barnes and Nobel are on your tail. Amazon wasn't even the first, second, or even third book store on the web, but their vision and drive won out in the end.
27. In the data center, only 30 percent of the staff time spent on infrastructure issues related to value creation, with the remaining 70 percent devoted to dealing with the "heavy lifting" of hardware procurement, software management, load balancing, maintenance, scalability challenges and so on.
28. Prohibit direct database access by clients. This means you can make you service scale and be more reliable without involving your clients. This is much like Google's ability to independently distribute improvements in their stack to the benefit of all applications.
29. Create a single unified service-access mechanism. This allows for the easy aggregation of services, decentralized request routing, distributed request tracking, and other advanced infrastructure techniques.
30. Making Amazon.com available through a Web services interface to any developer in the world free of charge has also been a major success because it has driven so much innovation that they couldn't have thought of or built on their own.
31. Developers themselves know best which tools make them most productive and which tools are right for the job.
32. Don't impose too many constraints on engineers. Provide incentives for some things, such as integration with the monitoring system and other infrastructure tools. But for the rest, allow teams to function as independently as possible.
33. Developers are like artists; they produce their best work if they have the freedom to do so, but they need good tools. Have many support tools that are of a self-help nature. Support an environment around the service development that never gets in the way of the development itself.
34. You build it, you run it. This brings developers into contact with the day-to-day operation of their software. It also brings them into day-to-day contact with the customer. This customer feedback loop is essential for improving the quality of the service.
35. Developers should spend some time with customer service every two years. Their they'll actually listen to customer service calls, answer customer service e-mails, and really understand the impact of the kinds of things they do as technologists.
36. Use a "voice of the customer," which is a realistic story from a customer about some specific part of your site's experience. This helps managers and engineers connect with the fact that we build these technologies for real people. Customer service statistics are an early indicator if you are doing something wrong, or what the real pain points are for your customers.
37. Infrastructure for Amazon, like for Google, is a huge competitive advantage. They can build very complex applications out of primitive services that are by themselves relatively simple. They can scale their operation independently, maintain unparalleled system availability, and introduce new services quickly without the need for massive reconfiguration.
38. [“Be brief, be bright, be gone”](http://highscalability.com/blog/2012/7/16/cinchcast-architecture-producing-1500-hours-of-audio-every-d.html): Respect another person’s time. Don’t come with problems, come with solutions.
40. Don’t go chasing hot technologies of the day. Instead ‘mitigate your top problems’.   We adopt new technologies but do so, when the business case requires it. Appetite for Production outages decreases significantly when you have millions of users.
41. Achieve “essential”, then worry about “excellent”.
42. Be a “how team” instead of a “no team”.
43. Build security into the software development lifecycle.  You need to train developers on how to write secure software and make it a business priority from the start.
44. Not using metrics in your development process is like trying to land a plane in a storm with your altimeter not working. Throughout your development process, compute metrics such as site throughput, time to fix Blocker/Critical bugs, code coverage and use them to gauge your performance.








# 4S Analysis System
1. Scenario
2. Service
3. Storage
4. Scale

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
   * How many daily active users (DAU) are there?
   * How many read requests per second (QPS) do we expect?
      *  QPS = 100: personal desktop
      *  QPS = 1K: A good web server is fine (logic, DB access etc.), consider single point of failure
      *  QPS = 1m: Need a cluster of 1000 servers, maintenance
      *  A single web server may take up to 1K QPS (consider cache)
      *  A SQL Database may take up to 1K QPS (less if there are more JOIN and INDEX query)
      *  A NoSQL Database (Cassandra) may take up to 10K QPS
      *  A NoSQL Database (Memcached) may take up to 1M QPS
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
1. Start with a Database to store the data for each service
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
      * Both SQL and NoSQL (Tweets, social graph etc.) databases can scale and perform well
      * Evaluate database against the non-functional requirements we wrote down before
      * File System (medias)
         * 数据库在读取上不一定比文件系统更高效：如果只是简单数据，通过文件系统读取比数据库要高效。数据库每一步操作都需要log、备份、事务提交等等，比修改文件麻烦多了
      * Cache
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
   * Object Storage, AWS S3  (A cloud Distributed File System, archived and infrequently accessed data)
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
## Step 1: Optimize
1. Solve Problems
   * Pull vs Push
   * Memcached QPS vs. MySQL: about 100~1000 times faster
1. More Features
   * Like, Follow & Unfollow, Ads
1. Special Cases

## Step 2: Maintenance
1. Robust
   * Fail of server, DB
1. Scalability
   * How to expand the system and handle jump of QPS
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
1.  it hides the details of parallelization, fault-tolerance, locality optimization, and load balancing
1. Master-Worker
2. Fault tolerance
   * Worker failure: retry on other worker 
   * Master failure:
3. We rely on atomic commits of map and reduce task outputs to achieve this property.
1. If the same reduce task is executed on multiple machines, multiple rename calls will be executed for the same final output file. We rely on the atomic rename operation provided by the underlying file system to guarantee that the final file system state contains just the data produced by one execution of the reduce task.
2. Task Granuity
   * There are practical bounds on how large M and R can be in our implementation, since the master must make O(M + R) scheduling decisions and keeps O(M ∗ R) state in memory as described above.
3. Handle long run tail task
   * When a MapReduce operation is close to completion, the master schedules backup executions of the remaining in-progress tasks. The task is marked as completed whenever either the primary or the backup execution completes.
1. Network bandwidth is a scarce resource. A number of optimizations in our system are therefore targeted at reducing the amount of data sent across the network: the locality optimization allows us to read data from local disks, and writing a single copy of the intermediate data to local disk saves network bandwidth.
   * The MapReduce master takes the location information of the input files into account and attempts to schedule a map task on a machine that contains a replica of the corresponding input data. Failing that, it attempts to schedule a map task near a replica of that task’s input data (e.g., on a worker machine that is on the same network switch as the machine containing the data)
   * A network switch is a device that operates at the Data Link layer of the OSI model—Layer 2. It takes in packets being sent by devices that are connected to its physical ports and sends them out again, but only through the ports that lead to the devices the packets are intended to reach. They can also operate at the network layer--Layer 3 where routing occurs. witches are a common component of networks based on ethernet, Fibre Channel, Asynchronous Transfer Mode (ATM), and InfiniBand, among others. In general, though, most switches today use ethernet.
   * Once a device is connected to a switch, the switch notes its media access control (MAC) address, a code that’s baked into the device’s network-interface card (NIC) that attaches to an ethernet cable that attaches to the switch. The switch uses the MAC address to identify which attached device outgoing packets are being sent from and where to deliver incoming packets.So the MAC address identifies the physical device as opposed to the network layer (Layer 3) IP address, which can be assigned dynamically to a device and change over time. 
   * When a device sends a packet to another device, it enters the switch and the switch reads its header to determine what to do with it. It matches the destination address or addresses and sends the packet out through the appropriate ports that leads to the destination devices.

# Pull Model
1. 




# References
1. [How to approach a system design interview question](https://github.com/donnemartin/system-design-primer#how-to-approach-a-system-design-interview-question)
2. [System Design Interview – Step By Step Guide](https://youtu.be/bUHFg8CZFws)
3. [Amazon Architecture](http://highscalability.com/amazon-architecture)
