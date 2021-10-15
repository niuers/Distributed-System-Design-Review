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

## Availability in numbers
1. Availability is often quantified by uptime (or downtime) as a percentage of time the service is available. 
2. Availability is generally measured in number of 9s--a service with 99.99% availability is described as having four 9s.

## Availability in parallel vs in sequence
If a service consists of multiple components prone to failure, the service's overall availability depends on whether the components are in sequence or in parallel.

### In sequence
1. Overall availability decreases when two components with availability < 100% are in sequence:
1. Availability (Total) = Availability (Foo) * Availability (Bar)
2. If both Foo and Bar each had 99.9% availability, their total availability in sequence would be 99.8%.

### In parallel
1. Overall availability increases when two components with availability < 100% are in parallel:
1. Availability (Total) = 1 - (1 - Availability (Foo)) * (1 - Availability (Bar))
2. If both Foo and Bar each had 99.9% availability, their total availability in parallel would be 99.9999%.


