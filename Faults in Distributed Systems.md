# Overview
1. We assume that anything that can go wrong will go wrong except that faults are non-Byzantine.

# Faults and Partial Failures
1. In a distributed system, there may well be some parts of the system that are broken in some unpredictable way, even though other parts of the system are working fine. This is known as a **partial failure**
   * Whenever you try to send a packet over the network, it may be lost or arbitrarily delayed. Likewise, the reply may be lost or delayed, so if you don’t get a reply, you have no idea whether the message got through.
   * A node’s clock may be significantly out of sync with other nodes (despite your best efforts to set up NTP), it may suddenly jump forward or back in time
   * A process may pause for a substantial amount of time at any point in its execu‐ tion (perhaps due to a stop-the-world garbage collector), be declared dead by other nodes, and then come back to life again without realizing that it was paused.
3. The difficulty is that partial failures are nondeterministic: if you try to do anything involving multiple nodes and the network, it may sometimes work and sometimes unpredictably fail.
4. You may not even know whether something succeeded or not, as the time it takes for a message to travel across a network is also nondeterministic!
5. This nondeterminism and possibility of partial failures is what makes distributed sys‐ tems hard to work with
6. Examples
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
1. Whenever any communication happens over a network, it may fail—there is no way around it.

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
1. However, timeouts can’t distinguish between network and node failures, and variable network delay sometimes causes a node to be falsely suspected of crash‐ ing. Moreover, sometimes a node can be in a degraded state: for example, a Gigabit network interface could suddenly drop to 1 Kb/s throughput due to a driver bug [94]. Such a node that is “limping” but not dead can be even more difficult to deal with than a cleanly failed node.
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
1. The most common implementation of snapshot isolation requires a monotonically increasing transaction ID.
2. when a database is distributed across many machines, potentially in multi‐ ple datacenters, a global, monotonically increasing transaction ID (across all parti‐ tions) is difficult to generate, because it requires coordination.
3. With lots of small, rapid transactions, creating transaction IDs in a distributed system becomes an untenable bottleneck
4. Can we use the timestamps from synchronized time-of-day clocks as transaction IDs? If we could get the synchronization good enough, they would have the right properties: later transactions have a higher timestamp. The problem, of course, is the uncer‐ tainty about clock accuracy.

## Process Pauses
1. In a single-leader per partition DB, how does a node know that it is still leader (that it hasn’t been declared dead by the others), and that it may safely accept writes?
2. One option is for the leader to obtain a lease from the other nodes, which is similar to a lock with a timeout. In order to remain leader, the node must periodically renew the lease before it expires.
   * it may rely on synchronized clocks: the expiry time on the lease is set by a different machine and it’s being compared to the local system clock
   * even if we change the protocol to only use the local monotonic clock, there is another problem: the code assumes that very little time passes between the point that it checks the time and the time when the request is processed. a thread might be paused for a long time:
      * Many programming language runtimes (such as the Java Virtual Machine) have a garbage collector (GC) that occasionally needs to stop all running threads. These “stop-the-world” GC pauses have sometimes been known to last for several minutes [64]!
      * In virtualized environments, a virtual machine can be suspended (pausing the execution of all processes and saving the contents of memory to disk) and resumed (restoring the contents of memory and continuing execution). This pause can occur at any time in a process’s execution and can last for an arbitrary length of time.
      * On end-user devices such as laptops, execution may also be suspended and resumed arbitrarily, e.g., when the user closes the lid of their laptop.
      * When the operating system context-switches to another thread, or when the hypervisor switches to a different virtual machine (when running in a virtual machine), the currently running thread can be paused at any arbitrary point in the code.
      * If the application performs synchronous disk access, a thread may be paused waiting for a slow disk I/O operation to complete
      * If the operating system is configured to allow swapping to disk (paging), a simple memory access may result in a page fault that requires a page from disk to be loaded into memory. The thread is paused while this slow I/O operation takes place. If memory pressure is high, this may in turn require a different page to be swapped out to disk. In extreme circumstances, the operating system may spend most of its time swapping pages in and out of memory and getting little actual work done (this is known as thrashing). To avoid this problem, paging is often disabled on server machines (if you would rather kill a process to free up mem‐ ory than risk thrashing)
      * A Unix process can be paused by sending it the SIGSTOP signal, for example by pressing Ctrl-Z in a shell.
   * All of these occurrences can preempt the running thread at any point and resume it at some later time, without the thread even noticing. The problem is similar to making multi-threaded code on a single machine thread-safe: you can’t assume anything about timing, because arbitrary context switches and parallelism may occur.
   * A node in a distributed system must assume that its execution can be paused for a significant length of time at any point, even in the middle of a function. During the pause, the rest of the world keeps moving and may even declare the paused node dead because it’s not responding

### Response Time Guarantees
1. real-time on the web describes servers pushing data to clients and stream processing without hard response time constraints
2. For most server-side data processing systems, real-time guarantees are simply not economical or appropriate. Consequently, these systems must suffer the pauses and clock instability that come from operating in a non-real-time environment.

### Limiting the impact of garbage collection
1. An emerging idea is to treat GC pauses like brief planned outages of a node, and to let other nodes handle requests from clients while one node is collecting its garbage
2. A variant of this idea is to use the garbage collector only for short-lived objects (which are fast to collect) and to restart processes periodically, before they accumu‐ late enough long-lived objects to require a full GC of long-lived objects

# Knowledge, Truth and Lies
1. distributed systems are different from programs running on a single computer: there is no shared memory, only message passing via an unreliable network with variable delays, and the systems may suffer from partial failures, unreliable clocks, and processing pauses.
2. In a dis‐ tributed system, we can state the assumptions we are making about the behavior (the system model) and design the actual system in such a way that it meets those assump‐ tions
3. Algorithms can be proved to function correctly within a certain system model. This means that reliable behavior is achievable, even if the underlying system model provides very few guarantees.


## The Truth is defined by the majority
1. many distributed algorithms rely on a quorum, that is, voting among the nodes: decisions require some minimum number of votes from several nodes in order to reduce the dependence on any one particular node
   * That includes decisions about declaring nodes dead. If a quorum of nodes declares another node dead, then it must be considered dead, even if that node still very much feels alive.
### The leader and the lock
1. Frequently, a system requires there to be only one of some thing
   * Only one leader to avoid split brain
   * Only one transaction or client is allowed to hold the lock for a particular resource or object, to prevent concurrently writing to it and corrupting it
   * Only one user is allowed to register a particular username
1. Implementing this in a distributed system requires care: even if a node believes that it is “the chosen one” (the leader of the partition, the holder of the lock, the request handler of the user who successfully grabbed the username), that doesn’t necessarily mean a quorum of nodes agrees!
2. If a node continues acting as the chosen one, even though the majority of nodes have declared it dead, it could cause problems in a system that is not carefully designed.
#### Fencing tokens
1. When using a lock or lease to protect access to some resource, such as the file storage, we need to ensure that a node that is under a false belief of being “the chosen one” cannot disrupt the rest of the system. A fairly simple technique that achieves this goal is called fencing
2. Let’s assume that every time the lock server grants a lock or lease, it also returns a fencing token, which is a number that increases every time a lock is granted (e.g., incremented by the lock service). We can then require that every time a client sends a write request to the storage service, it must include its current fencing token.
3. If ZooKeeper is used as lock service, the transaction ID zxid or the node version cversion can be used as fencing token. Since they are guaranteed to be monotoni‐ cally increasing, they have the required properties
4. Note that this mechanism requires the resource itself to take an active role in check‐ ing tokens by rejecting any writes with an older token than one that has already been processed—it is not sufficient to rely on clients checking their lock status themselves

## Byzantine Faults
1. Byzantine fault: if a node lies ( send arbitrary faulty or corrupted responses), e.g. if a node may claim to have received a particular message when in fact it didn’t. 
2. Here we can usually safely assume that there are no Byzantine faults. 
3. In peer-to-peer networks, where there is no such cen‐ tral authority, Byzantine fault tolerance is more relevant
### Weak forms of lying
1. Although we assume that nodes are generally honest, it can be worth adding mecha‐ nisms to software that guard against weak forms of “lying”—for example, invalid messages due to hardware issues, software bugs, and misconfiguration.

## System Model and Reality
1. Many algorithms have been designed to solve distributed systems problems. 
2. Algorithms need to be written in a way that does not depend too heavily on the details of the hardware and software configuration on which they are run. This in turn requires that we somehow formalize the kinds of faults that we expect to happen in a system. We do this by defining a system model, which is an abstraction that describes what things an algorithm may assume.
3. For modeling real systems, the partially synchronous model with crash-recovery faults is generally the most useful model. 
4. how do distributed algorithms cope with that model?
### Timing Issues
1. Synchronous model: The synchronous model assumes bounded network delay, bounded process pau‐ ses, and bounded clock error.
1. Partially Synchronous model: Partial synchrony means that a system behaves like a synchronous system most of the time, but it sometimes exceeds the bounds for network delay, process pauses, and clock drift [88]. This is a realistic model of many systems. 
1. Asynchronous model: In this model, an algorithm is not allowed to make any timing assumptions—in fact, it does not even have a clock

### Node Failures
1. Crash-stop faults: a node can fail in only one way, namely by crashing
2. Crash-recovery faults
3. Byzantine (arbitrary) faults

### Correctness of an algorithm
1. we can write down the properties we want of a distributed algorithm to define what it means to be correct. For
2. An algorithm is correct in some system model if it always satisfies its properties in all situations that we assume may occur in that system model. But how does this make sense? If all nodes crash, or all network delays suddenly become infinitely long, then no algorithm will be able to get anything done.

### Saftey and liveness
1. A giveaway is that liveness properties often include the word “eventually” in their definition
2. Safety is often informally defined as nothing bad happens, and liveness as something good eventually happens.
3. For distributed algorithms, it is common to require that safety properties always hold, in all possible situations of a system model
4. However, with liveness properties we are allowed to make caveats: for example, we could say that a request needs to receive a response only if a majority of nodes have not crashed, and only if the network eventually recovers from an outage. 
5. The defini‐ tion of the partially synchronous model requires that eventually the system returns to a synchronous state—that is, any period of network interruption lasts only for a finite duration and is then repaired.

### Mapping system models to the real world
1. Safety and liveness properties and system models are very useful for reasoning about the correctness of a distributed algorithm
