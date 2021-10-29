
3. Check the ["OSI 7 LAYER MODEL"](http://www.escotal.com/osilayer.html)
   * The OSI, or Open System Interconnection, model defines a networking framework for implementing protocols in seven layers. Control is passed from one layer to the next, starting at the application layer in one station, proceeding to the bottom layer, over the channel to the next station and back up the hierarchy.

# Hypertext transfer protocol (HTTP)
      * HTTP is a method for encoding and transporting data between a client (browser) and a server. It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request. HTTP is self-contained, allowing requests and responses to flow through many intermediate routers and servers that perform load balancing, caching, encryption, and compression. This self‑contained design allows for the distributed nature of the Internet, where a request or response might pass through many intermediate routers and proxy servers.
      * Information is exchanged between clients and servers in the form of Hypertext documents (HTML). 
      * A basic HTTP request consists of a verb (method) and a resource (endpoint). Below are common HTTP verbs: GET, POST, PUT
      * HTTP is an application layer protocol relying on lower-level protocols such as TCP and UDP.
      * HTTP resources such as web servers are identified across the Internet using unique identifiers known as Uniform Resource Locators (URLs).

# Transmission control protocol (TCP)
1. TCP is a connection-oriented protocol over an IP network. Connection is established and terminated using a handshake. All packets sent are guaranteed to reach the destination in the original order and without corruption through:
   * Sequence numbers and checksum fields for each packet
   * Acknowledgement packets and automatic retransmission
1. TCP connections are reliable and ordered. All data you send is guaranteed to arrive at the other side and in the order you wrote it. This imply the use of acknowledgement packets sent back to the sender. 
2. It’s also a stream protocol, so TCP automatically splits your data into packets and sends them over the network for you. 
3. When a file or message send it will get delivered unless connections fails. If connection lost, the server will request the lost part. There is no corruption while transferring a message.
4. If you have ever used a TCP socket, then you know it’s a reliable connection based protocol. This means you create a connection between two machines, then you exchange data much like you’re writing to a file on one side, and reading from a file on the other.
5. In telecommunications, a handshake is an automated process of negotiation between two participants (example "Alice and Bob") through the exchange of information that establishes the protocols of a communication link at the start of the communication, before full communication begins. The handshaking process usually takes place in order to establish rules for communication when a computer attempts to communicate with another device. Signals are usually exchanged between two devices to establish a communication link. For example, when a computer communicates with another device such as a modem, the two devices will signal each other that they are switched on and ready to work, as well as to agree to which protocols are being used.
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
6. Data is read as a “stream,” with nothing distinguishing where one packet ends and another begins. There may be multiple packets per read call.
7. 
#### Examples
Examples: World Wide Web (Apache TCP port 80), e-mail (SMTP TCP port 25 Postfix MTA), File Transfer Protocol (FTP port 21) and Secure Shell (OpenSSH port 22) etc.	

# User datagram protocol (UDP)
1. Instead of treating communications between computers like writing to files, what if we want to send and receive packets directly?
2. When you a send a data or message, you don’t know if it’ll get there, it could get lost on the way. There may be corruption while transferring a message.
3. Packets are sent individually and are guaranteed to be whole if they arrive. One packet per one read call.
4. UDP is connectionless. Datagrams (analogous to packets) are guaranteed only at the datagram level. Datagrams might reach their destination out of order or not at all. UDP does not support congestion control. Without the guarantees that TCP support, UDP is generally more efficient.
    * Not all data may be present. If the data is incomplete, don't resend
    * UDP can broadcast, sending datagrams to all devices on the subnet. This is useful with DHCP because the client has not yet received an IP address, thus preventing a way for TCP to stream without the IP address.
5. UDP is less reliable but works well in real time use cases such as VoIP, video chat, streaming, and realtime multiplayer games.
6. Use UDP over TCP when:
      * You need the lowest latency
      * Late data is worse than loss of data
      * You want to implement your own error correction
7. It's built on top of IP, but unlike TCP, instead of adding lots of features and complexity, UDP is a very thin layer over IP.
8. With UDP we can send a packet to a destination IP address (eg. 112.140.20.10) and port (say 52423), and it gets passed from computer to computer until it arrives at the destination or is lost along the way.
9. On the receiver side, we just sit there listening on a specific port (eg. 52423) and when a packet arrives from any computer (remember there are no connections!), we get notified of the address and port of the computer that sent the packet, the size of the packet, and can read the packet data.
10. Like IP, UDP is an unreliable protocol. In practice however, most packets that are sent will get through, but you’ll usually have around 1-5% packet loss, and occasionally you’ll get periods where no packets get through at all (remember there are lots of computers between you and your destination where things can go wrong…)
11. There is also no guarantee of ordering of packets with UDP. You could send 5 packets in order 1,2,3,4,5 and they could arrive completely out of order like 3,1,2,5,4. In practice, packets tend to arrive in order most of the time, but you cannot rely on this!
12. UDP also provides a 16 bit checksum, which in theory is meant to protect you from receiving invalid or truncated data, but you can’t even trust this, since 16 bits is just not enough protection when you are sending UDP packets rapidly over a long period of time. Statistically, you can’t even rely on this checksum and must add your own.
13. In certain situations UDP is used because it allows broadcast packet transmission. This is sometimes fundamental in cases like DHCP protocol, because the client machine hasn't still received an IP address (this is the DHCP negotiaton protocol purpose) and there won't be any way to establish a TCP stream without the IP address itself.


### Examples
Examples: Domain Name System (DNS UDP port 53), streaming media applications such as IPTV or movies, Voice over IP (VoIP), Trivial File Transfer Protocol (TFTP) and online multiplayer games etc

## TCP vs. UDP
Lets look at the properties of each:

### TCP:

Connection based
Guaranteed reliable and ordered
Automatically breaks up your data into packets for you
Makes sure it doesn't send data too fast for the internet connection to handle (flow control)
Easy to use, you just read and write data like its a file

### UDP:

No concept of connection, you have to code this yourself
No guarantee of reliability or ordering of packets, they may arrive out of order, be duplicated, or not arrive at all!
You have to manually break your data up into packets and send them
You have to make sure you don't send data too fast for your internet connection to handle
If a packet is lost, you need to devise some way to detect this, and resend that data if necessary
You can't even rely on the UDP checksum so you must add your own

###
1. Using TCP is the worst possible mistake you can make when developing a multiplayer game! 


### How TCP really works

1. TCP and UDP are both built on top of IP, but they are radically different. UDP behaves very much like the IP protocol underneath it, while TCP abstracts everything so it looks like you are reading and writing to a file, hiding all complexities of packets and unreliability from you.
2. So how does it do this?
   * Firstly, TCP is a stream protocol, so you just write bytes to a stream, and TCP makes sure that they get across to the other side. Since IP is built on packets, and TCP is built on top of IP, TCP must therefore break your stream of data up into packets. So, some internal TCP code queues up the data you send, then when enough data is pending the queue, it sends a packet to the other machine.
   * This can be a problem for multiplayer games if you are sending very small packets. What can happen here is that TCP may decide it’s not going to send data until you have buffered up enough data to make a reasonably sized packet to send over the network.
   * This is a problem because you want your client player input to get to the server as quickly as possible, if it is delayed or “clumped up” like TCP can do with small packets, the client’s user experience of the multiplayer game will be very poor. Game network updates will arrive late and infrequently, instead of on-time and frequently like we want.
   * TCP has an option to fix this behavior called TCP_NODELAY. This option instructs TCP not to wait around until enough data is queued up, but to flush any data you write to it immediately. This is referred to as disabling Nagle’s algorithm.
   * Unfortunately, even if you set this option TCP still has serious problems for multiplayer games and it all stems from how TCP handles lost and out of order packets to present you with the “illusion” of a reliable, ordered stream of data.

### How TCP implements reliability
1. Fundamentally TCP breaks down a stream of data into packets, sends these packets over unreliable IP, then takes the packets received on the other side and reconstructs the stream.
2. But what happens when a packet is lost?
3. What happens when packets arrive out of order or are duplicated?
1. Without going too much into the details of how TCP works because its super-complicated (please refer to TCP/IP Illustrated) in essence TCP sends out a packet, waits a while until it detects that packet was lost because it didn’t receive an ack (or acknowledgement), then resends the lost packet to the other machine. Duplicate packets are discarded on the receiver side, and out of order packets are resequenced so everything is reliable and in order.
1. The problem is that if we were to send our time critical game data over TCP, whenever a packet is dropped it has to stop and wait for that data to be resent. Yes, even if more recent data arrives, that new data gets put in a queue, and you cannot access it until that lost packet has been retransmitted. How long does it take to resend the packet?
1. Well, it’s going to take at least round trip latency for TCP to work out that data needs to be resent, but commonly it takes 2*RTT, and another one way trip from the sender to the receiver for the resent packet to get there. So if you have a 125ms ping, you’ll be waiting roughly 1/5th of a second for the packet data to be resent at best, and in worst case conditions you could be waiting up to half a second or more (consider what happens if the attempt to resend the packet fails to get through?). What happens if TCP decides the packet loss indicates network congestion and it backs off? Yes it actually does this. Fun times!
### Never use TCP for time critical data
1. The problem with using TCP for realtime games like FPS is that unlike web browsers, or email or most other applications, these multiplayer games have a real time requirement on packet delivery.
1. What this means is that for many parts of a game, for example player input and character positions, it really doesn’t matter what happened a second ago, the game only cares about the most recent data.
1. TCP was simply not designed with this in mind.
1. Consider a very simple example of a multiplayer game, some sort of action game like a shooter. You want to network this in a very simple way. Every frame you send the input from the client to the server (eg. keypresses, mouse input controller input), and each frame the server processes the input from each player, updates the simulation, then sends the current position of game objects back to the client for rendering.
1. So in our simple multiplayer game, whenever a packet is lost, everything has to stop and wait for that packet to be resent. On the client game objects stop receiving updates so they appear to be standing still, and on the server input stops getting through from the client, so the players cannot move or shoot. When the resent packet finally arrives, you receive this stale, out of date information that you don’t even care about! Plus, there are packets backed up in queue waiting for the resend which arrive at same time, so you have to process all of these packets in one frame. Everything is clumped up!
1. Unfortunately, there is nothing you can do to fix this behavior, it’s just the fundamental nature of TCP. This is just what it takes to make the unreliable, packet-based internet look like a reliable-ordered stream.
1. Thing is we don’t want a reliable ordered stream.
   * We want our data to get as quickly as possible from client to server without having to wait for lost data to be resent.
1. This is why you should never use TCP when networking time-critical data!

### Wait? Why can’t I use both UDP and TCP?
1. For realtime game data like player input and state, only the most recent data is relevant, but for other types of data, say perhaps a sequence of commands sent from one machine to another, reliability and ordering can be very important.
1. The temptation then is to use UDP for player input and state, and TCP for the reliable ordered data. If you’re sharp you’ve probably even worked out that you may have multiple “streams” of reliable ordered commands, maybe one about level loading, and another about AI. Perhaps you think to yourself, “Well, I’d really not want AI commands to stall out if a packet is lost containing a level loading command - they are completely unrelated!”. You are right, so you may be tempted to create one TCP socket for each stream of commands.

1. On the surface, this seems like a great idea. The problem is that since TCP and UDP are both built on top of IP, **the underlying packets sent by each protocol will affect each other**. Exactly how they affect each other is quite complicated and relates to how TCP performs reliability and flow control, but fundamentally you should remember that **TCP tends to induce packet loss in UDP packets**.
1. Also, it’s pretty complicated to mix UDP and TCP. If you mix UDP and TCP you lose a certain amount of control. Maybe you can implement reliability in a more efficient way that TCP does, better suited to your needs? Even if you need reliable-ordered data, it’s possible, provided that data is small relative to the available bandwidth to get that data across faster and more reliably that it would if you sent it over TCP. Plus, if you have to do NAT to enable home internet connections to talk to each other, having to do this NAT once for UDP and once for TCP (not even sure if this is possible…) is kind of painful.

### Conclusion
1. My recommendation is not only that you use UDP, but that you only use UDP for your game protocol. Don’t mix TCP and UDP! Instead, learn how to implement the specific features of TCP that you need inside your own custom UDP based protocol.
1. Of course, it is no problem to use HTTP to talk to some RESTful services while your game is running. I’m not saying you can’t do that. A few TCP connections running while your game is running isn’t going to bring everything down. The point is, don’t split your game protocol across UDP and TCP. Keep your game protocol running over UDP so you are fully in control of the data you send and receive and how reliability, ordering and congestion avoidance are implemented.




      
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


# IP or “internet protocol
1. Here there is no concept of connection, packets are simply passed from one computer to the next. You can visualize this process being somewhat like a hand-written note passed from one person to the next across a crowded room, eventually, reaching the person it’s addressed to, but only after passing through many hands.

There is also no guarantee that this note will actually reach the person it is intended for. The sender just passes the note along and hopes for the best, never knowing whether or not the note was received, unless the other person decides to write back!

1. since no one computer knows the exact sequence of computers to pass the packet along to so that it reaches its destination quickly. Sometimes IP passes along multiple copies of the same packet and these packets make their way to the destination via different paths, causing packets to arrive out of order and in duplicate.

This is because the internet is designed to be self-organizing and self-repairing, able to route around connectivity problems rather than relying on direct connections between computers.

1. 
