# Distributed System
## Fallacies of Distributed Systems
1. Network is reliable
2. Latency is zero
3. topology doesn't change
4. Network is secure
5. Bandwidth is infinite
6. Only one administor
7. transport cost is zero

## Common Distributed System Characteristics
1. No shared clock
   * Clock drift: clocks of different machines get out of sync, cause issues with ordering of events
   * Negative Latency: Time travel
   * Google Solution: GPS receiver, atomic clock
3. No shared memory
   * 
5. Shared resources
   * Anything in your distriubted system should be able to be shared between nodes on the system so that could be hardware, software or data 
7. Concurrency and Consistency
8. Different parts of Distributed System need to be able to talk
   * 
10. Requires agreed upon format or protocol

## Distributed System Communication
1. Lots of things can go wrong, need to handle them somehow
   * Client can't find server
   * Server crash mid request
   * Server response is lost (e.g. network)
   * Client crashes
## Benefits
1. More reliable, fault tolerant
2. Scalability
3. Lower latency, increased performance
4. Cost effective

## Performance Metrics
### Scalability 
1. Ability of a system to grow and manage increased traffic
2. Increased volume of data per request or total number of requests over time

### Reliability
1. Probability a system will fail during a period of time
2. Slightly harder to define than hardware reliability
3. Mean Time Between Failure (MTBF)
   * MTBF = (Total Elapsed Time - Total downtime)/number of failures

### Availability
1. Amount of time a system is operational during a period of time
2. Pooly designed software requiring downtime for updates is less available
3. Availability %: (available time/total time)*100%, Five 9's

#### Reliable vs. Available
1. Reliable system is always an available system
2. Availability can be maintained by redundancy, but system may not be reliable.
3. Reliable software will be more profitable because providing same service requires less backup resources
4. Requirements will depend on function of the software

### Efficiency
1. How well the system performs
2. Latency and throughput often used as metrics

### Manageability
1. Speed and difficulty involved with maintaining system
2. Observability, how hard to track bugs
3. Difficulty of deploying updates
4. Want to abstract away infrastructure so product engineers don't have to worry about it

## Numbers Programmers Should Know
### Latency Numbers
[System Design Course for Beginners, 21:27](https://youtu.be/MbjObHmDbZo)
1. Avoid network calls whenever possible
2. Replicate data across data centers for disaster recovery as well as performance
3. Use CDNs to reduce latency
4. Keep frequently accessed data in memory if possible rather than seeking from disk, caching


### Quick Math for Capacity Estimates
[System Design Course for Beginners, 27:58](https://youtu.be/MbjObHmDbZo)

### Common Data Types
1. Char: 1 byte
2. Integer: 4 bytes
3. UNIX Timestamp: 4 bytes

### Time:
1. 60x60 = 3600 Seconds per hour
2. 3600*24 hours = 86,400 seconds per day
3. 86400*30 days = 2.5 million seconds per month

### Traffic Estimates
1. DAU: Daily Active Users
2. Traffic: (Average DAU X average reads/writes per user)/86400 = requests per second

### Memory Estimates
1. Read Requests per day x Average Request size x 0.2 x Replication Number
2. 80-20 rule: 20% of your data will be 80% of your traffic

### Bandwidth Requestments
1. Requests per day x Request size / 864000 = bandwidth per second

### Storage
1. Writes per day x Size of write x Time to store data
2. 



