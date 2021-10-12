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

### Algorithms to decide how to distribute the requests
1. Round Robin: 
   * We can use a fancy DNS as simple load balancer, when user types a domain in browser, DNS just sends the request to different servers in order.
      * A popular DNS server called BIND
      * Disadvantages
         * One server may get disproportionally large amount of load by certain long-time request and doesn't respond to later requests
         * Since the browser can cache the session for a user, a heavy user can hit the same server all the time as well.
      * TTL: Time to Live
         * This is a value associated with an answer from the DNS server.
         * Only If a request lasts longer than TTL, a user is re-assgined to another server
   * We can use a separate load balancer
      * It can avoid the cache issue.
      * But you can still have a large load on a specific server
      * Sessions can be broken in PHP if our back-end server PHP based website, and they are using the session superglobal 
         * Load balancer may still break with sessions, which is specific to the machine(/temp in Linux). When you are directed to another server, you may have to be asked to log in again, or your shopping cart won't have all the stuffs you buy.
      * Sticky Sessions: even if you visit website multiple times, you are still going to end up with the same session, i.e. same back-end server
         * Shared Storage
            * We can have a file server that connects to every server and stores session data on hard disk. However, it now becomes single point of failure.
            * We can store the session data on RAID
               * RAID0: you write 'stripe' of data across two hard disks, double the write speed
               * RAID1: you write mirror data  in two hard disks.
            * MySQL
            * NFS (network file system)
            * However, your single file server can still go down. Here we need replication.
         * Cookies
            * Store the ID of the server in the cookie, but that exposes the private IP to external world. Also, what if the IP changed  for the server?
            * It's better to store the hash of the corresponding server. This can be set by the load balancer.
            * It breaks down when user disables the cookie though.
2. Randomly assign      
3. Take load etc. into account

### Software
1. AWS ELB (Elastic Load Balancer)
2. HAProxy (High Availability Proxy, open source)
3. LVS (Linux Virtual Server) 

### Hardware
1. Barracuuda
2. Cisco
3. Citrix

### Redundancy of Load Balancer
1. Load balancer can be a single point of failure
2. We can have multiple load balancers, 
   * Active-Active/Passive load balancer pairs
      * They send heartbeat from/to each other to check the status. If an active load balancer dies, the passive one can take over its IP address.
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
