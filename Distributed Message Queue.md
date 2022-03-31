# Distributed Message Queue
1. A publish-subscribe(pub-sub) model is the model where multiple consumers receive the same message sent from a single or multiple producers.
   * To implement the pub-sub pattern, message queues have exchanges that further push messages to the queues based on the exchange type and the set rules. Exchanges are just like telephone exchanges, which route messages from sender to the receiver through the infrastructure based on a certain logic.
   * The relationship between exchange and the queue is known as binding.
1. Point-to-Point Model: 
   * The use case for * Point-to-point* communication is pretty simple. It’s where the message from the producer is consumed by only one consumer.
3. Payload means the content of the message posted by the user.


## Handling Concurrent Requests With Message Queues
1. 
## Types of Exchanges
1. Direct, topic, headers, and fanout
## Message Protocols
1. AMQP Advanced Message Queue Protocol 
2. STOMP Simple or Streaming Text Oriented Message Protocol

## Communications between Producer and Consumer

### Synchronous Communication
1. Faster, easier to implement
2. Producer is Blocked for response (Assuming producer send requests to consumer)
3. Harder to deal with consumer service failure
4. How to deal with too many requests?
6. How to deal with slow consumer service ?

### Asynchronous Communication Using a Queue
1. Don't confuse queue with topic
   * In topic, the published message goes to each subscriber
   * In queue, message is received by one and only one consumer
1. 


## Distributed Message Queue
### Example Functional Requirements
1. sendmessage(message)
2. receivemessage()

### Examples of Non-Functional Requirements
1. Scalable: handles load/requests increases
2. Highly Available: survive failures
3. High Performant: single digit latency for operations
4. Durable: data is persist once submitted
5. Specific Service Level Agreements (SLAs)
   * Example, minimum throughput
6. Requirements around cost-effectiveness
   * Example, minimize operational cost

## VIP
1. Virtual IP addresses 
2. VIP refers to a symbolic hostname (e.g. myexample.domain.com) that resolves to a load balance system.
3. When a domain name is queried, request is transferred to one of the VIPs registered in DNS for our domain name
4. VIP is resolved to a load balance device which knows the front-end hosts
5. VIP Partitioning: 
   * We can assign multiple A records in DNS to the same domain name, so a request can be partitioned into several load balancers.
   * Improve both availability and performance
6. "Release It!" book by Michael T. Nygard (1st edition).
8. https://landing.google.com/sre/sre-book/chapters/load-balancing-frontend/

## Front-End WebServices
1. light-weight web service
2. Stateless service deployed across several data centers
3. The rule of thumb: To keep it as simple as possible
4. 
### Actions
1. Request Validation
   * Ensure all required parameters are present
   * Data falls within an acceptable range
3. Authentication/Authorization
   * Authentication: Validate identity of user or service
   * Authorization: Determine whether or not a specific actor is permitted to take an action
5. TLS(SSL) Termination
   * TLS is a protocol that aims to provide privacy and data integrity
   * TLS termination refers to the process of decrypting the request and passing on an unencrypted request to the back-end service
   * SSL on the load balancer is expensive
   * Termination is usually not handled by not the Front-End service itself, but a separate TLS HTTP proxy that runs as a process on the same host
7. Server-Side data encryption
   * Because  we want to save the message securely on back-end hosts, message are encrypted as soon as FrondEnd receive them
   * Messages are stored in encrypted form and frontend decrypts the messages only when they are sent back to consumer 
9. Caching
   * In FrontEnd cache, we store meta data about the mostly actively used queues as well as user identifications to save calls to auth service.
   * 
11. Rate Limiting (throttling)
   * It is the process of limiting the number of requests you can submit to a given operation in a given amount of time
   * Throttling protects the web from being overwhelmed with requests
   * Leaky bucket algorithm is the most famous
13. Request dispatching
   * Frontend service makes calls to at least two services: metadata and backend. Frontend service creates HTTP clients for both services and make sure that calls to these services are properly isolated. e.g. if metadata service is experiencing a slowdown, the requests to backend service are not impacted. 
   * Request dispatching is responsible for all the activities associated with sending requests to backend services (clients management, response handling, resources isolation etc.)
   * **Bulkhead Pattern** helps to isolate elements of an application into pools so that if one fails the others will continue
   * **Circuit Break** pattern prevents an application from repeatedly trying to execute an operation that's likely to fail
15. Request deduplication
   * It may occur when a response from a successful sendMessage request failed to reach a client
   * Lesser an issue for "at least once" delivery semantics, a bigger issue for 'exactly once' and 'at most once' delivery semantics
   * Caching is usually used to store previously seen request ids to avoid deduplication
17. Usage data collection
   * Real-time data for audit and billing

## MetaData Service
1. It stores information about queues
2. It's a caching layer between frontend and a persistent database (to store metadata)
3. It handles many reads, and relatively small number of writes (when new queue is created)
4. Strong consistency storage is preferred to avoid potential concurrent updates but not requried


## Ask Questions to Proceed
1. Where and how do we store the messages?
   * Database? How to build a highly scalable, high throughput DB? 
   * Memory?
   * File System?
   * Answer: RAM and local disk of a backend host
1. How do we replicate data ? 
   * Replicate copies within a group of hosts
1. How does Frontend select backend to send message to?    
3. How does Frontend know where to retrieve message from?    
   * Leverage metadata service to check where to send/retrieve message

## Backend Service
### OptionA: Leader-Follower Relationship
1. A backend instance is considered a leader for a particular set of queues. Meaning all requests for a particular queue go to this instance.
2. The leader is responsible for replicates in the followers
3. Need a component for leader election and management(in-cluster manager), e.g. ZooKeeper
   * Needs to be reliable, scalable, and performant

### Option B: A small cluster of independent hosts
1. Still need to manage queue to cluster assignments (out-cluster manager)


## Other Issues
1. Queue creation and deletion
2. Message deletion
   * Need maintain order of messages in a queue
      * Option 1: Kafka, only delete messages after serveral days
      * Option 2: Amazon SQS, mark messages as invisible so other consumers may not get already consumed messages
3. Message Replication
   * Synchronously: response is sent back to producer only after the data have been replicated
   * Asynchornously: response is sent back to producer as long as a single host has the message
4. Message Delivery Semantics
   * At most once: messages maybe lost but never redelivered
   * At least once:  messages are never lost but maybe redelivered (mostly supported by message queues), good balance between durability, availability and performance
   * Exactly once: each message is delivered once and only once (hard to achieve in practice)

5. Push vs. Pull
   * Pull model: consumer constantly sends retrieve message requests and when new message is available in the queue, it is sent back to a consumer. 
      * Pull is easier to implement than Push
      * From consumer perspective, we need to do more work in Pull model
   * Push model: consumer is not bombarding the frontend with receive calls, instead consumer is notified as soon as a new mesage arrives to a queue
6. FIFO
   * It's hard to maintaini a strict order in distributed queue.
7. Security
   * Need to make sure messages are securely transferred to and from a queue
      * Encryption using SSL over HTTPS helps to protect message in transit
      * We also encrypt messages while store in backend hosts
8. Monitoring
   * Monitor frontend, metadata, backend services
   * Provide visibility into customers' experience
   * 
# RabbitMQ
# In-Memory Message Queue using mmap

1. mmap: https://realpython.com/python-mmap/
 
1. Python’s mmap provides memory-mapped file input and output (I/O). It allows you to take advantage of lower-level operating system functionality to read files as if they were one large string or array. This can provide significant performance improvements in code that requires a lot of file I/O.
1. operating system uses page table to map from virtual memory to physical memory
1. mmap uses virtual memory to make it appear that you’ve loaded a very large file into memory, even if the contents of the file are too big to fit in your physical memory.
1. Python’s mmap uses shared memory to efficiently share large amounts of data between multiple Python processes, threads, and tasks that are happening concurrently
1. Memory mapping is another way to perform file I/O that can result in better performance and memory efficiency.
1. regular file I/O
Transferring control to the kernel or core operating system code with system calls
Interacting with the physical disk where the file resides
Copying the data into different buffers between user space (application buffer) and kernel space (read buffer)
1. All access with the physical hardware must happen in a protected environment called kernel space. System calls are the API that the operating system provides to allow your program to go from user space to kernel space, where the low-level details of the physical hardware are managed.
1. The most important thing to remember is that system calls are relatively expensive computationally speaking, so the fewer system calls you do, the faster your code will likely execute.
1. memory mapping doesn’t have to use more memory than the conventional approach. The operating system is very clever. It will lazily load the data as it’s requested
1. In addition, thanks to virtual memory, you can load a file that’s larger than your physical memory. However, you won’t see the huge performance improvements from memory mapping when there isn’t enough physical memory for your file, because the operating system will use a slower physical storage medium like a solid-state disk to mimic the physical memory it lacks.
1. Python’s mmap file object can be sliced just like string objects!
1. mmap objects as strings. mmap operates on bytes, not strings
1. Memory-Mapped Objects as Files: 
1. Sharing Data Between Processes With Python’s mmap
  * You can also create anonymous memory maps that have no physical storage
  * Data doesn’t have to be copied between processes.
  * The operating system handles the memory transparently.
  * Data doesn’t have to be pickled between processes, which saves CPU time.
  * mmap is incompatible with higher-level, more full-featured APIs like the built-in multiprocessing module. The multiprocessing module requires data passed between processes to support the pickle protocol, which mmap does not
1. Sharing between siblings: processes can use IPC to exchange the file descriptor or use an extra manage layer. Still need use IPC to exchange file descriptor.

