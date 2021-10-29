
3. Check the ["OSI 7 LAYER MODEL"](http://www.escotal.com/osilayer.html)
   * The OSI, or Open System Interconnection, model defines a networking framework for implementing protocols in seven layers. Control is passed from one layer to the next, starting at the application layer in one station, proceeding to the bottom layer, over the channel to the next station and back up the hierarchy.

# Hypertext transfer protocol (HTTP)
      * HTTP is a method for encoding and transporting data between a client (browser) and a server. It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request. HTTP is self-contained, allowing requests and responses to flow through many intermediate routers and servers that perform load balancing, caching, encryption, and compression. This self‑contained design allows for the distributed nature of the Internet, where a request or response might pass through many intermediate routers and proxy servers.
      * Information is exchanged between clients and servers in the form of Hypertext documents (HTML). 
      * A basic HTTP request consists of a verb (method) and a resource (endpoint). Below are common HTTP verbs: GET, POST, PUT
      * HTTP is an application layer protocol relying on lower-level protocols such as TCP and UDP.
      * HTTP resources such as web servers are identified across the Internet using unique identifiers known as Uniform Resource Locators (URLs).

# Transmission control protocol (TCP)
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
      
## HTTP vs. TCP
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


