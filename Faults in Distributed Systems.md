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
4. 

# Unreliable Networks
## Network Faults in Practice
## Detecting Faults
## Timeouts and Unbounded Delays
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

