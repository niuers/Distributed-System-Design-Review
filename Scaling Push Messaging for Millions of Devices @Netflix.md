# Scaling Push Messaging for Millions of Devices @Netflix
1. Netflix has 125 million customers
## We need to update the recommendataions on user screen while the user is browsing the movies
### Old App uses Polling to get updates from server
These two goals are contradicting each other
1. UI Freshness: You have to increase polling frequencies to get more UI freshness, but will load the servers
2. Server Efficiency

3. Now the Server just sends a message to client as soon as it generates a new recommendation list

## What is Push?
2. There's a persistent connection between a client and server for the entirety of client's lifetime
3. It's server that initiates data transfer instead of client requests

## Zuul Push Architecture
1. A complete pupsh messaging infrastructure
2. Need support 1 million persistent connections with clients
3. Thread per connection: can't support that many connections, can't scale
4. Async IO: One thread registers multiple connections, development is more complicate
### Requirements for Push Registry Store
1. Low Read Latency
2. Record Expiry or TTL
3. Sharding 
4. Replication

Following Stores can do this:
1. Cassandra
2. Redis (Dynamite)
   * Auto Sharding
   * Read/Write Quorum
   * Cross Region Replication
4. Amazon DynamoDB

### Message Queuing Infrastructure

1. We use Kfaka to decouple senders and receivers
2. Most senders are fire-and-forget
3. Priority Inversion: it can happen when use 1 queue, need use different queues for different priorities


## It's hard to operation the Zull Push because of persistent connections
1. They make quick deployment/rollback complicated
2. They auto close connections periodically: balance between client stickness and efficiency
3. Randomize each connection's lifetime totemper the thundering herd. If there's a network problem, they will tend to re-connect at the same time, and the next/next time. 

## How to Optimize Push Server Size
1. Most connections are idle
2. Single server connects to thousands of clients, single server down can cause thundering herd.
3. Solution: Godilock strategy
   * You don't want the server be too small or too large
   * Amazon M4 Large
4. Should optimize for cost not instance count
   * Should prefer more number of smaller servers over few big servers, withstand better thundering herd
   * 
## How to Auto Scale?
1. Two main strategies for scaling
   * RPS: request per second
   * CPU: Average load per CPU
2. Both are ineffective for push cluster, only limiting factor is number of open connections
3. So should auto scale based on average number of open connections
    
4. Amazon Elastic Balancer can't proxy web sockets
5. ELB doesn't understand the intial client request, WebSocket upgrade request.
6. So they handle it as any other HTTP requests, so as the response is returned the definition is broken.
7. We make ELB run as Layer 4 TCP load balancer

## A/B Test
## They use JSON protocol
1. Could use binary protocol as well

1. Deduplication of messages
