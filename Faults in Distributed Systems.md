# Overview
1. We assume that anything that can go wrong will go wrong except that faults are non-Byzantine.

# Faults and Partial Failures
1. In a distributed system, there may well be some parts of the system that are broken in some unpredictable way, even though other parts of the system are working fine. This is known as a **partial failure**
2. The difficulty is that partial failures are nondeterministic: if you try to do anything involving multiple nodes and the network, it may sometimes work and sometimes unpredictably fail.
3. You may not even know whether something succeeded or not, as the time it takes for a message to travel across a network is also nondeterministic!
4. This nondeterminism and possibility of partial failures is what makes distributed sys‐ tems hard to work with
5. Examples
   * long-lived network partitions in a single data center (DC), 
   * PDU [power distribution unit] failures, 
   * switch failures, 
   * accidental power cycles of whole racks, 
   * whole-DC backbone failures, 
   * whole-DC power failures, and 
   * DC’s HVAC [heating, ven‐ tilation, and air conditioning] system

## Cloud Computing and Supercomputing
1. spectrum of philosophies on how to build large-scale computing systems
   * At one end of the scale is the field of high-performance computing (HPC)
      * Thus, a supercomputer is more like a single-node computer than a distributed system. it deals with partial failure by letting it escalate into total failure—if any part of the system fails, just let everything crash (like a kernel panic on a single machine)
   * At the other extreme is cloud computing
      * Many internet-related applications are online: low latency and availability
      * nodes in cloud services have higher failure rates
      * Large datacenter networks are often based on IP and Ethernet, arranged in Clos topologies to provide high bisection bandwidth
      * The bigger a system gets, the more likely it is that one of its components is bro‐ ken. In a system with thousands of nodes, it is reasonable to assume that something is always broken
      * If the system can tolerate failed nodes and still keep working as a whole, that is a very useful feature for operations and maintenance
      * In a geographically distributed deployment (keeping data geographically close to your users to reduce access latency), communication most likely goes over the internet, which is slow and unreliable compared to local networks.

1. we need to build a reliable system from unreliable components
   * IP (the Internet Protocol) is unreliable: it may drop, delay, duplicate, or reorder packets. TCP (the Transmission Control Protocol) provides a more reliable transport layer on top of IP: it ensures that missing packets are retransmitted, duplicates are eliminated, and packets are reassembled into the order in which they were sent.
   * Although the system can be more reliable than its underlying parts, there is always a limit to how much more reliable it can be.
      * TCP can hide packet loss, duplication, and reordering from you, but it cannot magically remove delays in the network.
3. The fault han‐ dling must be part of the software design, and you (as operator of the software) need to know what behavior to expect from the software in the case of a fault. In distributed systems, suspicion, pessimism, and paranoia pay off.

# Unreliable Networks
1. The internet and most internal networks in datacenters (often Ethernet) are *asynchronous packet networks*. In this kind of network, one node can send a message (a packet) to another node, but the network gives no guarantees as to when it will arrive, or whether it will arrive at all. If you send a request and expect a response, many things could go wrong:
   * Request may get lost
   * Request may be waiting in a queue and will be delivered later (overloaded network or recipient)
   * The rmote node failed
   * The remote node temporarily stopped responding but it'll start responding again later
   * The remote node may have processed your request, but the response is lost or delayed (your own network is overloaded)
1. The sender can’t even tell whether the packet was delivered: the only option is for the recipient to send a response message, which may in turn be lost or delayed.
2. If you send a request to another node and don’t receive a response, it is impossible to tell why.
   * The usual way of handling this issue is a timeout: after some time you give up waiting and assume that the response is not going to arrive.
1. Whenever any communi‐ cation happens over a network, it may fail—there is no way around it.

## Network Faults in Practice
1. network partition or netsplit: When one part of the network is cut off from the rest due to a network fault
2. Handling network faults doesn’t necessarily mean tolerating them: if your network is normally fairly reliable, a valid approach may be to simply show an error message to users while your network is experiencing problems.
3. However, you do need to know how your software reacts to network problems and ensure that the system can recover from them.

## Detecting Faults
1. the uncertainty about the network makes it difficult to tell whether a node is working or not.
## Timeouts and Unbounded Delays
1. A long timeout means a long wait until a node is declared dead (and during this time, users may have to wait or see error messages). 
2. A short timeout detects faults faster, but carries a higher risk of incorrectly declaring a node dead when in fact it has only suffered a temporary slowdown (e.g., due to a load spike on the node or the network).
3. Prematurely declaring a node dead is problematic: 
   * if the node is actually alive and in the middle of performing some action (for example, sending an email), and another node takes over, the action may end up being performed twice. 
   * When a node is declared dead, its responsibilities need to be transferred to other nodes, which places additional load on other nodes and the network. If the system is already struggling with high load, declaring nodes dead prematurely can make the problem worse
   * In particular, it could happen that the node actually wasn’t dead but only slow to respond due to overload; transferring its load to other nodes can cause a cascading failure (in the extreme case, all nodes declare each other dead, and every‐ thing stops working).
1. asynchronous networks have unbounded delays (that is, they try to deliver packets as quickly as possible, but there is no upper limit on the time it may take for a packet to arrive), and most server implementations cannot guarantee that they can handle requests within some maximum time. 

### Network congestion and queueing
1. The variability of packet delays on computer networks is most often due to queueing
   * network congestion: On a busy network link, a packet may have to wait a while until it can get a slot
      * If several different nodes simultaneously try to send packets to the same destination, the network switch must queue them up and feed them into the destination network link one by one.
      * If there is so much incoming data that the switch queue fills up, the packet is dropped, so it needs to be resent—even though the network is functioning fine.
   * When a packet reaches the destination machine, if all CPU cores are currently busy, the incoming request from the network is queued by the operating system until the application is ready to handle it.
   * In virtualized environments, a running operating system is often paused for tens of milliseconds while another virtual machine uses a CPU core. During this time, the VM cannot consume any data from the network, so the incoming data is queued (buffered) by the virtual machine monitor [26], further increasing the variability of network delays.
   * TCP performs flow control (also known as **congestion avoidance or backpressure**), in which a node limits its own rate of sending in order to avoid overloading a network link or the receiving node. This means additional queueing at the sender before the data even enters the network.
   * Moreover, TCP considers a packet to be lost if it is not acknowledged within some timeout (which is calculated from observed round-trip times), and lost packets are automatically retransmitted. Although the application does not see the packet loss and retransmission, it does see the resulting delay (waiting for the timeout to expire, and then waiting for the retransmitted packet to be acknowledged).


## Synchronous Versus Asynchronous Networks

# Unreliable Clocks
## Monotonic Versus Time-of-Day Clocks
## Clock Synchronization and Accuracy
## Relying on Synchronized Clocks
## Process Pauses

# Knowledge, Truth and Lies
## The Truth is defined by the majority
## Byzantine Faults
## System Model and Reality

