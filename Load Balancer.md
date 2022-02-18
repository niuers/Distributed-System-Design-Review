
# Load Balancer

1. Load balancer distribute incoming data traffic among computing resources such as application servers and databases. 
2. Load balancers are effective at:
   * (Reliability) Preventing requests from going to unhealthy servers
      * To ensure that the user request is always routed to the machine that is up and running, load balancers regularly perform health checks on the machines in the cluster.
      * Ideally, a load balancer maintains a list of machines that are up and running in the cluster in real-time, and the user requests are forwarded to only those machines that are in service.
   * (Scalability) Preventing overloading resources
   * Helping to eliminate a single point of failure
   * SSL Termination
   * Session Persistence
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
### Layer 4
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

1. When you use TCP (layer 4) for both front-end and back-end connections, your load balancer forwards the request to the back-end instances without modifying the headers. After your load balancer receives the request, it attempts to open a TCP connection to the back-end instance on the port specified in the listener configuration. 
2. ELB: Using this configuration, you do not receive cookies for session stickiness or X-Forwarded headers.



### Layer 7
   * Layer 7 load balancers look at the application layer to decide how to distribute requests. This can involve contents of the header, message, and cookies. Layer 7 load balancers terminate network traffic, reads the message, makes a load-balancing decision, then opens a connection to the selected server. For example, a layer 7 load balancer can direct video traffic to servers that host videos while directing more sensitive user billing traffic to security-hardened servers.
At the cost of flexibility, layer 4 load balancing requires less time and computing resources than Layer 7, although the performance impact can be minimal on modern commodity hardware.
   * A Layer 7 load balancer terminates the network traffic and reads the message within. It can make a load‑balancing decision based on the content of the message (the URL or cookie, for example). It then makes a new TCP connection to the selected upstream server (or reuses an existing one, by means of HTTP keepalives) and writes the request to the server.
   * Application(Layer 7) This layer supports application and end-user processes. Communication partners are identified, quality of service is identified, user authentication and privacy are considered, and any constraints on data syntax are identified. Everything at this layer is application-specific. This layer provides application services for file transfers, e-mail, and other network software services.
   * Full access to HTTP protocol and data
   * SSL termination: Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
      * Removes the need to install X.509 certificates on each server
   * Check for authentication
   * Smarter routing options
1. When you use HTTP (layer 7) for both front-end and back-end connections, your load balancer parses the headers in the request and terminates the connection before sending the request to the back-end instances.
1. ELB: When you use HTTP/HTTPS, you can enable sticky sessions on your load balancer. A sticky session binds a user's session to a specific back-end instance. This ensures that all requests coming from the user during the session are sent to the same back-end instance.


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
#### AWS ELB (Elastic Load Balancer)
#### LVS (Linux Virtual Server) 

#### Reverse Proxy
1. [A reverse proxy server](https://www.imperva.com/learn/performance/reverse-proxy/) is an intermediate connection point positioned at a network’s edge. It receives initial HTTP connection requests, acting like the actual endpoint.
2. Essentially your network’s traffic cop, the reverse proxy serves as a gateway between users and your application origin server. In so doing it handles all policy management and traffic routing.
3. A reverse proxy operates by:
   * Receiving a user connection request
   * Completing a TCP three-way handshake, terminating the initial connection
   * Connecting with the origin server and forwarding the original request

1. HAProxy (High Availability Proxy, open source)
1. Nginx
1. Apache mod_proxy
1. A reverse proxy accepts a request from a client, forwards it to a server that can fulfill it, and returns the server’s response to the client.
   * Whereas deploying a load balancer makes sense only when you have multiple servers, it often makes sense to deploy a reverse proxy even with just one web server or application server. You can think of the reverse proxy as a website’s “public face.” Its address is the one advertised for the website, and it sits at the edge of the site’s network to accept requests from web browsers and mobile apps for the content hosted at the website. The benefits are two-fold:
   * Increased security and Increased scalability and flexibility

1. Advantages of Reverse Proxy
   * Content caching (CDN uses reverse proxy technology)
   * Traffic scrubbing
      * DDos (Distributed denial of service) Mitigation
      * Web application security
   * IP masking
   * Load Balancing



1. In contrast, a forward proxy server is also positioned at your network’s edge, but regulates outbound traffic according to preset policies in shared networks. Additionally, it disguises a client’s IP address and blocks malicious incoming traffic.
2. Forward proxies are typically used internally by large organizations, such as universities and enterprises, to:
   * Block employees from visiting certain websites
   * Monitor employee online activity
   * Block malicious traffic from reaching an origin server
   * Improve the user experience by caching external site content



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
4. Sometimes on the internet, you will find a few percent of the clients which disable cookies on their browser. Obviously they have troubles everywhere on the web, but you can still help them access your site by using the "source" balancing algorithm instead of the "roundrobin". It ensures that a given IP address always reaches the same server as long as the number of servers remains unchanged. Never use this behind a proxy or in a small network, because the distribution will be unfair. However, in large internal networks, and on the internet, it works quite well. Clients which have a dynamic address will not be affected as long as they accept the cookie, because the cookie always has precedence over load balancing.

### KeepAlive
1. Keepalived implements VRRP (Virtual Router Redundancy Protocol) on a Linux system as well as managing Linux Virtual Server configuration. Keepalived can implement High Availability (active/passive) and load balancing (active/active) setups that can be made responsive to several customisable factors.
2. A keepalive (KA) is a message sent by one device to another to check that the link between the two is operating, or to prevent the link from being broken.





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
1. To better understand this design, you need to understand how NGINX runs. There’s one worker process per core to make efficient use of hardware resources, the ability to interleave multiple connections within a single worker process, and the capability to switch from connection to connection almost instantaneously as network traffic arrives. Put this magic together and you create the massively scalable  **HTTP application delivery engine** that is NGINX.
1. nginx (pronounced "engine x") is a free open source web server written by Igor Sysoev, a Russian software engineer. Since its public launch in 2004, nginx has focused on high performance, high concurrency and low memory usage. Additional features on top of the web server functionality, like load balancing, caching, access and bandwidth control, and the ability to integrate efficiently with a variety of applications, makes it the second most popular open source web server on the Internet.


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
      * Each NGINX worker process is initialized with the NGINX configuration and is provided with a set of listen sockets by the master process.
      * Events are initiated by new incoming connections. These connections are assigned to a state machine – the HTTP state machine is the most commonly used, but NGINX also implements state machines for stream (raw TCP) traffic and for a number of mail protocols (SMTP, IMAP, and POP3).
      * The state machine is essentially the set of instructions that tell NGINX how to process a request.
      * Think of the state machine like the rules for chess. Each HTTP transaction is a chess game. On one side of the chessboard is the web server – a grandmaster who can make decisions very quickly. On the other side is the remote client – the web browser that is accessing the site or application over a relatively slow network.
      * Most web application platforms use A Blocking I/O (State Machine): Most web servers and web applications use a process‑per‑connection or thread‑per‑connection model to play the chess game. Each process or thread contains the instructions to play one game through to the end. During the time the process is run by the server, it spends most of its time ‘blocked’ – waiting for the client to complete its next move.
      * The web server process listens for new connections (new games initiated by clients) on the listen sockets.
      * When it gets a new game, it plays that game, blocking after each move to wait for the client’s response.
      * Once the game completes, the web server process might wait to see if the client wants to start a new game (this corresponds to a keepalive connection). If the connection is closed (the client goes away or a timeout occurs), the web server process returns to listening for new games.
      * The important point to remember is that every active HTTP connection (every chess game) requires a dedicated process or thread (a grandmaster). This architecture is simple and easy to extend with third‑party modules (‘new rules’). However, there’s a huge imbalance: the rather lightweight HTTP connection, represented by a file descriptor and a small amount of memory, maps to a separate thread or process, a very heavyweight operating system object. It’s a programming convenience, but it’s massively wasteful.

1. The NGINX configuration recommended in most cases – running one worker process per CPU core – makes the most efficient use of hardware resources.
2. When an NGINX server is active, only the worker processes are busy. Each worker process handles multiple connections in a nonblocking fashion, reducing the number of context switches. Each worker process is single‑threaded and runs independently, grabbing new connections and processing them. The processes can communicate using shared memory for shared cache data, session persistence data, and other shared resources.
3. NginX uses a non-blocking "Event-driven" architecture: That’s how an NGINX worker process plays “chess.” Each worker (remember – there’s usually one worker for each CPU core) is a grandmaster that can play hundreds (in fact, hundreds of thousands) of games simultaneously.
   * The worker waits for events on the listen and connection sockets.
   * Events occur on the sockets and the worker handles them:
      * An event on the listen socket means that a client has started a new chess game. The worker creates a new connection socket.
      * An event on a connection socket means that the client has made a new move. The worker responds promptly.
   * A worker never blocks on network traffic, waiting for its “opponent” (the client) to respond. When it has made its move, the worker immediately proceeds to other games where moves are waiting to be processed, or welcomes new players in the door.
1. NGINX scales very well to support hundreds of thousands of connections per worker process. Each new connection creates another file descriptor and consumes a small amount of additional memory in the worker process. There is very little additional overhead per connection. NGINX processes can remain pinned to CPUs. Context switches are relatively infrequent and occur when there is no work to be done.

In the blocking, connection‑per‑process approach, each connection requires a large amount of additional resources and overhead, and context switches (swapping from one process to another) are very frequent.






#### Rate Limiting and Concurrency Limiting

## Security
You can create a load balancer with the following security features.

### SSL server certificates
1. If you use HTTPS or SSL for your front-end connections, you must deploy an X.509 certificate (SSL server certificate) on your load balancer. The load balancer decrypts requests from clients before sending them to the back-end instances (known as SSL termination). 
1. If you don't want the load balancer to handle the SSL termination (known as SSL offloading), you can use TCP for both the front-end and back-end connections, and deploy certificates on the registered instances handling requests.

### SSL negotiation
1. Elastic Load Balancing provides predefined SSL negotiation configurations that are used for SSL negotiation when a connection is established between a client and your load balancer. The SSL negotiation configurations provide compatibility with a broad range of clients and use high-strength cryptographic algorithms called ciphers. However, some use cases might require all data on the network to be encrypted and allow only specific ciphers. Some security compliance standards (such as PCI, SOX, and so on) might require a specific set of protocols and ciphers from clients to ensure that the security standards are met. In such cases, you can create a custom SSL negotiation configuration, based on your specific requirements. Your ciphers and protocols should take effect within 30 seconds. For more information, see SSL negotiation configurations for Classic Load Balancers.

### Back-end server authentication
If you use HTTPS or SSL for your back-end connections, you can enable authentication of your registered instances. You can then use the authentication process to ensure that the instances accept only encrypted communication, and to ensure that each registered instance has the correct public key.


# Reverse proxy (web server)
1. A reverse proxy is a web server that centralizes internal services and provides unified interfaces to the public. Requests from clients are forwarded to a server that can fulfill it before the reverse proxy returns the server's response to the client.

## Additional benefits include:
1. Increased security - Hide information about backend servers, blacklist IPs, limit number of connections per client
1. Increased scalability and flexibility - Clients only see the reverse proxy's IP, allowing you to scale servers or change their configuration
1. SSL termination - Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
   * Removes the need to install X.509 certificates on each server
1. Compression - Compress server responses
1. Caching - Return the response for cached requests
1. Static content - Serve static content directly
   * HTML/CSS/JS
   * Photos
   * Videos
   * Etc

### Load balancer vs reverse proxy
1. Deploying a load balancer is useful when you have multiple servers. Often, load balancers route traffic to a set of servers serving the same function.
1. Reverse proxies can be useful even with just one web server or application server, opening up the benefits described in the previous section.
1. Solutions such as NGINX and HAProxy can support both layer 7 reverse proxying and load balancing.
### Disadvantage(s): reverse proxy
1. Introducing a reverse proxy results in increased complexity.
1. A single reverse proxy is a single point of failure, configuring multiple reverse proxies (ie a failover) further increases complexity.

#### Common Points
1. Both types of application sit between clients and servers, accepting requests from the former and delivering responses from the latter. No wonder there’s confusion about what’s a reverse proxy vs. load balancer.

#### Differences
1. Load balancers are most commonly deployed when a site needs multiple servers because the volume of requests is too much for a single server to handle efficiently. Deploying multiple servers also eliminates a single point of failure, making the website more reliable. Most commonly, the servers all host the same content, and the load balancer’s job is to distribute the workload in a way that makes the best use of each server’s capacity, prevents overload on any server, and results in the fastest possible response to the client.
2. A load balancer can also enhance the user experience by reducing the number of error responses the client sees. It does this by detecting when servers go down, and diverting requests away from them to the other servers in the group. In the simplest implementation, the load balancer detects server health by intercepting error responses to regular requests. 
3. Another useful function provided by some load balancers is session persistence, which means sending all requests from a particular client to the same server.
#### Reverse Proxy
4. It often makes sense to deploy a reverse proxy even with just one web server or application server. You can think of the reverse proxy as a website’s “public face.” Its address is the one advertised for the website, and it sits at the edge of the site’s network to accept requests from web browsers and mobile apps for the content hosted at the website. The benefits are two-fold:
   * Increased security – No information about your backend servers is visible outside your internal network, so malicious clients cannot access them directly to exploit any vulnerabilities. Many reverse proxy servers include features that help protect backend servers from distributed denial-of-service (DDoS) attacks, for example by rejecting traffic from particular client IP addresses (blacklisting), or limiting the number of connections accepted from each client.
   * Increased scalability and flexibility – Because clients see only the reverse proxy’s IP address, you are free to change the configuration of your backend infrastructure. This is particularly useful In a load-balanced environment, where you can scale the number of servers up and down to match fluctuations in traffic volume.

1.Another reason to deploy a reverse proxy is for web acceleration – reducing the time it takes to generate a response and return it to the client. Techniques for web acceleration include the following:
   * Compression – Compressing server responses before returning them to the client (for instance, with gzip) reduces the amount of bandwidth they require, which speeds their transit over the network.
   * SSL termination – Encrypting the traffic between clients and servers protects it as it crosses a public network like the Internet. But decryption and encryption can be computationally expensive. By decrypting incoming requests and encrypting server responses, the reverse proxy frees up resources on backend servers which they can then devote to their main purpose, serving content.
   * Caching – Before returning the backend server’s response to the client, the reverse proxy stores a copy of it locally. When the client (or any client) makes the same request, the reverse proxy can provide the response itself from the cache instead of forwarding the request to the backend server. This both decreases response time to the client and reduces the load on the backend server.

1. Every load balancer that operates at layer seven (http) is a reverse proxy, but not every reverse proxy is a load balancer. You could say that a load balancer is a type of reverse proxy.
   * Load balancers that work at layer four (eg AWS NLB) or below are probably also reverse proxies, but since they don't parse requests like http packets they're not as functional and have fewer features. They're usually faster.
   * A load balancer's primary job is to take requests and distribute them to a number of servers to service the request. It may also do things like path based routing, so for example static resource requests are filled from one server farm or AWS S3, while application pages are filled by another server farm.
   * A reverse proxy, if it's not a load balancer, can be installed on a single server to send requests to another application on the server. For example, you may have Nginx or Apache in front of Tomcat, as they have more features than Tomcat and can protect Tomcat from some classes of attacks. For example, Apache may be configured to cache Tomcat responses if for some reason you don't want to do that in Tomcat.

1. A reverse proxy is a layer 7 load balancer (or, vice versa) that operates at the highest level applicable and provides for deeper context on the Application Layer protocols such as HTTP. By using additional application awareness, a reverse proxy or layer 7 load balancer has the ability to make more complex and informed load balancing decisions on the content of the message – whether it’s to optimise and change the content (HTTP header manipulation, compression and encryption) and/or monitor the health of applications to ensure reliability and availability. On the other hand, layer 4 load balancers are FAST routers rather than application (reverse) proxies where the client effectively talks directly (transparently) to the backend servers.  

1. All modern load balancers are capable of doing both – layer 4 as well as layer 7 load balancing, by acting either as reverse proxies (layer 7 load balancers) or routers (layer 4 load balancers). An initial tier of layer 4 load balancers can distribute the inbound traffic across a second tier of layer 7 (proxy-based) load balancers. Splitting up the traffic allows the computationally complex work of the proxy load balancers to be spread across multiple nodes. Thus, the two-tiered model serves far greater volumes of traffic than would otherwise be possible and therefore, is a great option for load balancing object storage systems – the demand for which has significantly exploded in the recent years.
1. Essentially Load Balancer is one of the applications of a Reverse Proxy.
1. Both reverse proxy and load balancer are key components of the client-server architecture. A client-server model is basically an application structure that separates between the providers of resources and the ones which are getting the services.
2. A forward proxy is essentially used by a client and a reverse proxy is used by servers.


## Load balancer Implementation
1. A moderately large system may balance load at three layers:
   * user to your web servers,
   * web servers to an internal platform layer,
   * internal platform layer to your database.

### Smart Client
1. What is a smart client? It is a client which takes a pool of service hosts and balances load across them, detects downed hosts and avoids sending requests their way (they also have to detect recovered hosts, deal with adding new hosts, etc, making them fun to get working decently and a terror to setup).
1. Seems impossible? 






