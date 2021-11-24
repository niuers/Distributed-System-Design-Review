# Overview
1. The problem with daily batch processes is that changes in the input are only reflected in the output a day later, which is too slow for many impatient users.
2. we will look at event streams as a data management mechanism: the unbounded, incrementally processed counterpart to the batch data we saw in the last chapter

# Transmitting Event Stream
1. When the input is a file (a sequence of bytes), the first processing step is usually to parse it into a sequence of records. In a stream processing context, a record is more commonly known as an event, but it is essentially the same thing: a small, self- contained, immutable object containing the details of something that happened at some point in time.
2. in streaming terminology, an event is generated once by a producer (also known as a publisher or sender), and then potentially processed by multiple con‐ sumers (subscribers or recipients
3. In a filesystem, a filename identifies a set of  related records; in a streaming system, related events are usually grouped together into a topic or stream.
4. in principle, a file or database is sufficient to connect producers and consumers: a producer writes every event that it generates to the datastore, and each consumer periodically polls the datastore to check for events that have appeared since it last ran.
5. However, when moving toward continual processing with low delays, polling becomes expensive if the datastore is not designed for this kind of usage. The more often you poll, the lower the percentage of requests that return new events, and thus the higher the overheads become. Instead, it is better for consumers to be notified when new events appear.
6. 
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
1. is essentially a kind of database that is optimized for handling message streams [13]. It runs as a server, with producers and consumers connecting to it as clients. Producers write messages to the broker, and consumers receive them by reading them from the broker
2. By centralizing the data in the broker, these systems can more easily tolerate clients that come and go (connect, disconnect, and crash), and the question of durability is moved to the broker instead
3. Faced with slow consumers, they generally allow unboun‐ ded queueing (as opposed to dropping messages or backpressure), although this choice may also depend on the configuration.
4. A consequence of queueing is also that consumers are generally asynchronous: when a producer sends a message, it normally only waits for the broker to confirm that it has buffered the message and does not wait for the message to be processed by con‐ sumers. The delivery to consumers will happen at some undetermined future point in time—often within a fraction of a second, but sometimes significantly later if there is a queue backlog.

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
1. 
## Change Data Capture
## Event Sourcing
## State, Streams, and Immutability
# Processing Streams
## Uses of Stream Processing
## Reasoning About Time
## Stream Joins
## Fault Tolerance

