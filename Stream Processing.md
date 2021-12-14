# Overview
1. The problem with daily batch processes is that changes in the input are only reflected in the output a day later, which is too slow for many impatient users.
2. we will look at event streams as a data management mechanism: the unbounded, incrementally processed counterpart to the batch data we saw in the last chapter
3. stream processing is very much like the batch processing, but done continuously on unbounded (never- ending) streams rather than on a fixed-size input. From this perspective, message brokers and event logs serve as the streaming equivalent of a filesystem
4. 

# Transmitting Event Stream
1. When the input is a file (a sequence of bytes), the first processing step is usually to parse it into a sequence of records. In a stream processing context, a record is more commonly known as an event, but it is essentially the same thing: a small, self- contained, immutable object containing the details of something that happened at some point in time.
2. 
3. in streaming terminology, an event is generated once by a producer (also known as a publisher or sender), and then potentially processed by multiple con‐ sumers (subscribers or recipients
4. In a filesystem, a filename identifies a set of  related records; in a streaming system, related events are usually grouped together into a topic or stream.
5. in principle, a file or database is sufficient to connect producers and consumers: a producer writes every event that it generates to the datastore, and each consumer periodically polls the datastore to check for events that have appeared since it last ran.
6. However, when moving toward continual processing with low delays, polling becomes expensive if the datastore is not designed for this kind of usage. The more often you poll, the lower the percentage of requests that return new events, and thus the higher the overheads become. Instead, it is better for consumers to be notified when new events appear.
7. 
## Messaging Systems
1. A common approach for notifying consumers about new events is to use a messaging system: a producer sends a message containing the event, which is then pushed to consumers.
2. A direct communication channel like a Unix pipe or TCP connection between pro‐ ducer and consumer would be a simple way of implementing a messaging system
3. Within this publish/subscribe model, different systems take a wide range of approaches, To differentiate the systems, it is particularly helpful to ask the following two questions:
   * What happens if the producers send messages faster than the consumers can pro‐ cess them? Broadly speaking, there are three options: 
      * the system can drop messages, 
      * buffer messages in a queue: If messages are buffered in a queue, it is important to understand what happens as that queue grows. Does the system crash if the queue no longer fits in memory, or does it write messages to disk? If so, how does the disk access affect the performance of the messaging system
      * apply backpressure (also known as flow control; i.e., blocking the producer from sending more messages). For example, Unix pipes and TCP use backpressure: they have a small fixed-size buffer, and if it fills up, the sender is blocked until the recipient takes data out of the buffer
      * What happens if nodes crash or temporarily go offline—are any messages lost? As with databases, durability may require some combination of writing to disk and/or replication (see the sidebar “Replication and Durability” on page 227), which has a cost. If you can afford to sometimes lose messages, you can probably get higher throughput and lower latency on the same hardware.

### Direct messaging from producers to consumers
1. UDP multicast is widely used in the financial industry for streams such as stock market feeds, where low latency is important. Although UDP itself is unrelia‐ ble, application-level protocols can recover lost packets (the producer must remember packets it has sent so that it can retransmit them on demand)
2. Brokerless messaging libraries such as ZeroMQ [9] and nanomsg take a similar approach, implementing publish/subscribe messaging over TCP or IP multicast.
3. StatsD [10] and Brubeck [7] use unreliable UDP messaging for collecting metrics from all machines on the network and monitoring them.
4. If the consumer exposes a service on the network, producers can make a direct HTTP or RPC request to push messages to the consumer. This is the idea behind webhooks [12], a pattern in which a callback URL of one service is registered with another service, and it makes a request to that URL whenever an event occurs
1. they generally require the application code to be aware of the possibility of message loss. The faults they can tolerate are quite limited: even if the protocols detect and retransmit packets that are lost in the network, they generally assume that producers and consumers are constantly online
2. If a consumer is offline, it may miss messages that were sent while it is unreachable

### Message Brokers/Message Queue
1. It is essentially a kind of database that is optimized for handling message streams [13]. It runs as a server, with producers and consumers connecting to it as clients. Producers write messages to the broker, and consumers receive them by reading them from the broker
2. By centralizing the data in the broker, these systems can more easily tolerate clients that come and go (connect, disconnect, and crash), and the question of durability is moved to the broker instead
3. Faced with slow consumers, they generally allow unboun‐ ded queueing (as opposed to dropping messages or backpressure), although this choice may also depend on the configuration.
4. A consequence of queueing is also that consumers are generally asynchronous: when a producer sends a message, it normally only waits for the broker to confirm that it has buffered the message and does not wait for the message to be processed by con‐ sumers. The delivery to consumers will happen at some undetermined future point in time—often within a fraction of a second, but sometimes significantly later if there is a queue backlog.
1. The broker assigns individual messages to consumers, and consumers acknowl‐ edge individual messages when they have been successfully processed.Messages are deleted from the broker once they have been acknowledged. This approach is appropriate as an asynchronous form of RPC. for example in a task queue, where the exact order of mes‐ sage processing is not important and where there is no need to go back and read old messages again after they have been processed

#### Message brokers compared to databases
1. Databases usually keep data until it is explicitly deleted, whereas most message brokers automatically delete a message when it has been successfully delivered to its consumers. Such message brokers are not suitable for long-term data storage.
2. Since they quickly delete messages, most message brokers assume that their working set is fairly small—i.e., the queues are short. If the broker needs to buffer a lot of messages because the consumers are slow (perhaps spilling messages to disk if they no longer fit in memory), each individual message takes longer to process, and the overall throughput may degrade [6].
3. Databases often support secondary indexes and various ways of searching for data, while message brokers often support some way of subscribing to a subset of topics matching some pattern. The mechanisms are different, but both are essentially ways for a client to select the portion of the data that it wants to know about.
4. When querying a database, the result is typically based on a point-in-time snap‐ shot of the data; if another client subsequently writes something to the database that changes the query result, the first client does not find out that its prior result is now outdated (unless it repeats the query, or polls for changes). By contrast, message brokers do not support arbitrary queries, but they do notify clients when data changes (i.e., when new messages become available).
5. This is the traditional view of message brokers, which is encapsulated in standards like JMS [14] and AMQP [15] and implemented in software like RabbitMQ, ActiveMQ, HornetQ, Qpid, TIBCO Enterprise Message Service, IBM MQ, Azure Ser‐ vice Bus, and Google Cloud Pub/Sub

### Multiple consumers
1. When multiple consumers read messages in the same topic, two main patterns of messaging are used
   * Load balancing
      * Each message is delivered to one of the consumers, so the consumers can share the work of processing the messages in the topic. The broker may assign messages to consumers arbitrarily. This pattern is useful when the messages are expensive to process, and so you want to be able to add consumers to parallelize the processing. (In AMQP, you can implement load balancing by having multi‐ ple clients consuming from the same queue, and in JMS it is called a shared subscription.)
   * Fan-out
      * Each message is delivered to all of the consumers. Fan-out allows several independent consumers to each “tune in” to the same broadcast of messages, without affecting each other—the streaming equivalent of having several different batch jobs that read the same input file. (This feature is provided by topic subscriptions in JMS, and exchange bindings in AMQP.)
1. The two patterns can be combined: for example, two separate groups of consumers may each subscribe to a topic, such that each group collectively receives all messages, but within each group only one of the nodes receives each message.
### Acknowledgments and redelivery
1. Consumers may crash at any time, so it could happen that a broker delivers a mes‐ sage to a consumer but the consumer never processes it, or only partially processes it before crashing.
2. In order to ensure that the message is not lost, message brokers use acknowledgments: a client must explicitly tell the broker when it has finished process‐ ing a message so that the broker can remove it from the queue
3. If the connection to a client is closed or times out without the broker receiving an acknowledgment, it assumes that the message was not processed, and therefore it delivers the message again to another consumer. (Note that it could happen that the message actually was fully processed, but the acknowledgment was lost in the net‐ work. Handling this case requires an atomic commit protocol
4. When combined with load balancing, this redelivery behavior has an interesting effect on the ordering of messages.
5. Even if the message broker otherwise tries to preserve the order of messages (as required by both the JMS and AMQP standards), the combination of load balancing with redelivery inevitably leads to messages being reordered. To avoid this issue, you can use a separate queue per consumer (i.e., not use the load balancing feature).
## Partitioned Logs
1. Sending a packet over a network or making a request to a network service is normally a transient operation that leaves no permanent trace. Although it is possible to record it permanently (using packet capture and logging), we normally don’t think of it that way. Even message brokers that durably write messages to disk quickly delete them again after they have been delivered to consumers, because they are built around a transient messaging mindset.
2. This difference in mindset has a big impact on how derived data is created.
3. receiving a message is destructive if the acknowledgment causes it to be deleted from the broker, so you cannot run the same consumer again and expect to get the same result
4. If you add a new consumer to a messaging system, it typically only starts receiving messages sent after the time it was registered; any prior messages are already gone and cannot be recovered.
5. Why can we not have a hybrid, combining the durable storage approach of databases with the low-latency notification facilities of messaging? This is the idea behind log- based message brokers
6. Log-based message broker: The broker assigns all messages in a partition to the same consumer node, and always delivers messages in the same order. Parallelism is achieved through partitioning, and consumers track their progress by checkpointing the offset of the last message they have processed. The broker retains messages on disk, so it is possible to jump back and reread old messages if necessary.
7. We saw that this approach is especially appropriate for stream processing systems that consume input streams and generate derived state or derived output streams

### Using logs for message storage
1. A log is simply an append-only sequence of records on disk
2. to implement a message broker:a producer sends a message by appending it to the end of the log, and a consumer receives messages by reading the log sequentially. If a consumer reaches the end of the log, it waits for a notification that a new message has been appended. The Unix tool tail -f, which watches a file for data being appended, essentially works like this.
3. In order to scale to higher throughput than a single disk can offer, the log can be partitioned. A topic can then be defined as a group of parti‐ tions that all carry messages of the same type. This
4. Within each partition, the broker assigns a monotonically increasing sequence num‐ ber, or offset, to every message. Such a sequence number makes sense because a partition is append-only, so the messages within a partition are totally ordered. There is no ordering guarantee across different partitions.
5. Even though these message brokers write all messages to disk, they are able to achieve throughput of millions of messages per second by partitioning across multiple machines, and fault tolerance by replicating messages
6. Apache Kafka [17, 18], Amazon Kinesis Streams [19], and Twitter’s DistributedLog [20, 21] are log-based message brokers that work like this.

### Logs compared to traditional messaging
1. The log-based approach trivially supports fan-out messaging, because several con‐ sumers can independently read the log without affecting each other—reading a mes‐ sage does not delete it from the log. To achieve load balancing across a group of consumers, instead of assigning individual messages to consumer clients, the broker can assign entire partitions to nodes in the consumer group
2. Each client then consumes all the messages in the partitions it has been assigned.
   * The number of nodes sharing the work of consuming a topic can be at most the number of log partitions in that topic, because messages within the same partition are delivered to the same node
   * If a single message is slow to process, it holds up the processing of subsequent messages in that partition (a form of head-of-line blocking)
1. Thus, in situations where messages may be expensive to process and you want to par‐ allelize processing on a message-by-message basis, and where message ordering is not so important, the JMS/AMQP style of message broker is preferable. On
2. in situations with high message throughput, where each message is fast to pro‐ cess and where message ordering is important, the log-based approach works very well.

### Consumer offsets
1. Consuming a partition sequentially makes it easy to tell which messages have been processed: all messages with an offset less than a consumer’s current offset have already been processed, and all messages with a greater offset have not yet been seen.
2. Thus, the broker does not need to track acknowledgments for every single message— it only needs to periodically record the consumer offsets. The reduced bookkeeping overhead and the opportunities for batching and pipelining in this approach help increase the throughput of log-based systems.
3. This offset is in fact very similar to the log sequence number that is commonly found in single-leader database replication. In database replication, the log sequence number allows a follower to reconnect to a leader after it has become disconnected, and resume repli‐ cation without skipping any writes. Exactly the same principle is used here: the mes‐ sage broker behaves like a leader database, and the consumer like a follower
4. If a consumer node fails, another node in the consumer group is assigned the failed consumer’s partitions, and it starts consuming messages at the last recorded offset. If the consumer had processed subsequent messages but not yet recorded their offset, those messages will be processed a second time upon restart.
### Disk space usage
1. To reclaim disk space, the log is actually divided into segments, and from time to time old segments are deleted or moved to archive storage
2. This means that if a slow consumer cannot keep up with the rate of messages, and it falls so far behind that its consumer offset points to a deleted segment, it will miss some of the messages.
3.  Effectively, the log implements a bounded-size buffer that dis‐ cards old messages when it gets full, also known as a circular buffer or ring buffer. However, since that buffer is on disk, it can be quite large
4.  At the time of writing, a typical large hard drive has a capacity of 6 TB and a sequential write throughput of 150 MB/s. If you are writing messages at the fastest possible rate, it takes about 11 hours to fill the drive. In practice, deployments rarely use the full write bandwidth of the disk, so the log can typically keep a buffer of several days’ or even weeks’ worth of messages
5. Regardless of how long you retain messages, the throughput of a log remains more or less constant, since every message is written to disk anyway. This behavior is in contrast to messaging systems that keep messages in memory by default and only write them to disk if the queue grows too large: such systems are fast when queues are short and become much slower when they start writing to disk, so the throughput depends on the amount of history retained.

### When consumers cannot keep up with producers
1. the log-based approach is a form of buffering with a large but fixed-size buffer (limited by the available disk space)
2. If a consumer falls so far behind that the messages it requires are older than what is retained on disk, it will not be able to read those messages—so the broker effectively drops old messages that go back further than the size of the buffer can accommodate.
3. Even if a consumer does fall too far behind and starts missing messages, only that consumer is affected; it does not disrupt the service for other consumers
4. This behavior also contrasts with traditional message brokers, where you need to be careful to delete any queues whose consumers have been shut down—otherwise they continue unnecessarily accumulating messages and taking away memory from con‐ sumers that are still active

### Replaying old messages
1. in a log-based message broker, con‐ suming messages is more like reading from a file: it is a read-only operation that does not change the log
2. The only side effect of processing, besides any output of the consumer, is that the consumer offset moves forward. But the offset is under the consumer’s control, so it can easily be manipulated if necessary

# Databases and Streams
1. a replication log (see “Implementation of Replication Logs” on page 158) is a stream of database write events, produced by the leader as it processes transactions
2. We also came across the state machine replication principle in “Total Order Broad‐ cast” on page 348, which states: if every event represents a write to the database, and every replica processes the same events in the same order, then the replicas will all end up in the same final state. It’s just another case of event streams!
1. we will first look at a problem that arises in heterogeneous data sys‐ tems, and then explore how we can solve it by bringing ideas from event streams to databases.

## Keeping Systems in Sync
1. As the same or related data appears in several different places, they need to be kept in sync with one another: if an item is updated in the database, it also needs to be upda‐ ted in the cache, search indexes, and data warehouse.
2. If periodic full database dumps are too slow, an alternative that is sometimes used is dual writes, in which the application code explicitly writes to each of the systems when data changes
3. dual writes have some serious problems, one of which is a race condition
4. Another problem with dual writes is that one of the writes may fail while the other succeeds. This is a fault-tolerance problem rather than a concurrency problem. Ensur‐ ing that they either both succeed or both fail is a case of the atomic commit problem, which is expensive to solve (2PC)
5. Representing databases as streams opens up powerful opportunities for integrating systems. You can keep derived data systems such as search indexes, caches, and ana‐ lytics systems continually up to date by consuming the log of changes and applying them to the derived system. You can even build fresh views onto existing data by starting from scratch and consuming the log of changes from the beginning all the way to the present.
## Change Data Capture
1. For decades, many databases simply did not have a documented way of getting the log of changes written to them. For this reason it was difficult to take all the changes made in a database and replicate them to a different storage technology such as a search index, cache, or data warehouse.
2. More recently, there has been growing interest in change data capture (CDC), which is the process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems. CDC is especially interest‐ ing if changes are made available as a stream, immediately as they are written.
### Implementing change data capture
1. Essentially, change data capture makes one database the leader (the one from which the changes are captured), and turns the others into followers. A log-based message broker is well suited for transporting the change events from the source database, since it preserves the ordering of messages
2. Parsing the replication log can be a more robust approach, although it also comes with challenges, such as handling schema changes.
3. LinkedIn Databus, FB Wormhole, Yahoo Sherpa
4. Like message brokers, change data capture is usually asynchronous: the system of record database does not wait for the change to be applied to consumers before com‐ mitting it. This design has the operational advantage that adding a slow consumer does not affect the system of record too much, but it has the downside that all the issues of replication lag apply

### Initial Snapshot
1. However, in many cases, keeping all changes forever would require too much disk space, and replaying it would take too long, so the log needs to be truncated. So we need a snapshot
### Log compaction
1. If the CDC system is set up such that every change has a primary key, and every update for a key replaces the previous value for that key, then it’s sufficient to keep just the most recent write for a particular key.
2. This log compaction feature is supported by Apache Kafka. As we shall see later in this chapter, it allows the message broker to be used for durable storage, not just for transient messaging.

### API support for change streams

## Event Sourcing
1. event sourcing, a technique that was developed in the domain-driven design (DDD) community
2. event sourcing involves storing all changes to the application state as a log of change events. The biggest difference is that event sourc‐ ing applies the idea at a different level of abstraction:
   * In change data capture, the application uses the database in a mutable way, updating and deleting records at will. The application writing to the database does not need to be aware that CDC is occurring.
   * In event sourcing, the application logic is explicitly built on the basis of immuta‐ ble events that are written to an event log. In this case, the event store is append- only, and updates or deletes are discouraged or prohibited. Events are designed to reflect things that happened at the application level, rather than low-level state changes.

1. from an application point of view it is more meaningful to record the user’s actions as immutable events, rather than recording the effect of those actions on a mutable database.
2. Event sourcing is similar to the chronicle data model [45], and there are also similari‐ ties between an event log and the fact table that you find in a star schema
### Deriving current state from the event log
1. replaying the event log allows you to reconstruct the current state of the system. However, log compaction needs to be handled differently:
   * an event typically expresses the intent of a user action, not the mechanics of the state update that occurred as a result of the action. In this case, later events typically do not override prior events, and so you need the full history of events to recon‐ struct the final state. Log compaction is not possible in the same way.
### Commands and events
1. The event sourcing philosophy is careful to distinguish between events and com‐ mands
2. When a request from a user first arrives, it is initially a command. If the validation is successful and the command is accepted, it becomes an event, which is durable and immutable.
3. A consumer of the event stream is not allowed to reject an event: by the time the con‐ sumer sees the event, it is already an immutable part of the log, and it may have already been seen by other consumers.
4. Thus, any validation of a command needs to happen synchronously, before it becomes an event

## State, Streams, and Immutability
1. This principle of immutability is also what makes event sourcing and change data capture so powerful.
2. . The key idea is that mutable state and an append-only log of immut‐ able events do not contradict each other: they are two sides of the same coin. The log of all changes, the changelog, represents the evolution of state over time

### Advantages of immutable events
### Deriving several views from the same event log
1. Having an explicit translation step from an event log to a database makes it easier to evolve your application over time
2. you gain a lot of flexibility by sepa‐ rating the form in which data is written from the form it is read, and by allowing sev‐ eral different read views. This idea is sometimes known as command query responsibility segregation (CQRS)
3. The traditional approach to database and schema design is based on the fallacy that data must be written in the same form as it will be queried. 
### Concurrency control
1. The biggest downside of event sourcing and change data capture is that the consum‐ ers of the event log are usually asynchronous, so there is a possibility that a user may make a write to the log, then read from a log-derived view and find that their write has not yet been reflected in the read view
2. On the other hand, deriving the current state from an event log also simplifies some aspects of concurrency control. 
### Limitations of immutability
1. To what extent is it feasible to keep an immutable history of all changes forever? The answer depends on the amount of churn in the dataset. Some workloads mostly add data and rarely update or delete; they are easy to make immutable. Other workloads have a high rate of updates and deletes on a comparatively small dataset; in these cases, the immutable history may grow prohibitively large, fragmentation may become an issue, and the performance of compaction and garbage collection becomes crucial for operational robustness
2. there may also be circumstances in which you need data to be deleted for administrative reasons, in spite of all immutability
3. In these circumstances, it’s not sufficient to just append another event to the log to indicate that the prior data should be considered deleted—you actually want to rewrite history and pretend that the data was never written in the first place.
4. Truly deleting data is surprisingly hard [64], since copies can live in many places
# Processing Streams
1. The facilities for maintaining state as streams and replaying messages are also the basis for the techniques that enable stream joins and fault tolerance in various stream processing frameworks. 
1. you can process events. Broadly, there are three options:
   * You can take the data in the events and write it to a database, cache, search index, or similar storage system, from where it can then be queried by other clients. 
   * You can push the events to users in some way, for example by sending email alerts or push notifications, or by streaming the events to a real-time dashboard where they are visualized.
   * You can process one or more input streams to produce one or more output streams. A piece of code that processes streams like this is known as an operator or a job: a stream processor consumes input streams in a read-only fashion and writes its output to a different location in an append-only fashion. 
1. The one crucial difference to batch jobs is that a stream never ends.
   * sorting does not make sense with an unbounded dataset, and so sort-merge joins cannot be used. 
   * Fault-tolerance mechanisms must also change: with a batch job that has been running for a few minutes, a failed task can simply be restarted from the beginning, but with a stream job that has been running for several years, restarting from the beginning after a crash may not be a viable option.

## Uses of Stream Processing
### Complex event processing (CEP)
1. geared toward the kind of application that requires search‐ ing for certain event patterns. CEP allows you to specify rules to search for certain patterns of events in a stream
2. In these systems, the relationship between queries and data is reversed compared to normal databases.

### Stream analytics
1. analytics tends to be less interested in finding specific event sequences and is more oriented toward aggregations and statistical metrics over a large number of events
2. Stream analytics systems sometimes use probabilistic algorithms, 
   * such as Bloom filters (which we encountered in “Performance optimizations” on page 79) for set membership, 
   * HyperLogLog [72] for cardinality estimation, and various percentile estimation algorithms
1. Many open source distributed stream processing frameworks are designed with ana‐ lytics in mind: for example, Apache Storm, Spark Streaming, Flink, Concord, Samza, and Kafka Streams

### Maintaining materialized views
1. materialized views (see “Aggregation: Data Cubes and Materialized Views” on page 101): deriving an alternative view onto some dataset so that you can query it efficiently, and updating that view whenever the underlying data changes
   * caches, search indexes, and data warehouses
   * application state
### Search on streams
### Message passing and RPC
1. Although message-passing systems systems (e.g. actor model) are also based on messages and events, we normally don’t think of them as stream processors:
   * Actor frameworks are primarily a mechanism for managing concurrency and distributed execution of communicating modules, whereas stream processing is primarily a data management technique
   * Communication between actors is often ephemeral and one-to-one, whereas event logs are durable and multi-subscriber.
   * Actors can communicate in arbitrary ways (including cyclic request/response patterns), but stream processors are usually set up in acyclic pipelines where every stream is the output of one particular job, and derived from a well-defined set of input streams

1. there is some crossover area between RPC-like systems and stream pro‐ cessing.
   * Apache Storm has a feature called distributed RPC, which allows user queries to be farmed out to a set of nodes that also process event streams;
1. It is also possible to process streams using actor frameworks. However, many such frameworks do not guarantee message delivery in the case of crashes, so the process‐ ing is not fault-tolerant unless you implement additional retry logic


## Reasoning About Time
1. many stream processing frameworks use the local system clock on the processing machine (the processing time) to determine windowing
1. This approach has the advantage of being simple, and it is reasonable if the delay between event creation and event processing is negligibly short. 
2. However, it breaks down if there is any significant processing lag—i.e., if the processing may happen noticeably later than the time at which the event actually occurred.
### Event time versus processing time
1. message delays can also lead to unpredictable ordering of messages
2. Confusing event time and processing time leads to bad data
### Knowing when you’re ready
1. A tricky problem when defining windows in terms of event time is that you can never be sure when you have received all of the events for a particular window, or whether there are some events still to come
2. You need to be able to handle such straggler events that arrive after the window has already been declared complete. Broadly, you have two options
   * Ignore the straggler events, as they are probably a small percentage of events in normal circumstances.
   * Publish a correction, an updated value for the window with stragglers included
### Whose clock are you using, anyway?
1. Assigning timestamps to events is even more difficult when events can be buffered at several points in the system.
2. the clock on a user-controlled device often cannot be trusted, as it may be accidentally or deliberately set to the wrong time. one approach is to log three timestamps
   * The time at which the event occurred, according to the device clock
   * The time at which the event was sent to the server, according to the device clock
   * The time at which the event was received by the server, according to the server clock
### Types of windows
1. the next step is to decide how windows over time periods should be defined
2. Tumbling window: A tumbling window has a fixed length, and every event belongs to exactly one window. For
3. Hopping window: A hopping window also has a fixed length, but allows windows to overlap in order to provide some smoothing.
4. Sliding window: A sliding window contains all the events that occur within some interval of each other.
5. Session window: Unlike the other window types, a session window has no fixed duration. Instead, it is defined by grouping together all events for the same user that occur closely together in time, and the window ends when the user has been inactive for some time (for example, if there have been no events for 30 minutes). Sessionization is a common requirement for website analytics

## Stream Joins
1. the fact that new events can appear anytime on a stream makes joins on streams more challenging than in batch jobs.
### Stream-stream join (window join)
1. Both input streams consist of activity events, and the join operator searches for related events that occur within some window of time. For
1. To implement this type of join, a stream processor needs to maintain state
### Stream-table join (stream enrichment)
1. One input stream consists of activity events, while the other is a database change‐ log. The changelog keeps a local copy of the database up to date. For each activity event, the join operator queries the database and outputs an enriched activity event.
1. A stream-table join is actually very similar to a stream-stream join; the biggest differ‐ ence is that for the table changelog stream, the join uses a window that reaches back to the “beginning of time” (a conceptually infinite window), with newer versions of records overwriting older ones. For the stream input, the join might not maintain a window at all.
### Table-table join (materialized view maintenance)
1. Both input streams are database changelogs. In this case, every change on one side is joined with the latest state of the other side. The result is a stream of changes to the materialized view of the join between the two tables.
### Time-dependence of joins
1. The three types of joins described here (stream-stream, stream-table, and table-table) have a lot in common: they all require the stream processor to maintain some state (search and click events, user profiles, or follower list) based on one join input, and query that state on messages from the other join input.
2. The order of the events that maintain the state is important (it matters whether you first follow and then unfollow, or the other way round). In
3. but there is typically no ordering guarantee across different streams or partitions
4. This raises a question: if events on different streams happen around a similar time, in which order are they processed?
5. Put another way: if state changes over time, and you join with some state, what point in time do you use for the join
6. In data warehouses, this issue is known as a slowly changing dimension (SCD), and it is often addressed by using a unique identifier for a particular version of the joined record:
## Fault Tolerance
1. The same issue of fault tolerance arises in stream processing, but it is less straightfor‐ ward to handle: waiting until a task is finished before making its output visible is not an option, because a stream is infinite and so you can never finish processing it.
1. In particular, the batch approach to fault tolerance ensures that the output of the batch job is the same as if nothing had gone wrong, even if in fact some tasks did fail. It appears as though every input record was processed exactly once—no records are skipped, and none are processed twice. Although restarting tasks means that records may in fact be processed multiple times, the visible effect in the output is as if they had only been processed once. This principle is known as exactly-once semantics, although effectively-once would be a more descriptive term


### Microbatching and checkpointing
1. One solution is to break the stream into small blocks, and treat each block like a min‐ iature batch process. This approach is called microbatching, and it is used in Spark Streaming
2. Microbatching also implicitly provides a tumbling window equal to the batch size (windowed by processing time, not event timestamps); any jobs that require larger windows need to explicitly carry over state from one microbatch to the next.
3. A variant approach, used in Apache Flink, is to periodically generate rolling check‐ points of state and write them to durable storage
4. The checkpoints are triggered by bar‐ riers in the message stream, similar to the boundaries between microbatches, but without forcing a particular window size.
5. as soon as output leaves the stream processor (for example, by writing to a database, sending messages to an external message broker, or sending emails), the framework is no longer able to discard the output of a failed batch. In this case, restarting a failed task causes the external side effect to happen twice, and micro‐ batching or checkpointing alone is not sufficient to prevent this problem.

### Transactions: Atomic commit revisited
1. In order to give the appearance of exactly-once processing in the presence of faults, we need to ensure that all outputs and side effects of processing an event take effect if and only if the processing is successful. 
### Idempotence
1. Our goal is to discard the partial output of any failed tasks so that they can be safely retried without taking effect twice. Distributed transactions are one way of achieving that goal, but another way is to rely on idempotence
2. An idempotent operation is one that you can perform multiple times, and it has the same effect as if you performed it only once.
3. Relying on idempotence implies several assumptions: restarting a failed task must replay the same messages in the same order (a log-based message broker does this), the process‐ ing must be deterministic, and no other node may concurrently update the same value [98, 99].
4. When failing over from one processing node to another, fencing may be required (see “The leader and the lock” on page 301) to prevent interference from a node that is thought to be dead but is actually alive. Despite all those caveats, idempotent opera‐ tions can be an effective way of achieving exactly-once semantics with only a small overhead

### Rebuilding state after a failure
1. Any stream process that requires state—for example, any windowed aggregations (such as counters, averages, and histograms) and any tables and indexes used for joins—must ensure that this state can be recovered after a failure.
2. One option is to keep the state in a remote datastore and replicate it, although having to query a remote database for each individual message can be slow
3. An alternative is to keep state local to the stream processor, and replicate it periodically
4. Samza and Kafka Streams replicate state changes by sending them to a dedicated Kafka topic with log compaction, similar to change data capture
5. 

## Lambda Architecture
1. Lambda is a distributed data processing architecture that leverages both the batch and the real-time streaming data processing approaches to tackle the latency issues that arise out of the batch processing approach. It joins the results from both approaches before presenting them to the end-user.
2. The architecture has typically three layers:

Batch layer
Speed layer
Serving layer

## Kappa Architecture
1. In Kappa architecture, all the data flows through a single data streaming pipeline as opposed to the Lambda architecture, which has different data streaming layers that converge into one.
2. Kappa contains only two layers: Speed, which is the streaming processing layer, and serving, which is the final layer.

