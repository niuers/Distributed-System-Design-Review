# Distributed Message Queue

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
