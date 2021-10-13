
# Replication
## Replication Types 
1. Leader-follower or Master-Slave
   * Read on followers and write on leader
   * Improve on redundancy
   * DB Load Balancer to improve read performance: For read heavy service, the followers can handle read requests, the leader can handle write requests.
2. Leader-Leader
   * Multiple leaders to remove the single point of failur  from one leader (after its failure and before a follower is promoted to leader)  
3. Tree-Replication
4. Buddy Replication

## Active-Passiv Replication
### Push: Active Replication

### Pull: Passive Replication
1. Data not available, read from peers, then store locally
2. Works well with time-out caches


## More Applications
3. We can have replication of network swtiches as well
4. Have the system in a different Data Centers
   * Need global load balancer (DNS) to direct the traffic to different DCs
   * Need to make sure you get sticky session 



