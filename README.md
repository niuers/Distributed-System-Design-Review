1. We shall find useful ways of thinking about data systems—not just **how they work**, but also **why they work that way**, and **what questions we need to ask**.
2. Building for scale that you don’t need is wasted effort and may lock you into an inflexible design.

# Key Characteristics of Distributed Systems
## 1. Reliability
### Definition 

The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software **faults**, and even human error). The probability a system will fail in a given period.

1. In simple terms, a distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.
1. **Fault-Tolerant or Resilient Systems**: The systems anticipate (certain types of ) faults and can cope with them.
1. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. 
2. We generally prefer tolerating faults over preventing faults.

### Examples
1. AMAZON: A purchase shouldn't be cancelled due to the failure of the machine which runs the transaction.

### Faults

#### Software Faults
1. These are systematic errors in the system.
2. Solutions
   * Carefully thinking about assumptions and interactions in the system
   * Thorough testing
   * Process isolation
   * Allowing processes to crash and restart
   * Measuring, monitoring, and analyzing system behavior in production.
#### Hardware Faults
1. We usually think of hardware faults as being random and independent from each other, i.e. weakly correlated.
1. Hard Disks Crash
   * Hard disks *mean time to failure (MTTF)*: 10-50 years.
   * On a storage with 10,000 disks, 1 disk dies per day on average.
3. RAM Faults
4. Power Outage
5. Network Outage
6. Solutions
   * Add redundancy to the individual hardware components to reduce the failure rate of the system. 
   * Multi-machine redundancy (for high availability): Due to large data volumes, more applications begin to use larger number of machines, there's a move toward systems that can *tolerate the loss of entire machines*, by using software fault-tolerance techniques in preference or in addition to hardware redundancy.
      * Rolling Upgrade
      * 
#### Human Error
1. Solutions
   * Design systems in a way that minimizes opportunities for error. Make it easy to do “the right thing” and discourage “the wrong thing.”
   * Decouple the places where people make the most mistakes from the places where they can cause failures.
   * Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests
   * Allow quick and easy recovery from human errors, to minimize the impact in the case of a failure.
   * Set up detailed and clear monitoring, such as performance metrics and error rates.
   * Implement good management practices and training
### Techniques to Increase Reliability
1. Build Reliable Systems from Unreliable Parts
2. 


## 2. Scalablility
### Definition
The ability of a system to deal with increased load.

1. A system can become unreliable if the load increases a lot.
1. If the system grows in a particular ways, what are our options for coping with the growth? 
2. How can we add computing resources to handle the additional load?


### Describing Load
1. We use **load parameters** to describe load
   * Requests per Second to a web server
   * The ratios of reads to writes in DB
   * The number of  simultaneously active users in a chat room
   * The hit rate on a cache
   * 
## 3. Maintainability
## 4. Availability
## 5. Efficiency

# References
1. Designing Data-Intensive Applications (DDIA), The Big Ideas Behind Reliable, Scalable, and Maintainable Systems, Martin Kleppmann, 2017.
2. Grokking the System Design Interview

# Twitter
## Main Operations
1. post tweet
   * Scaling challenge not primarily due to tweet volume but due to fan-out. 
3. home timeline

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


# Resources
1. [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://youtu.be/-W9F__D3oY4)
2. Check Bronson's TAO pdf paper in same link as video
3. Check Nishtala's pdf paper in same link as video
4. Lamport's ordering paper: https://lamport.azurewebsites.net/pubs/time-clocks.pdf
5. Paxos (simpler version, not the greek parliament metaphor version): https://www.microsoft.com/en-us/research/publication/paxos-made-simple/
6. Raft: http://thesecretlivesofdata.com/raft/ and also http://kanaka.github.io/raft.js/
7. Dynamo: https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html
8. DDIA - https://dataintensive.net/buy.html
9. Grokking - https://www.educative.io/courses/grokking-the-system-design-interview
10. Nathan Bronson's Tao: https://www.usenix.org/conference/atc13/technical-sessions/presentation/bronson
11. Nishtala's memcache: https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala
12. Kulkarni's Facebook live: https://www.youtube.com/watch?v=IO4teCbHvZw
13. Aroskar's Zuul push: https://www.youtube.com/watch?v=6w6E_B55p0E
14. NALSD on Google SRE book: https://sre.google/workbook/non-abstract-design/
15. Krikorian's timelines at scale: https://www.infoq.com/presentations/Twitter-Timeline-Scalability/
16. Cassandra's capacity planning guidelines by datastax: https://docs.datastax.com/en/dse-planning/doc/planning/capacityPlanning.html
17. Youtube guy: https://www.youtube.com/channel/UC9vLsnF6QPYuH51njmIooCQ
