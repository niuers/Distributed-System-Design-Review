We will try to find useful ways of thinking about data systems—not just **how they work**, but also **why they work that way**, and **what questions we need to ask**.

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
