
# Load Balancer

1. Load balancer distribute data traffic among multiple servers

## Networking Protocols Served by Load Balancer
1. TCP: 
   * Simply forward network packets without inspecting the content of the packets
   * Super fast, millions of requests per second 
1. HTTP: 
 * It terminates the connection, load balancer gets a HTTP request from a client, establishes a connection to a server, and sends request to the server, 
 * It can look inside the message and make a load-balacing decision based on the content of the message, e.g. cookie, header, 


## Types of Load Balancer
### Software Load Balancer
1. AWS ELB (Elastic Load Balancer)
2. HAProxy (High Availability Proxy, open source)
3. LVS (Linux Virtual Server) 

### Hardware Load Balancer
Network devices that are powerful machines optimized to handle very high throughput: millions of requests per second
1. Barracuuda
2. Cisco
3. Citrix


## Load Balancer Algorithms
### Round Robin 
It distributes the requests in order across the list of servers. 

1. We can use a fancy DNS as simple load balancer, when user types a domain in browser, DNS just sends the request to different servers in order.
   * A popular DNS server called BIND
   * Disadvantages
      * One server may get disproportionally large amount of load by certain long-time request and doesn't respond to later requests
      * Since the browser can cache the session for a user, a heavy user can hit the same server all the time as well.
   * TTL: **Time to Live**
      * This is a value associated with an answer from the DNS server.
      * Only If a request lasts longer than TTL, a user is re-assgined to another server
1. We can use a separate load balancer (than DNS)
   * It can avoid the user-end cache issue.
   * But you can still have a large load on a specific server
   * Also sessions can be broken in PHP if our back-end server is PHP based, and they are using the session superglobal 
      * Load balancer may break with sessions, which is specific to the machine(/temp in Linux). When you are directed to another server, you may have to be asked to log in again, or your shopping cart won't have all the stuffs you buy.

### Randomly Assignment

### Least Connections
It sends requests to the server with the lowest number of active connections.

### Least Response Time
It sends requests to the server with the fastest response time.

### Hash Based Algrithms
It distributes requests based on a key we define, such as the client IP address or the request URL. 

## Deal with the First Golden Rule for Scalability

Every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory. 

### Sticky Sessions

Even if you visit website multiple times, you should end up with the same session, or the same back-end server

1. Use Shared Persistent Storage
   * We can have a file server that connects to every server and stores session data on hard disk. However, it now becomes single point of failure.
      * We can store the session data on RAID
         * RAID0: you write 'stripe' of data across two hard disks, double the write speed
         * RAID1: you write mirror data  in two hard disks.
   * Use Database: MySQL
   * Use NFS (network file system)
   * Persistent Cache, e.g. Redis
   * However, your single file server can still go down. Here we need Replication.

2. Use Cookies
   * Store the ID of the server in the cookie, but that exposes the private IP to external world. Also, what if the IP changed  for the server?
   * It's better to store the hash of the corresponding server. This can be set by the load balancer.
   * It breaks down when user disables the cookie though.

## Redundancy of Load Balancer
1. Load balancer can be a single point of failure
2. We can have multiple load balancers, 
   * Active-Active/Passive load balancer pairs
      * They send heartbeat from/to each other to check the status. If an active load balancer dies, the passive one can take over its IP address.