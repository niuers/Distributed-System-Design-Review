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

