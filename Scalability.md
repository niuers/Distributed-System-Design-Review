# Scalability

1. A service is said to be scalable if when we increase the resources in a system, it results in increased performance in a manner proportional to resources added. Generally, increasing performance means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.

1. In distributed systems there are other reasons for adding resources to a system
   * For example to improve the reliability of the offered service. Introducing redundancy is an important first line of defense against failures. 
   * An always-on service is said to be scalable if adding resources to facilitate redundancy does not result in a loss of performance.
1. For the systems we build we must carefully inspect along which axis we expect the system to grow, where redundancy is required, and how one should handle heterogeneity in this system


## Types of Scaling
### Vertical Scaling
1. Increase the power of CPU, storage of Disk (PATA, SATA-7200rmp,SAS-15000rpm, SSD), capacity of Memory
2. Constrained by real-world limits

### Horizontal Scaling
1. Increase the number of servers, each may still has a unique IP address
2. HTTP requests now need to be distributed across the servers through a load balancer
3. DNS instead of returning the IP address of the server 1 or 2 etc., now returns the IP address of the 'load balancer'. So the load balancer now has the public IP address.
   * The back-end servers now have private IP addresses, which can't be seen by public 
   * Load balancer can now send requests to the servers using TCP/IP, 
4. It's not a good idea to put DB server on the same web server, as a user can connect to another server, which doesn't have his/her information there.

## Rules for Scalibility
1. The first golden rule for scalability: Every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory. 
   * A user expects to get the same results back regardless of which server he "lands on" 

## Scalibility Patterns: State
### Partitioning
### HTTP Caching
### RDBMS Sharding
### NOSQL
### Distributed Caching
### Data Grids/Clustering
1. Parallel Data Storage
   * Data Replication
   * Data partition
   * Continous availability
   * Data Invalidation
   * Fail-over
   * C+P in CAP
2. Products
   * Coherence,Terracotta etc.
### Concurrency
#### Shared-State Concurrency
1. Everyone can access anything anytime
2. Totally indeterministic
3. Introduce determinism at well defined locations using locks
   * Locks do not compose
   * We may take too few/too many/wrong locks
   * Taking locks in the wrong order
   * Error recovery is hard

#### Message-Passing Concurrency
1. Actors
   * Encapsulates state and behavior, closer to the definition of OO than class
   * Share NOTHING
   * Isolate lightweight processes
   * Communicate through messages
   * Asynchronous and non-blocking: no shared state so nothing to synchronize
   * Each actor has a mailbox (message queue)
2. Advantages
   * Easier to reason about
   * Raised abstraction level
   * Easier to avoid: race conditions, dead locks, starvation, live locks
#### Dataflow Concurrency
1. Declarative
2. No observable non-determinism
3. Data driven: thread blocks until data is avaiable
4. On deman, lazy
5. No difference between concurrent and sequential code
6. Limitations: can't have side effects

#### Software Transactional Memory
1. See the memory (heap and stack) as a transactional dataset
2. Similar to a database: begin, commit, roll back/abort
3. Transactions are retried automatically on collision
4. Rolls back the memory on abort
5. Transactions can nest/compose
6. All operations in the scope of transactions need to be idempotent

## Scalibility Patterns: Behavior
### Event-Driven Architecture
1. Domain Events
   * It represents the state at given time when an important event occurred and decouple subsystems with event stream, 
3. Event Sourcing
4. Command and Query Responsibility Segration (CQRS) Pattern
   * All state changes are represented by domain events
   * Aggregate roots receive commands and publish events
   * Reporting (query database) is updated as a result of the published events
   * All queries from presentations go directly to Reporting and the domain is not involved
   * Advantages
      * Fully encapsulated domain that only exposes behavior
      * Queries do not use domain model
      * 
6. Event Stream Processing
7. Messaging
   * Publish-Subscribe
   * Point-to-Point
   * Store forward
   * Request Replay
   * Products
      * RabbitMQ (AMQP)
9. Enterprise Service Bus
10. Actors
11. Enterprise Integration Architecture (EIA): Enterprise Integration Patterns Book
### Compute Grids
1. Divide and Conquer
   * Split up job in independent tasks
   * Execute tasks in parallel
   * Aggregate and return results
3. MapReduce - Master/Worker
1. Features
   * Automatical provisioning
   * Load balancing
   * Fail over
   * Topology resolution
1. Products
   * Google MapReduce
   * Hadoop
### Load Balancer
### Parallel Computing
#### Patterns
1. SPMD Pattern (Single Program Multiple Data)
   * Use the UE's ID to select different pathways through the program,e.g. branching on ID, use ID in loop index to split loop
   * Keep interactions between UE explict
3. Master/Worker Pattern
   * Good scalability
   * Automatic load-balancing
   * How to detect termination?
      *  Bag of tasks is empty
      *  Posion pill
   *  What if we bottleneck on single queue?
      *  Use multiple work queue
      *  Work stealing
   *  What about fault tolerance?
      *  Use in-progress queue
5. Loop Parallelism Pattern
   *  
7. Fork/Join Pattern
8. MapReduce Pattern
   * Hadoop, AWS MapReduce
   * Many NoSQL DB uses for query/search
#### UE: Unit of Execution
1. Process
2. Thread
3. Coroutine
4. Actor




## Replication
## High Availability
## Security
1. Types of traffic allowed to come in
   * TCP:80
   * TCP:443: default port for SSL for HTTP based urls
   * TCP:22, for SSH or SSL based VPN to connect to the DC remotely
3. Types of traffic from Load Balancer to web servers (TCP:80)
   * It's common to offload your SSL to the load balancer or some special device to keep everything else unencrypted as you control it.
   * So everything from LB down stream can go as HTTP unencrypted, so you don't need put your SSL certificates on all the web servers. You can just put them on the load balancers.
4. Traffic from web servers to databases
   * TCP:3306, default MySQL port

# Performance vs Scalability

1. If you have a performance problem, your system is slow for a single user.
1. If you have a scalability problem, your system is fast for a single user but slow under heavy load.

# Latency vs. Throughput

1. Latency is the time to perform some action or to produce some result. Latency is measured in units of time -- hours, minutes, seconds, nanoseconds or clock periods.

1. Throughput is the number of such actions or results per unit of time. This is measured in units of whatever is being produced (cars, I/O samples, memory words, iterations) per unit of time.

1. You should strive for maximum throughput with acceptable latency


# Availability vs. Consistency

## Brewer's CAP Theorem
1. You can only pick two out of three guarantees across a write/read pair at any given time in a distributed system.
   * Consistency: Every read receives the most recent write or an error.
   * Availability: Every request receives a response, without guarantee that it contains the most recent version of the information.
   * Partition Tolerance: The system continues to operate despite arbitrary partitioning due to network failures.
      * Given that networks aren't completely reliable, you must tolerate partitions in a distributed system.


### Centralized System
1. In RDBMS, we don't have the 'P', i.e. network partitions
2. ACID
   * Atomic
   * Consistent
   * Isolated
   * Durable

### Distributed System
1. We'll always have 'P', so we can only pick one from Availability and Consistency
2. There are only two types of systems
   * CP: Wait for a response from the partitioned node which could result in a timeout error. The system can also choose to return an error, depending on the scenario you desire. 
      * Choose Consistency over Availability when your business requirements dictate atomic reads and writes.
   * AP: Return the most recent version of the data you have, which could be stale. This system state will also accept writes that can be processed later when the partition is resolved. Writes might take some time to propagate when the partition is resolved. 
      * Choose Availability over Consistency when your business requirements allow for some flexibility around when the data in the system synchronizes (eventual consistency).
      * Availability is also a compelling option when the system needs to continue to function in spite of external errors (shopping carts, etc.)
3. BASE (Basically Available, Soft State, Eventually Consistent)


# Resources:
1. [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://youtu.be/-W9F__D3oY4)
1. [Scalability for Dummies](https://www.lecloud.net/tagged/scalability)
1. [A Word on Scalability](https://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html)
