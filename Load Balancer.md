
# Load Balancer

1. Load balancer distribute incoming data traffic among computing resources such as application servers and databases. 
2. Load balancers are effective at:
   * (Reliability) Preventing requests from going to unhealthy servers
   * (Scalability) Preventing overloading resources
   * Helping to eliminate a single point of failure
   * SSL Termination
   * Session Persistence


## Networking Protocols Served by Load Balancer
1. TCP: 
   * Simply forward network packets without inspecting the content of the packets
   * Super fast, millions of requests per second 
1. HTTP: 
 * It terminates the connection, load balancer gets a HTTP request from a client, establishes a connection to a server, and sends request to the server, 
 * It can look inside the message and make a load-balacing decision based on the content of the message, e.g. cookie, header, 

## Layer 4 vs. Layer 7 Load Balancers
1. Layer 4
   * Transport(Layer 4) This layer provides transparent transfer of data between end systems, or hosts, and is responsible for end-to-end error recovery and flow control. It ensures complete data transfer.
   * Only has access to TCP and UDP data
   * Faster
   * Lack of information can lead to uneven traffic
   * It's good on your edge of your data center or your network because it can look at the ip address and for example, if you are getting a Denial of Service attack, instead of wastiing processing power allowing it through your web server, you can just toss that request right at the edge. So a lot of data route all incoming traffice throgh  a L4 load balancer before allowing it further into your application. 
1. Layer 7
   * Application(Layer 7) This layer supports application and end-user processes. Communication partners are identified, quality of service is identified, user authentication and privacy are considered, and any constraints on data syntax are identified. Everything at this layer is application-specific. This layer provides application services for file transfers, e-mail, and other network software services.
   * Full access to HTTP protocol and data
   * SSL termination: Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
      * Removes the need to install X.509 certificates on each server
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
### Assign based on Session/cookies
### Assign based on Transport Layer
1. Layer 4
2. Layer 7
3. Check the ["OSI 7 LAYER MODEL"](http://www.escotal.com/osilayer.html)
   * The OSI, or Open System Interconnection, model defines a networking framework for implementing protocols in seven layers. Control is passed from one layer to the next, starting at the application layer in one station, proceeding to the bottom layer, over the channel to the next station and back up the hierarchy.
   * Hypertext transfer protocol (HTTP)
      * HTTP is a method for encoding and transporting data between a client (browser) and a server. It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request. HTTP is self-contained, allowing requests and responses to flow through many intermediate routers and servers that perform load balancing, caching, encryption, and compression. This self‑contained design allows for the distributed nature of the Internet, where a request or response might pass through many intermediate routers and proxy servers.
      * Information is exchanged between clients and servers in the form of Hypertext documents (HTML). 
      * A basic HTTP request consists of a verb (method) and a resource (endpoint). Below are common HTTP verbs: GET, POST, PUT
      * HTTP is an application layer protocol relying on lower-level protocols such as TCP and UDP.
      * HTTP resources such as web servers are identified across the Internet using unique identifiers known as Uniform Resource Locators (URLs).
   * Transmission control protocol (TCP)
      * TCP is a connection-oriented protocol over an IP network. Connection is established and terminated using a handshake. All packets sent are guaranteed to reach the destination in the original order and without corruption through:
         * Sequence numbers and checksum fields for each packet
         * Acknowledgement packets and automatic retransmission
      * In telecommunications, a handshake is an automated process of negotiation between two participants (example "Alice and Bob") through the exchange of information that establishes the protocols of a communication link at the start of the communication, before full communication begins. The handshaking process usually takes place in order to establish rules for communication when a computer attempts to communicate with another device. Signals are usually exchanged between two devices to establish a communication link. For example, when a computer communicates with another device such as a modem, the two devices will signal each other that they are switched on and ready to work, as well as to agree to which protocols are being used.
      * If the sender does not receive a correct response, it will resend the packets. If there are multiple timeouts, the connection is dropped. TCP also implements flow control and congestion control. These guarantees cause delays and generally result in less efficient transmission than UDP.
      * In data communications, flow control is the process of managing the rate of data transmission between two nodes to prevent a fast sender from overwhelming a slow receiver. It provides a mechanism for the receiver to control the transmission speed, so that the receiving node is not overwhelmed with data from transmitting node. Flow control should be distinguished from congestion control, which is used for controlling the flow of data when congestion has actually occurred.[1] Flow control mechanisms can be classified by whether or not the receiving node sends feedback to the sending node.
      * Congestion control modulates traffic entry into a telecommunications network in order to avoid congestive collapse resulting from oversubscription. This is typically accomplished by reducing the rate of packets. Whereas congestion control prevents senders from overwhelming the network, flow control prevents the sender from overwhelming the receiver.
     * To ensure high throughput, web servers can keep a large number of TCP connections open, resulting in high memory usage. It can be expensive to have a large number of open connections between web server threads and say, a memcached server. 
        * Connection pooling can help in addition to switching to UDP where applicable.
        * A connection pool is a cache of database connections maintained so that the connections can be reused when future requests to the database are required. Connection pools are used to enhance the performance of executing commands on a database. Opening and maintaining a database connection for each user, especially requests made to a dynamic database-driven website application, is costly and wastes resources. In connection pooling, after a connection is created, it is placed in the pool and it is used again so that a new connection does not have to be established. If all the connections are being used, a new connection is made and is added to the pool. Connection pooling also cuts down on the amount of time a user must wait to establish a connection to the database.
     * TCP is useful for applications that require high reliability but are less time critical. Some examples include web servers, database info, SMTP, FTP, and SSH.
     * Use TCP over UDP when:
         * You need all of the data to arrive intact
         * You want to automatically make a best estimate use of the network throughput
   * User datagram protocol (UDP)
      * UDP is connectionless. Datagrams (analogous to packets) are guaranteed only at the datagram level. Datagrams might reach their destination out of order or not at all. UDP does not support congestion control. Without the guarantees that TCP support, UDP is generally more efficient.
      * Not all data may be present. If the data is incomplete, don't resend
      * UDP can broadcast, sending datagrams to all devices on the subnet. This is useful with DHCP because the client has not yet received an IP address, thus preventing a way for TCP to stream without the IP address.
      * UDP is less reliable but works well in real time use cases such as VoIP, video chat, streaming, and realtime multiplayer games.
      * Use UDP over TCP when:
          * You need the lowest latency
          * Late data is worse than loss of data
          * You want to implement your own error correction
      
4. HTTP vs. TCP
   * The short answer: TCP is a transport-layer protocol, and HTTP is an application-layer protocol that runs over TCP. 
   * let us start with the network level (Layer 3) which is home to the famous IPv4 protocol. Its main aim is to route the data packets from the source to the destination. It does not care about what data it is carrying and whether it was delivered to the destination or not. It will make its best effort and that's all.
   * We can talk to a computer somewhere around the world with Layer 3, but that computer is running lots of different programs. How should it know which one to deliver your message to? The transport layer (Layer 4) takes care of this, usually with port numbers. The two most popular transport layer protocols are TCP and UDP. TCP does a lot of interesting things to smooth over the rough spots of network-layer packet-switched communication like reordering packets, retransmitting lost packets, etc. UDP is more unreliable, but has less overhead.
   * how does the server know what page you want? How can you post a question or an answer? These are things that application-layer protocols handle. For web traffic, this is the HyperText Transfer Protocol (HTTP). There are thousands of application-layer protocols: SMTP, IMAP, and POP3 for email; XMPP, IRC, ICQ for chat; Telnet, SSH, RDP for remote administration; etc.
   * In reality, some protocols shim between various layers, or can work at multiple layers at once. TLS/SSL for instance provides encryption and session information between the network and transport layers. Above the application layer, Application Programming Interfaces (APIs) govern communication with web applications like Quora, Twitter, and Facebook.

5. What's the differences between PUT and PATCH?
   * The HTTP methods PATCH can be used to update partial resources. For instance, when you only need to update one field of the resource, PUTting a complete resource representation might be cumbersome and utilizes more bandwidth
   * Also, the PUT method is idempotent. PUTting the same data multiple times to the same resource, should not result in different resources, while POSTing to the same resource can result in the creation of multiple resources.
   * PATCH is neither safe nor idempotent. An API implementing PATCH must patch atomically. It MUST not be possible that resources are half-patched when requested by a GET.
   * In a PUT request, the enclosed entity is considered to be a modified version of the resource stored on the origin server, and the client is requesting that the stored version be replaced. With PATCH, however, the enclosed entity contains a set of instructions describing how a resource currently residing on the origin server should be modified to produce a new version. The PATCH method affects the resource identified by the Request-URI, and it also MAY have side effects on other resources; i.e., new resources may be created, or existing ones modified, by the application of a PATCH.















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
1. Ninginx
2. Apache
