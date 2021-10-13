
# Replication

1. Leader-follower or Master-Slave
   * Read on followers and write on leader
   * Improve on redundancy
   * DB Load Balancer to improve read performance: For read heavy service, the followers can handle read requests, the leader can handle write requests.
2. Leader-Leader
   * Multiple leaders to remove the single point of failur  from one leader (after its failure and before a follower is promoted to leader)  
3. We can have replication of network swtiches as well
4. Have the system in a different Data Centers
   * Need global load balancer (DNS) to direct the traffic to different DCs
   * Need to make sure you get sticky session 

