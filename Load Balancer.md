
# Load Balancer

1. Load balancer distribute incoming data traffic among computing resources such as application servers and databases. 
2. Load balancers are effective at:
   * (Reliability) Preventing requests from going to unhealthy servers
   * (Scalability) Preventing overloading resources
   * Helping to eliminate a single point of failure
   * SSL Termination
   * Session Persistence
3. Horizontal scaling
1. Load balancers can also help with horizontal scaling, improving performance and availability.
1. Disadvantages
   * Scaling horizontally introduces complexity and involves cloning servers
   * Servers should be stateless: they should not contain any user-related data like sessions or profile pictures
   * Sessions can be stored in a centralized data store such as a database (SQL, NoSQL) or a persistent cache (Redis, Memcached)
   * Downstream servers such as caches and databases need to handle more simultaneous connections as upstream servers scale out


## Disadvantage(s): load balancer
1. The load balancer can become a performance bottleneck if it does not have enough resources or if it is not configured properly.
1. Introducing a load balancer to help eliminate a single point of failure results in increased complexity.
1. A single load balancer is a single point of failure, configuring multiple load balancers further increases complexity.

## Networking Protocols Served by Load Balancer
1. TCP: 
   * Simply forward network packets without inspecting the content of the packets
   * Super fast, millions of requests per second 
1. HTTP: 
 * It terminates the connection, load balancer gets a HTTP request from a client, establishes a connection to a server, and sends request to the server, 
 * It can look inside the message and make a load-balacing decision based on the content of the message, e.g. cookie, header, 

## Layer 4 vs. Layer 7 Load Balancers
1. Layer 4
   * Layer 4 load balancers look at info at the transport layer to decide how to distribute requests. Generally, this involves the source, destination IP addresses, and ports in the header, but not the contents of the packet. Layer 4 load balancers forward network packets to and from the upstream server, performing Network Address Translation (NAT). For Internet traffic specifically, a Layer 4 load balancer bases the load-balancing decision on the source and destination IP addresses and ports recorded in the packet header, without considering the contents of the packet.
   * Transport(Layer 4) This layer provides transparent transfer of data between end systems, or hosts, and is responsible for end-to-end error recovery and flow control. It ensures complete data transfer.
   * Only has access to TCP and UDP data
   * Faster
   * Lack of information can lead to uneven traffic
   * It's good on your edge of your data center or your network because it can look at the ip address and for example, if you are getting a Denial of Service attack, instead of wastiing processing power allowing it through your web server, you can just toss that request right at the edge. So a lot of data route all incoming traffice throgh  a L4 load balancer before allowing it further into your application. 
   * Today the term “Layer 4 load balancing” most commonly refers to a deployment where the load balancer’s IP address is the one advertised to clients for a web site or service (via DNS, for example). As a result, clients record the load balancer’s address as the destination IP address in their requests.
   * When the Layer 4 load balancer receives a request and makes the load balancing decision, it also performs Network Address Translation (NAT) on the request packet, changing the recorded destination IP address from its own to that of the content server it has chosen on the internal network. Similarly, before forwarding server responses to clients, the load balancer changes the source address recorded in the packet header from the server’s IP address to its own. (The destination and source TCP port numbers recorded in the packets are sometimes also changed in a similar way.)
   * Layer 4 load balancers make their routing decisions based on address information extracted from the first few packets in the TCP stream, and do not inspect packet content. A Layer 4 load balancer is often a dedicated hardware device supplied by a vendor and runs proprietary load-balancing software, and the NAT operations might be performed by specialized chips rather than in software.
   * Layer 4 load balancing was a popular architectural approach to traffic handling when commodity hardware was not as powerful as it is now, and the interaction between clients and application servers was much less complex. It requires less computation than more sophisticated load balancing methods (such as Layer 7), but CPU and memory are now sufficiently fast and cheap that the **performance advantage for Layer 4 load balancing has become negligible or irrelevant in most situations**.

1. Layer 7
   * Layer 7 load balancers look at the application layer to decide how to distribute requests. This can involve contents of the header, message, and cookies. Layer 7 load balancers terminate network traffic, reads the message, makes a load-balancing decision, then opens a connection to the selected server. For example, a layer 7 load balancer can direct video traffic to servers that host videos while directing more sensitive user billing traffic to security-hardened servers.
At the cost of flexibility, layer 4 load balancing requires less time and computing resources than Layer 7, although the performance impact can be minimal on modern commodity hardware.
   * A Layer 7 load balancer terminates the network traffic and reads the message within. It can make a load‑balancing decision based on the content of the message (the URL or cookie, for example). It then makes a new TCP connection to the selected upstream server (or reuses an existing one, by means of HTTP keepalives) and writes the request to the server.
   * Application(Layer 7) This layer supports application and end-user processes. Communication partners are identified, quality of service is identified, user authentication and privacy are considered, and any constraints on data syntax are identified. Everything at this layer is application-specific. This layer provides application services for file transfers, e-mail, and other network software services.
   * Full access to HTTP protocol and data
   * SSL termination: Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
      * Removes the need to install X.509 certificates on each server
   * Check for authentication
   * Smarter routing options
1. Layer 4 vs. Layer 7
   * Layer 7 load balancers operate at the highest level in the OSI model, the application layer (on the Internet, HTTP is the dominant protocol at this layer). Layer 7 load balancers base their routing decisions on various characteristics of the HTTP header and on the actual contents of the message, such as the URL, the type of data (text, video, graphics), or information in a cookie.

Taking into consideration so many more aspects of the information being transferred can make Layer 7 load balancing more expensive than Layer 4 in terms of time and required computing power, but it can nevertheless lead to greater overall efficiency. For instance, because a Layer 7 load balancer can determine what type of data (video, text, and so on) a client is requesting, you don’t have to duplicate the same data on all of the load-balanced servers.

Modern general-purpose load balancers, such as NGINX Plus and the open source NGINX software, generally operate at Layer 7 and serve as full reverse proxies. Rather than manage traffic on a packet-by-packet basis like Layer 4 load balancers that use NAT, Layer 7 load balancing proxies can read requests and responses in their entirety. They manage and manipulate traffic based on a full understanding of the transaction between the client and the application server.

Some load balancers can be configured to provide Layer 4 or Layer 7 load balancing, depending on the nature of the service. As mentioned previously, modern commodity hardware is generally powerful enough that the savings in computational cost from Layer 4 load balancing are not large enough to outweigh the benefits of greater flexibility and efficiency from Layer 7 load balancing.

1. The more accurate term for layer 4 is “Layer 3/4 load balancing” – because the load balancer bases its decision on both the IP addresses of the origin and destination servers (Layer 3) and the TCP port number of the applications (Layer 4). The more exact term for “Layer 7 load balancing” might be “Layer 5 through 7 load balancing” because HTTP combines the functions of OSI Layers 5, 6, and 7.
1. Layer 7 load balancing is more CPU‑intensive than packet‑based Layer 4 load balancing, but rarely causes degraded performance on a modern server. Layer 7 load balancing enables the load balancer to make smarter load‑balancing decisions, and to apply optimizations and changes to the content (such as compression and encryption). It uses buffering to offload slow connections from the upstream servers, which improves performance.
1. A device that performs Layer 7 load balancing is often referred to as a reverse‑proxy server.



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
### Assign based on Session/cookies
### Assign based on Transport Layer
1. Layer 4
2. Layer 7


## Deal with the First Golden Rule for Scalability
Every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory. 

### Sticky Sessions/Session Persistence

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
   * Store the ID of the server in the cookie, but that exposes the private IP to external world. Also, what if the IP changed for the server?
   * It's better to store the hash of the corresponding server. This can be set by the load balancer.
   * It breaks down when user disables the cookie though.
3. Hash based Loading balancer

## Redundancy of Load Balancer
1. Load balancer can be a single point of failure
2. We can have multiple load balancers, 
   * Active-Active load balancer pairs
      * In active-active, both servers are managing traffic, spreading the load between them.
      * If the servers are public-facing, the DNS would need to know about the public IPs of both servers. If the servers are internal-facing, application logic would need to know about both servers.
      * Active-active failover can also be referred to as master-master failover.
   * Active-Passive Fail-over load balancer pairs
      * heartbeats are sent between the active and the passive server on standby. If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.
      * The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby. Only the active server handles traffic.
      * Active-passive failover can also be referred to as master-slave failover.
   * Disadvantages of Fail-over
      * Fail-over adds more hardware and additional complexity.
      * There is a potential for loss of data if the active system fails before any newly written data can be replicated to the passive.





## Web Server
1. Nginx
2. Apache

# Nginx
1. Whereas many web servers and application servers use a simple threaded or process‑based architecture, NGINX stands out with a sophisticated event‑driven architecture that enables it to scale to hundreds of thousands of concurrent connections on modern hardware.

## The NGINX Process Model
1. NGINX has a master process (which performs the privileged operations such as reading configuration and binding to ports) and a number of worker and helper processes (Cache loader/Cache manager, cache helper processes which manage the on‑disk content cache).
1. From the Linux OS perspective, threads and processes are mostly identical; the major difference is the degree to which they share memory
2. Processes and threads consume resources. They each use memory and other OS resources, and they need to be swapped on and off the cores (an operation called a context switch). Most modern servers can handle hundreds of small, active threads or processes simultaneously, but performance degrades seriously once memory is exhausted or when high I/O load causes a large volume of context switches.
3. The common way to design network applications is to assign a thread or process to each connection. This architecture is simple and easy to implement, but it does not scale when the application needs to handle thousands of simultaneous connections.

### How Does NGINX Work?
1. NGINX uses a predictable process model that is tuned to the available hardware resources:
1. The master process performs the privileged operations such as reading configuration and binding to ports, and then creates a small number of child processes (the next three types).
   * The cache loader process runs at startup to load the disk‑based cache into memory, and then exits. It is scheduled conservatively, so its resource demands are low.
   * The cache manager process runs periodically and prunes entries from the disk caches to keep them within the configured sizes.
   * The worker processes do all of the work! They handle network connections, read and write content to disk, and communicate with upstream servers.
1. The NGINX configuration recommended in most cases – running one worker process per CPU core – makes the most efficient use of hardware resources.
2. When an NGINX server is active, only the worker processes are busy. Each worker process handles multiple connections in a nonblocking fashion, reducing the number of context switches. Each worker process is single‑threaded and runs independently, grabbing new connections and processing them. The processes can communicate using shared memory for shared cache data, session persistence data, and other shared resources.





