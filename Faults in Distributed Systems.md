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
1. The variability of packet delays on computer networks is most often due to **queueing**
   * network congestion: On a busy network link, a packet may have to wait a while until it can get a slot
      * If several different nodes simultaneously try to send packets to the same destination, the network switch must queue them up and feed them into the destination network link one by one.
      * If there is so much incoming data that the switch queue fills up, the packet is dropped, so it needs to be resent—even though the network is functioning fine.
   * When a packet reaches the destination machine, if all CPU cores are currently busy, the incoming request from the network is queued by the operating system until the application is ready to handle it.
   * In virtualized environments, a running operating system is often paused for tens of milliseconds while another virtual machine uses a CPU core. During this time, the VM cannot consume any data from the network, so the incoming data is queued (buffered) by the virtual machine monitor [26], further increasing the variability of network delays.
   * TCP performs flow control (also known as **congestion avoidance or backpressure**), in which a node limits its own rate of sending in order to avoid overloading a network link or the receiving node. This means additional queueing at the sender before the data even enters the network.
   * Moreover, TCP considers a packet to be lost if it is not acknowledged within some timeout (which is calculated from observed round-trip times), and lost packets are automatically retransmitted. Although the application does not see the packet loss and retransmission, it does see the resulting delay (waiting for the timeout to expire, and then waiting for the retransmitted packet to be acknowledged).
1. In public clouds and multi-tenant datacenters, resources are shared among many customers: the network links and switches, and even each machine’s network inter‐ face and CPUs are shared. Batch workloads such as MapReduce can easily saturate network links. net‐ work delays can be highly variable if someone near you (a noisy neighbor) is using a lot of resources
2. In such environments, you can only choose timeouts experimentally
1. Even better, rather than using configured constant timeouts, systems can continually measure response times and their variability (**jitter**), and automatically adjust time‐outs according to the observed response time distribution. This can be done with a Phi Accrual failure detector, which is used for example in Akka and Cassandra. TCP retransmission timeouts also work similarly

## Synchronous Versus Asynchronous Networks
1. the tradi‐ tional fixed-line telephone network is synchronous: When you make a call over the telephone network, it establishes a circuit: a fixed, guaranteed amount of bandwidth is allocated for the call, along the entire route between the two callers.
   * And because there is no queueing, the maximum end-to-end latency of the network is fixed. We call this a bounded delay.
1. The packets of a TCP connection opportunistically use whatever network bandwidth is available. They are not reserved.
1. Ethernet and IP are packet-switched protocols, which suffer from queueing and thus unbounded delays in the network. These protocols do not have the concept of a circuit. This is because they are optimized for bursty traffic.
2. Thus, using circuits for bursty data transfers wastes network capacity and makes transfers unnecessarily slow. By contrast, TCP dynamically adapts the rate of data transfer to the available network capacity.
3. We have to assume that *network congestion, queueing, and unbounded delays* will happen. Consequently,

### Latency and Resource Utilization
1. More generally, you can think of variable delays as a consequence of dynamic resource partitioning.
2. multi-tenancy with dynamic resource partitioning provides better utilization, so it is cheaper, but it has the downside of variable delays.

# Unreliable Clocks
1. The time when a message is received is always later than the time when it is sent, but due to variable delays in the network, we don’t know how much later. This fact sometimes makes it difficult to determine the order in which things happened when multiple machines are involved.
2. It is possible to synchronize clocks to some degree: the most commonly used mechanism is the Network Time Protocol (NTP), which allows the computer clock to be adjusted according to the time reported by a group of servers. The servers in turn get their time from a more accurate time source, such as a GPS receiver.


## Monotonic Versus Time-of-Day Clocks
1. Modern computers have at least two different kinds of clocks: a time-of-day clock and a monotonic clock.

### Time-of-day clocks
1. it returns the current date and time according to some calendar (also known as wall-clock time)
2. if the local clock is too far ahead of the NTP server, it may be forcibly reset and appear to jump back to a previous point in time. These jumps, as well as the fact that they often ignore leap seconds, make time-of-day clocks unsuitable for measuring elapsed time
### Monotonic clocks
1. A monotonic clock is suitable for measuring a duration (time interval), such as a timeout or a service’s response time
2. The name comes from the fact that they are guaranteed to always move forward (whereas a time-of- day clock may jump back in time).
3. it makes no sense to compare monotonic clock values from two different computers, because they don’t mean the same thing
4. NTP may adjust the frequency at which the monotonic clock moves forward (this is known as slewing the clock) if it detects that the computer’s local quartz is moving faster or slower than the NTP server
5. but NTP cannot cause the monotonic clock to jump forward or backward. The resolution of monotonic clocks is usually quite good: on most systems they can measure time intervals in microseconds or less.
6. In a distributed system, using a monotonic clock for measuring elapsed time (e.g., timeouts) is usually fine, because it doesn’t assume any synchronization between dif‐ ferent nodes’ clocks and is not sensitive to slight inaccuracies of measurement.

## Clock Synchronization and Accuracy
1. Monotonic clocks don’t need synchronization, but time-of-day clocks need to be set according to an NTP server or other external time source in order to be useful
   * The quartz clock in a computer is not very accurate: it drifts (runs faster or slower than it should). This drift limits the best possible accuracy you can achieve, even if everything is working correctly. 
      * Google assumes a 6 ms drift for a clock that is resynchronized with a server every 30 seconds, or 17 seconds drift for a clock that is resynchronized once a day.
   * If a computer’s clock differs too much from an NTP server, it may refuse to syn‐ chronize, or the local clock will be forcibly reset
   * NTP synchronization can only be as good as the network delay, so there is a limit to its accuracy when you’re on a congested network with variable packet delays.
   * Leap seconds result in a minute that is 59 seconds or 61 seconds long, which messes up timing assumptions in systems that are not designed with leap seconds in mind. The best way of handling leap seconds may be to make NTP servers “lie,” by performing the leap second adjustment gradually over the course of a day (this is known as smearing)
   * In virtual machines, the hardware clock is virtualized, which raises additional challenges for applications that need accurate timekeeping

## Relying on Synchronized Clocks
1. if you use software that requires synchronized clocks, it is essential that you also carefully monitor the clock offsets between all the machines. Any node whose clock drifts too far from the others should be declared dead and removed from the cluster.

### Timestamps for ordering events
1. Let’s consider one particular situation in which it is tempting, but dangerous, to rely on clocks: ordering of events across multiple nodes.
2. e.g. in multi-leader replication, when a write is replicated to other nodes, it is tagged with a timestamp according to the time-of-day clock on the node where the write originated. if we use conflict resoltuion strategy of LWW (Cassandra, Riak), we may lose the later update if we use time stamp to order. There can be more issues with LWW: 
   * Database writes can mysteriously disappear: a node with a lagging clock is unable to overwrite values previously written by a node with a fast clock until the clock skew between the nodes has elapsed
   * LWW cannot distinguish between writes that occurred sequentially in quick succession and writes that were truly concurrent. Additional causality tracking mechanisms, such as version vectors are needed in order to prevent violations of causality
   * It is possible for two nodes to independently generate writes with the same time‐ stamp, especially when the clock only has millisecond resolution. 
1. Thus, even though it is tempting to resolve conflicts by keeping the most “recent” value and discarding others, it’s important to be aware that the definition of “recent” depends on a local time-of-day clock, which may well be incorrect.
2. Could NTP synchronization be made accurate enough that such incorrect orderings cannot occur? Probably not, because NTP’s synchronization accuracy is itself limited by the network round-trip time, in addition to other sources of error such as quartz drift. For
3. So-called logical clocks [56, 57], which are based on incrementing counters rather than an oscillating quartz crystal, are a safer alternative for ordering events

### Clock readings have a confidence interval
1. it doesn’t make sense to think of a clock reading as a point in time—it is more like a range of times, within a confidence interval
2. If you’re getting the time from a server, the uncertainty is based on the expected quartz drift since your last sync with the server, plus the NTP server’s uncertainty, plus the network round-trip time to the server (to a first approximation, and assuming you trust the server)
### Synchronized clocks for global snapshots
1. 

## Process Pauses

# Knowledge, Truth and Lies
## The Truth is defined by the majority
## Byzantine Faults
## System Model and Reality

