# Overview

1. We shall find useful ways of thinking about data systems—not just **how they work**, but also **why they work that way**, and **what questions we need to ask**.
2. Building for scale that you don’t need is wasted effort and may lock you into an inflexible design.
3. Data-Intensive Applications: Bigger problems are usually the amount of data, the complexity of data, and the speed at which it is changing, instead of compute-intensive.
4. Common Building Blocks of Data Systems
   * Databases: store data
   * Caches: speed up reads
   * Search Indexes: Allow users to search data by keyword or filter it in various ways
   * Stream Processing: Send a message to another process, to be handled asynchronously
   * Batch Processing: Periodically crunch a large amount of accumulated data
5. We try to achieve reliable, scalable, and maintainable data systems.
6. Questions to ask
   * How do you ensure that the data remains correct and complete, even when things go wrong internally?
   * How do you provide consistently good performance to clients, even when parts of your system are degraded?
   * How do you scale to handle an increase in load? 
      * If the system grows in a particular ways, what are our options for coping with the growth? 
      * How can we add computing resources to handle the additional load?
   * What does a good API for the service look like?
7. Why you might want to distribute a database across multiple machines?
   * Scalability
   * Fault Tolerance/high availability
      * Two common ways data is distributed across multiple nodes:
         * Replication: provides redundancy and help improve performance
         * Partitioning: 
   * Latency


# Key Characteristics of Distributed Systems
## 1. Reliability

### Definition 
The system should continue to work *correctly* (performing the correct function at the desired level of performance) even in the face of *adversity* (hardware & software faults, human errors). 

1. **Fault-Tolerant or Resilient Systems**: The systems anticipate (certain types of) faults and can cope with them.
1. Fault is not the same as failure
   * Fault is defined as one component of the system deviating from its spec
   * Failure is when the system as a whole stops providing the required service to the user.
   * It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures.
1. Use Netflix *Chaos Monkey* to introduce/increase the rate of faults and test the fault-toerance machinery.
1. We generally prefer tolerating faults over preventing faults.
1. We want to build reliable systems from unreliable parts.
   * In simple terms, a distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.
   * Example of AMAZON: A purchase shouldn't be cancelled due to the failure of the machine which runs the transaction.
1. We sometimes balance reliability with development cost, operational cost.

### Faults
#### Hardware Faults
1. We usually think of hardware faults as being random and independent from each other, i.e. weakly correlated. It is unlikely that a large number of hardware components will fail at the same time
1. Types of Hardware Faults
   * Hard Disks Crash
      * Hard disks *mean time to failure (MTTF)*: 10-50 years.
      * On a storage with 10,000 disks, 1 disk dies everyday on average.
   * RAM Faults
   * Power Outage
   * Network Outage
1. Solutions
   * Add redundancy to the individual hardware components to reduce the failure rate of the system. 
      * Use RAID for disks
      * Use dual power supplies for servers
      * Use hot-swappable CPUs
      * Use batteries and diesel generators for backup power
      * Until recently, It's sufficient for most applications since it makes total failure of a single machine fairly rare.
   * Multi-machine redundancy (for high availability): Due to large data volumes, increased computing demands, more applications use larger number of machines, which increases the rate of hardware faults. There's a move toward systems that can *tolerate the loss of entire machines*, by using software fault-tolerance techniques in preference or in addition to hardware redundancy.
      * Rolling Upgrade/Staged Rollout: deploying the new version to a few nodes at a time, checking results, and gradually working through all the nodes.

#### Software Faults
1. These are systematic errors in the system.Harder to predict, correlated across nodes. They tend to cause many more system failures than uncorrelated hardware faults.
2. Examples
   * Bug
   * Process uses up shared resources: CPU, memory, disk space, network bandwidth
   * A service that the system depends on that slows down, unresponsive, returns corrupted responses
   * Cascading failures
4. Solutions
   * Carefully thinking about assumptions and interactions in the system
   * Thorough testing
   * Process isolation
   * Allowing processes to crash and restart
   * Measuring, monitoring, and analyzing system behavior in production.

#### Human Error
1. Solutions
   * Design systems in a way that minimizes opportunities for error. Well-designed abstractions, APIs, admin interfaces make it easy to do “the right thing” and discourage “the wrong thing.”
   * Decouple the places where people make the most mistakes from the places where they can cause failures.
      * Provide fully featured non-production *sandbox* environments
   * Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests
   * Allow quick and easy recovery from human errors, to minimize the impact in the case of a failure.
   * Set up detailed and clear monitoring, such as performance metrics and error rates.
   * Implement good management practices and training


## 2. Scalablility
### Definition
As the system *grows* (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.

1. A system can become unreliable if the load increases a lot.
1. If the system grows in a particular ways, what are our options for coping with the growth? 
2. How can we add computing resources to handle the additional load?

* Shared-Memory Architecture: scale up
* Shared-Disk Architecture: users several machines with independent CPUs and RAM, but stores data on an array of disks connected via a fast network (NAS or SAN). Contention and the overhead of locking limit the scalability of this approach.
* Shared-Nothing Architecture (scaling out): 


### Describing Load
1. We use **load parameters** to describe load
   * Requests per Second to a web server
   * The ratios of reads to writes in DB
   * The number of simultaneously active users in a chat room
   * The hit rate on a cache
   * The average cases or extreme cases
#### Example
1. Twitter
   * Write requests (Post Tweet): 4.6K reqs/sec on average, 12K reqs/sec on peak
   * Read requests (Home timeline): 300K reqs/sec
1. Twitter's scaling challenge is not primarily due to tweet volume, but due to fan-out
   * Method 1: When a user requests their home timeline, look up all the people they follow, find all the tweets for each of those users, and merge them (sorted by time)
   * Method 2: Maintain a cache for each user’s home timeline—like a mailbox of tweets for each recipient user.
      * Downside: For popular user, the fan-out load is huge, 30M writes to home timelines. How to achieve this within 5 seconds ?
      * So the distribution of followers per user is a key load parameter
      * Hybrid of 1&2 to solve this.

### Describing Performance
1. Look in two ways
   * When increase a load parameter and keep the system resources unchanged, how is the performance of your system affected?
   * When increase a load parameter, how much do you need to increase the resources if you want to keep the performance unchanged? 

#### Throughput
1. The number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.
1. In an ideal world, the running time of a batch job is the size of the dataset divided by the throughput. In practice, the running time is often longer, due to skew (data not being spread evenly across worker processes) and needing to wait for the slowest task to complete.
2. Important for Batch processing system

#### Response Time
1. The time between a client sending a request and receiving a response
1. Important for stream processing system
1. Latency vs. Response Time
   * The response time is what the client sees, besides the actual time to process the request (service time), it includes network delays and queueing delays.
   * Latency is the duration that a request is waiting to be handled- during which it is latent, awaiting service.
1. We often think about the distribution of response time
   * Random additional latency could be introduced by a context switch to a backgroud process, loss of network packet and TCP retransmission, garbage collection pause, page fault forcing a read from disk, mechanical vibration in server rack
   * Percentiles are better than average: p50 (median), p95 (95th percentile), p99, p999
   * Tail latencies (high percentiles of response times) are important because they directly affect users’ experience of the service.
   * Percentiles are often used in SLO (service level objectives)  and SLA
   * Tail Latency Amplification: High percentiles become especially important in backend services that are called multiple times as part of serving a single end-user request. Even if you make the calls in parallel, the end-user request still needs to wait for the slowest of the parallel calls to complete.  Even if only a small percentage of backend calls are slow, the chance of getting a slow call increases if an end-user request requires multiple back‐ end calls, and so a higher proportion of end-user requests end up being slow.
1. Head-of-Line Blocking
   * Queueing delays often account for a large part of the response time at high percentiles. As a server can only process a small number of things in parallel, it only takes a small number of slow requests to hold up the processing of subsequent requests.
   * Due to this effect, it is important to measure response times on the client side.
#### Coping with Load
1. (scaling out) Distributing load across multiple machines is also known as a shared-nothing architecture. 
1. Elastic systems: they can automatically add computing resources when they detect a load increase, whereas other systems are scaled manually. 
1. While distributing stateless services across multiple machines is fairly straightforward, taking stateful data systems from a single node to a distributed setup can intro‐ duce a lot of additional complexity.
   * For this reason, common wisdom until recently was to keep your database on a single node (scale up) until scaling cost or high- availability requirements forced you to make it distributed.

## 3. Maintainability
### Definition
Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it *productively*. 

1. It is well known that the majority of the cost of software is not in its initial develop‐ ment, but in its ongoing maintenance

### Three Design Principles for Software Systems
#### Operability: Making Life Easy for Operations
1. Mkae it easy for operations teams to keep the system running smoothly, such as
   * Monitoring the health, quickly restoring service
   * Tracking down the cause of problems
   * Keeping software and platforms up to date, including security patches
   * Keeping tabs on how different systems affect each other
   * Anticipating future problems, e.g. capacity planning
   * Establishing good practices and tools for deployment, config etc.
   * Performing complex maintenance tasks
   * Maintaining the security
1. Data Systems can do following:
   * Provide visibility into the runtime behavior and internals of the systetm, with good monitoring
   * Provide good support for automation and integration with standard tools
   * Avoid dependency on individual machines (Allow machines to be taken down while the system continus running uninterrupted)
   * Provid good documentation and easy-to-understand operational model
   * Provide good default behavior, but give freedom to overwrite defaults
   * Self-healing where appropriate but give manual control over the system state
   * Exhibiting predictable behavior, minimizing surprises


#### Simplicity: Managing Complexity
1. Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. 
1. Complexities can be:
   * Explosion of state space
   * Tight coupling of modules
   * Tangled dependencies
   * Inconsistent naming and terminology
   * Hacks to solve performance problems
   * Special-casing to work around
1. Solutions
   * Abstraction: One of the best tools we have for removing accidental complexity is abstraction
   * we will keep our eyes open for good abstractions that allow us to extract parts of a large system into well-defined, reusable components.

#### Evolvability/Extensibility/Modifiability/Plasticity: Making Change Easy
1. Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change.
1. Agile working patterns provide a framework for adapting to change. 
   * TDD and refactoring
   * 

## 4. Availability
## 5. Efficiency


# Offline Processing
## Message Queues
1. For processing you’d like to perform inline with a request but is too slow, the easiest solution is to create a message queue (for example, RabbitMQ). Message queues allow your web applications to quickly publish messages to the queue, and have other consumers processes perform the processing outside the scope and timeline of the client request.

1. Dividing work between off-line work handled by a consumer and in-line work done by the web application depends entirely on the interface you are exposing to your users. Generally you’ll either:
   * perform almost no work in the consumer (merely scheduling a task) and inform your user that the task will occur offline, usually with a polling mechanism to update the interface once the task is complete (for example, provisioning a new VM on Slicehost follows this pattern), or
   * perform enough work in-line to make it appear to the user that the task has completed, and tie up hanging ends afterwards (posting a message on Twitter or Facebook likely follow this pattern by updating the tweet/message in your timeline but updating your followers' timelines out of band; it’s simple isn’t feasible to update all the followers for a Scobleizer in real-time).
   * Message queues have another benefit, which is that they allow you to create a separate machine pool for performing off-line processing rather than burdening your web application servers. This allows you to target increases in resources to your current performance or throughput bottleneck rather than uniformly increasing resources across the bottleneck and non-bottleneck systems.

## Scheduling periodic tasks
1. Almost all large systems require daily or hourly tasks, but unfortunately this seems to still be a problem waiting for a widely accepted solution which easily supports redundancy.
2. In the meantime you’re probably still stuck with cron, but you could use the cronjobs to publish messages to a consumer, which would mean that the cron machine is only responsible for scheduling rather than needing to perform all the processing.

## Map-Reduce
1. Adding a map-reduce layer makes it possible to perform data and/or processing intensive operations in a reasonable amount of time. You might use it for calculating suggested users in a social graph, or for generating analytics reports.
2. For sufficiently small systems you can often get away with adhoc queries on a SQL database, but that approach may not scale up trivially once the quantity of data stored or write-load requires sharding your database, and will usually require dedicated slaves for the purpose of performing these queries (at which point, maybe you’d rather use a system designed for analyzing large quantities of data, rather than fighting your database).

# MSA (Micro-Service Architecture)
1. Its hard to keep track of the service URL’s
   * When there are too many services maintaining the configuration for their individual addresses becomes harrowing, especially since addresses change with environments, for misc infra reasons etc. Now you might use a load balancer and a proxy like HAproxy to centralize this, I have done this myself for simplicity, however, a load balancer is just another layer to add its own latency, failure point, the need to effectively manage DNS switches based on geography if you have a multi-region deployment (multiple data centers) etc.
1. Don't make them nano-services
2. So what can you do if you don’t like load balancers? You can use a concept popularly known as “Service Discovery”. In this concept services which start up register themselves with a central authority with some basic parameters such as environment & application name in case multiple environments share the same infrastructure and the applications which wishes to connect with them polls the one service asking a connection against the app name and environment name etc. This software in turn then can also ensure basic load balancing like round robin to load balance in case multiple services registered themselves with the same parameters, effectively enabling direct communication. 
3. The first question that arises is… so we replaced the latency of the load balancer by the service discovery program… fair comment… however generally speaking these softwares generally reside in the same machine as the service so as to inherently scale and be performant, moreover since the data passed between them is fairly rudimentary they use faster protocols and are generally DNS queries.
4. Note that the load balancer technique is also known as “Server side discovery” and what we are talking about here is also know as “Client side discovery”.

### Advantages & Disadvantages
1. Everything in the software world is subjective out of context of the specific use case, consider the advantages and disadvantages before you choose any model, the article seemed skewed towards client side discovery only because it is lesser known and fits in a variety of use cases.

1. Advantages
   * Client side service discovery or just service discovery reduces network hops and increases the overall performance of the system.
   * The above also mitigates the need of a highly available and performant load balancer system, if however you are in the cloud, this is generally cheaply available e.g. AWS ELB.
1. Disadvantages
   * Yet another software to learn (Eureka/Consul .. whatever).
   * This couples the client with the service registry, using something like Zuul (like a proxy) negates this issue, however removes the performance benefit at the same time.
You have to make sure that the registry has client side code in all languages / frameworks that you build your services in.


### People have issues in discovering services, the available API’s & writing & maintaining clients
1. With MSA, one implied complexity that gets introduced in the system is well, a lot of services. If we don’t document these services well and let’s not kid ourselves we won’t, numerous issues arise over time, some of them that I have noticed are:

   * With lack of documentation a team implementing service X which should be using service of team Y has unnecessary overheads of communication, general mistakes and in general bitching as to why team Y doesn’t care enough about their deadlines.
   * No documented contract of the service means a central person/group cannot make sure that all the API’s follow a particular standard, leading to each team or even the same team developing different looking API contracts.
   * Every time a service is being called in another service plumbing code needs to be written to integrate with the service, including models, connectors etc, if you are one of those super organized teams which churn out clients after every time a service is built for every language that other teams uses, then rule this statement out.
   * Every time a contract changes slightly every team needs to be informed about the new functionality and the plumbing code needs to be updated.
   * Every time an automation QA guy needs to understand the 20 services that you have he will need to sit with the relevant team to understand or force them to make a document for him, this will repeat with changes to the contract.
   * For those services which are public in nature, the documentation’s test harness or try it functionality (which you should have) is always lagging from the actual service contract.

### Too many deployment jobs
This is an inherent issue of having multiple services, I wish I could tell you that some software will solve this, unfortunately though, you can’t escape this one.

### One service connection fails, everything fails
With the spider web of all the services to work with, chances of failures increases, and handling them becomes more complex, well, more complex than calling a method in a class anyway. Fortunately, MSA is not a new thing and there are well defined and tried and tested methodologies to work with the complexities. Some of things that you must have while connecting to other services:

1. Retries

## Technologies
1. NoSQL
2. Message Queues
3. Caches
4. Search Indexes
5. Frameworks for batch and stream processing
6. A/B Test

# Products
## Databases

### Redis
1. They are datastores that can be used as message queues

## Streaming
## Apache Kafka
1. They are message queues with database-like durability guarantees
## Cache
## Memcached

## Search Index
## Elasticsearch
## Solr

## Batch Processing Systems
## Hadoop


# Related Issues in Practice
1. NFS disk usage to the limit
2. NFS number of innodes become  very large, how do we provide reliability? 
3. Why we choose NFS over HDFS?
4. How does the skewness of our data affect our throughput?
5. Scaling out vs. Scaling up decision

# Resources
1. [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://youtu.be/-W9F__D3oY4)
2. Check Bronson's TAO pdf paper in same link as video
3. Check Nishtala's pdf paper in same link as video
4. Lamport's ordering paper: https://lamport.azurewebsites.net/pubs/time-clocks.pdf
5. Paxos (simpler version, not the greek parliament metaphor version): https://www.microsoft.com/en-us/research/publication/paxos-made-simple/
6. Raft: http://thesecretlivesofdata.com/raft/ and also http://kanaka.github.io/raft.js/
7. Dynamo: https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html
8. Designing Data-Intensive Applications (DDIA), The Big Ideas Behind Reliable, Scalable, and Maintainable Systems, Martin Kleppmann, 2017.
9. Grokking the System Design Interview - https://www.educative.io/courses/grokking-the-system-design-interview
10. Nathan Bronson's Tao: https://www.usenix.org/conference/atc13/technical-sessions/presentation/bronson
11. Nishtala's memcache: https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala
12. Kulkarni's Facebook live: https://www.youtube.com/watch?v=IO4teCbHvZw
13. Aroskar's Zuul push: https://www.youtube.com/watch?v=6w6E_B55p0E
14. NALSD on Google SRE book: https://sre.google/workbook/non-abstract-design/
15. Krikorian's timelines at scale: https://www.infoq.com/presentations/Twitter-Timeline-Scalability/
16. Cassandra's capacity planning guidelines by datastax: https://docs.datastax.com/en/dse-planning/doc/planning/capacityPlanning.html
17. Youtube guy: https://www.youtube.com/channel/UC9vLsnF6QPYuH51njmIooCQ
