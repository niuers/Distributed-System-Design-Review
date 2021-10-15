# Availability

## Availability Patterns
There are two complementary patterns to support high availability: fail-over and replication.

### Fail-Over

#### Active-passive

1. With active-passive fail-over, heartbeats are sent between the active and the passive server on standby. If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.
2. The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby. Only the active server handles traffic.
3. Active-passive failover can also be referred to as master-slave failover.

### Active-active
1. In active-active, both servers are managing traffic, spreading the load between them.
2. If the servers are public-facing, the DNS would need to know about the public IPs of both servers. If the servers are internal-facing, application logic would need to know about both servers.
3. Active-active failover can also be referred to as master-master failover.

### Disadvantage(s): failover
1. Fail-over adds more hardware and additional complexity.
2. There is a potential for loss of data if the active system fails before any newly written data can be replicated to the passive.

### [Replication](https://github.com/niuers/Distributed-System-Design-Review/blob/main/Replication.md)

