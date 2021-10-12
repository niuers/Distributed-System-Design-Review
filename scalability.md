# Scale Up Web Server

## Vertical Scaling
1. Increase the power of CPU, storage of Disk (PATA, SATA-7200rmp,SAS-15000rpm, SSD), capacity of Memory
2. Constrained by real-world limits

## Horizontal Scaling
1. Increase the number of servers, each may still has a unique IP address
2. HTTP requests now need to be distributed across the servers through a load balancer
3. DNS instead of returning the IP address of the server 1 or 2 etc., now returns the IP address of the 'load balancer'. So the load balancer now has the public IP address.
   * The back-end servers now have private IP addresses, which can't be seen by public 
   * Load balancer can now send requests to the servers using TCP/IP, 
4. It's not a good idea to put DB server on the same web server, as a user can connect to another server, which doesn't have his/her information there.

## Load Balancer

## Caching
1. .html: Craiglist saves data as .html and re-use them to improve performance instead of in XML, MySQL and generate pages dynamically
   * It increases storage cost
   * You also have to store the same data somewhere in DB if you allow edit. So there's redundancy.
   * Another redundancy: same HTML tags in every single page 
   * It's very hard to change the layout etc. of these pages now.
3. MySQL Query Cache: disabled-by-default since MySQL 5.6 (2013) as it is known to not scale with high-throughput workloads on multi-core machines.
4. memcached
   * You can run out of RAM for the cache
   * We can expire objects based on when they are put in (FIFO), or recently used. 

## Replication
1. Leader-follower
    1. Improve on redundancy
    2. DB Load Balancer to improve read performance: For read heavy service, the followers can handle read requests, the leader can handle write requests.
2. Leader-Leader
   * Multiple leaders to remove the single point of failur  from one leader (after its failure and before a follower is promoted to leader)  
3. We can have replication of network swtiches as well
4. Have the system in a different Data Centers
   * Need global load balancer (DNS) to direct the traffic to different DCs
   * Need to make sure you get sticky session 

## Partition
## High Availability
## Security
1. Types of traffic allowed to come in
   * TCP:80
   * TCP:443: default port for SSL for HTTP based urls
   * TCP:22, for SSH or SSL based VPN to connect to the DC remotely
3. Types of traffic from Load Balancer to web servers (TCP:80)
   * It's common to offload your SSL to the load balancer or some special device to keep everything else unencrypted as you control it.
   * So everything from LB down stream can go as HTTP unencrypted, so you don't need put your SSL certificates on all the web servers. You can just put them on the load balancers.
4. Traffic from web servers to databases
   * TCP:3306, default MySQL port

# Resources:
1. [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://youtu.be/-W9F__D3oY4)
