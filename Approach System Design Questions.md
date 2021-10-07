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
   * It tells me that I should be choosing between Availability and consistency, we choose availability.


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
      * 
10. 






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
