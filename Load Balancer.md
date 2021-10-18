
# Load Balancer

1. Load balancer distribute incoming data traffic among multiple servers
2. Used to improve reliability and scalibility of application


## Networking Protocols Served by Load Balancer
1. TCP: 
   * Simply forward network packets without inspecting the content of the packets
   * Super fast, millions of requests per second 
1. HTTP: 
 * It terminates the connection, load balancer gets a HTTP request from a client, establishes a connection to a server, and sends request to the server, 
 * It can look inside the message and make a load-balacing decision based on the content of the message, e.g. cookie, header, 

## L4 vs. L7
1. Layer 4
   * Only has access to TCP and UDP data
   * Faster
   * Lack of information can lead to uneven traffic
   * It's good on your edge of your data center or your network because it can look at the ip address and for example, if you are getting a Denial of Service attack, instead of wastiing processing power allowing it through your web server, you can just toss that request right at the edge. So a lot of data route all incoming traffice throgh  a L4 load balancer before allowing it further into your application. 
1. Layer 7
   * Full access to HTTP protocol and data
   * SSL termination
   * Check for authentication
   * Smarter routing options

## Types of Load Balancer

### Software Load Balancer
1. AWS ELB (Elastic Load Balancer)
3. LVS (Linux Virtual Server) 
1. Reverse Proxy
   * HAProxy (High Availability Proxy, open source)
   * Nginx
   * Apache mod_proxy
   * A reverse proxy accepts a request from a client, forwards it to a server that can fulfill it, and returns the server’s response to the client.
       * Whereas deploying a load balancer makes sense only when you have multiple servers, it often makes sense to deploy a reverse proxy even with just one web server or application server. You can think of the reverse proxy as a website’s “public face.” Its address is the one advertised for the website, and it sits at the edge of the site’s network to accept requests from web browsers and mobile apps for the content hosted at the website. The benefits are two-fold:
       * Increased security and Increased scalability and flexibility


### Hardware Load Balancer
Network devices that are powerful machines optimized to handle very high throughput: millions of requests per second
1. Barracuuda
2. Cisco
3. Citrix
4. F5


## Load Balancer Algorithms

### Round Robin
In a nutshell, round-robin algorithms pair an incoming request to a specific machine by cycling (or, more specifically, circling) through a list of servers capable of handling the request. It distributes the requests in order across the list of servers. 

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

#### Round Robin vs. Weighted Round Robin
1. Weighted round-robin provides a clean and effective way of focusing on fairly distributing the load amongst available resources, verses attempting to equally distribute the requests. In a weighted round-robin algorithm, each destination (in this case, server) is assigned a value that signifies, relative to the other servers in the pool, how that server performs. This “weight” determines how many more (or fewer) requests are sent that server’s way; compared to the other servers on the pool.
1. Weights must be positive integer values; a node with a weight of ‘4’ will receive 4 times as many requests as a node with a weight of ‘1’.


### Randomly Assignment
### Dynamic Load Balancing
1. Least Connections: It sends requests to the server with the lowest number of active connections.
   * Chat or streaming services
3. Least Response Time (Latency Based): It sends requests to the server with the fastest response time.
4. Hash Based Algrithms: It distributes requests based on a key we define, such as the client IP address or the request URL. 
   * Useful to maintain a stateful session
   * Amazon shopping cart, when you refresh your page, the same information sent to the same server

### Weighted Allocation

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
3. Hash based Loading balancer

## Redundancy of Load Balancer
1. Load balancer can be a single point of failure
2. We can have multiple load balancers, 
   * Active-Active/Passive load balancer pairs
      * They send heartbeat from/to each other to check the status. If an active load balancer dies, the passive one can take over its IP address.

## Web Server
1. Ninginx
2. Apache
