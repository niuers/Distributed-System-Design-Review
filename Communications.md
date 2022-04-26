
# OSI
1. Check the ["OSI 7 LAYER MODEL"](http://www.escotal.com/osilayer.html)
   * The OSI, or Open System Interconnection, model defines a networking framework for implementing protocols in seven layers. Control is passed from one layer to the next, starting at the application layer in one station, proceeding to the bottom layer, over the channel to the next station and back up the hierarchy.
1. At each layer there are standards that define how data is packaged and transported. Among other things, the standards define how to segment the stream of bits that constitute a request or response into discrete packages called protocol data units (PDUs). The standards also define the metadata added to each PDU in the form of a header; the metadata might specify the addresses of the origin and destination hosts, for example.
1. The Open System Interconnection (OSI) defines a model framework for implementing a standard format for communication, called a protocol, in these layers.
# SSL
The Secure Sockets Layer (SSL) protocol is primarily used to encrypt confidential data over insecure networks such as the internet. The SSL protocol establishes a secure connection between a client and the back-end server, and ensures that all the data passed between your client and your server is private and integral.


# Hypertext transfer protocol (HTTP)
1. On the software side, a web server includes several parts that control how web users access hosted files. At a minimum, this is an HTTP server. On the hardware side, a web server is a computer that stores web server software and a website's component files (for example, HTML documents, images, CSS stylesheets, and JavaScript files). A web server connects to the Internet and supports physical data interchange with other devices connected to the web.
1. A Protocol is a set of rules for communication between two computers. HTTP is a textual, stateless protocol. HTTP follows a classical client-server model (which means requests are initiated by the recipient, usually the Web browser), with a client opening a connection to make a request, then waiting until it receives a response. HTTP is a stateless protocol, meaning that the server does not keep any data (state) between two requests.
1. HTTP is a method for encoding and transporting data between a client (browser) and a server. It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request. HTTP is self-contained, allowing requests and responses to flow through many intermediate routers and servers that perform load balancing, caching, encryption, and compression. This self‑contained design allows for the distributed nature of the Internet, where a request or response might pass through many intermediate routers and proxy servers.
1. Information is exchanged between clients and servers in the form of Hypertext documents (HTML). 
1. A basic HTTP request consists of a verb (method) and a resource (endpoint). Below are common HTTP verbs: GET, POST, PUT
1. HTTP is an application layer protocol relying on lower-level protocols such as TCP and UDP.
1. HTTP resources such as web servers are identified across the Internet using unique identifiers known as Uniform Resource Locators (URLs).
2. Usually only clients make HTTP requests, and only to servers. Servers respond to a client's HTTP request. A server can also populate data into a client cache, in advance of it being requested, through a mechanism called a server push.
3. On a web server, the HTTP server is responsible for processing and answering incoming requests.
4. HTTP is stateless, but not sessionless
  1. HTTP is stateless: there is no link between two requests being successively carried out on the same connection. This immediately has the prospect of being problematic for users attempting to interact with certain pages coherently, for example, using e-commerce shopping baskets. 
  2. But while the core of HTTP itself is stateless, HTTP cookies allow the use of stateful sessions. Using header extensibility, HTTP Cookies are added to the workflow, allowing session creation on each HTTP request to share the same context, or the same state.
5. HTTP and connections
  1. A connection is controlled at the transport layer, and therefore fundamentally out of scope for HTTP. HTTP doesn't require the underlying transport protocol to be connection-based; it only requires it to be reliable, or not lose messages (at minimum, presenting an error in such cases). Among the two most common transport protocols on the Internet, TCP is reliable and UDP isn't. HTTP therefore relies on the TCP standard, which is connection-based.
  2. Before a client and server can exchange an HTTP request/response pair, they must establish a TCP connection, a process which requires several round-trips. The default behavior of HTTP/1.0 is to open a separate TCP connection for each HTTP request/response pair. This is less efficient than sharing a single TCP connection when multiple requests are sent in close succession.
  3. In order to mitigate this flaw, HTTP/1.1 introduced pipelining (which proved difficult to implement) and persistent connections: the underlying TCP connection can be partially controlled using the Connection header. HTTP/2 went a step further by multiplexing messages over a single connection, helping keep the connection warm and more efficient.
6. What can be controlled by HTTP
  1. Caching: How documents are cached can be controlled by HTTP. The server can instruct proxies and clients about what to cache and for how long. The client can instruct intermediate cache proxies to ignore the stored document.
  2. Relaxing the origin constraint: To prevent snooping and other privacy invasions, Web browsers enforce strict separation between Web sites. Only pages from the same origin can access all the information of a Web page. Though such a constraint is a burden to the server, HTTP headers can relax this strict separation on the server side, allowing a document to become a patchwork of information sourced from different domains; there could even be security-related reasons to do so.
  3. Authentication: Some pages may be protected so that only specific users can access them. Basic authentication may be provided by HTTP, either using the WWW-Authenticate and similar headers, or by setting a specific session using HTTP cookies.
  4. Proxy and tunneling: Servers or clients are often located on intranets and hide their true IP address from other computers. HTTP requests then go through proxies to cross this network barrier. Not all proxies are HTTP proxies. The SOCKS protocol, for example, operates at a lower level. Other protocols, like ftp, can be handled by these proxies.
  5. Sessions: Using HTTP cookies allows you to link requests with the state of the server. This creates sessions, despite basic HTTP being a state-less protocol. This is useful not only for e-commerce shopping baskets, but also for any site allowing user configuration of the output.

## HTTP flow
When a client wants to communicate with a server, either the final server or an intermediate proxy, it performs the following steps:

1. Open a TCP connection: The TCP connection is used to send a request, or several, and receive an answer. The client may open a new connection, reuse an existing connection, or open several TCP connections to the servers.
2. Send an HTTP message: HTTP messages (before HTTP/2) are human-readable. With HTTP/2, these simple messages are encapsulated in frames, making them impossible to read directly, but the principle remains the same. For example:
```
GET / HTTP/1.1
Host: developer.mozilla.org
Accept-Language: fr
```
3. Read the response sent by the server, such as:
```
HTTP/1.1 200 OK
Date: Sat, 09 Oct 2010 14:28:02 GMT
Server: Apache

Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
ETag: "51142bc1-7449-479b075b2891b"
Accept-Ranges: bytes
Content-Length: 29769
Content-Type: text/html

<!DOCTYPE html... (here come the 29769 bytes of the requested web page)
```
4. Close or reuse the connection for further requests.
  5. If HTTP pipelining is activated, several requests can be sent without waiting for the first response to be fully received. HTTP pipelining has proven difficult to implement in existing networks, where old pieces of software coexist with modern versions. HTTP pipelining has been superseded in HTTP/2 with more robust multiplexing requests within a frame.

## APIs based on HTTP
1. The most commonly used API based on HTTP is the XMLHttpRequest API, which can be used to exchange data between a user agent and a server. 
2. The modern Fetch API provides the same features with a more powerful and flexible feature set.
3. Another API, server-sent events, is a one-way service that allows a server to send events to the client, using HTTP as a transport mechanism. Using the EventSource interface, the client opens a connection and establishes event handlers. The client browser automatically converts the messages that arrive on the HTTP stream into appropriate Event objects, delivering them to the event handlers that have been registered for the events' type if known, or to the onmessage event handler if no type-specific event handler was established.


## Components of HTTP-based systems
1. Between the client and the server there are numerous entities, collectively called proxies, which perform different operations and act as gateways or caches, for example.
2. The user-agent is any tool that acts on behalf of the user. This role is primarily performed by the Web browser, but it may also be performed by programs used by engineers and Web developers to debug their applications.
3. The Web server
4. Proxies
  1. Between the Web browser and the server, numerous computers and machines relay the HTTP messages. Due to the layered structure of the Web stack, most of these operate at the transport, network or physical levels, becoming transparent at the HTTP layer and potentially having a significant impact on performance. 
  2. Those operating at the application layers are generally called proxies. These can be transparent, forwarding on the requests they receive without altering them in any way, or non-transparent, in which case they will change the request in some way before passing it along to the server. Proxies may perform numerous functions:
    3. caching (the cache can be public or private, like the browser cache)
    4. filtering (like an antivirus scan or parental controls)
    5. load balancing (to allow multiple servers to serve different requests)
    6. authentication (to control access to different resources)
    7. logging (allowing the storage of historical information)
8. 

## HTTP caching
1. These can be grouped into two main categories: shared and private caches. A shared cache is a cache that stores responses for reuse by more than one user. A private cache is dedicated to a single user. This page will mostly talk about browser and proxy caches, but there are also gateway caches, CDN, reverse proxy caches and load balancers that are deployed on web servers for better reliability, performance and scaling of web sites and web applications.
2. Private browser caches
  1. A private cache is dedicated to a single user. You may have seen "caching" in your browser's settings already. A browser cache holds all documents the user downloads via HTTP. This cache is used to make visited documents available for back/forward navigation, saving, viewing-as-source, etc. without requiring an additional trip to the server. It also improves offline browsing of cached content.
3. Shared proxy caches
  1. A shared cache is a cache that stores responses to be reused by more than one user. For example, an Internet Service Provider (ISP) or your company might have set up a web proxy as part of its local network infrastructure to serve many users so that popular resources are reused a number of times, reducing network traffic and latency.
4. Common forms of caching entries are:
  5. Successful results of a retrieval request: a 200 (OK) response to a GET request containing a resource like HTML documents, images or files.
  6. Permanent redirects: a 301 (Moved Permanently) response.
  7. Error responses: a 404 (Not Found) result page.
  8. Incomplete results: a 206 (Partial Content) response.
  9. Responses other than GET if something suitable for use as a cache key is defined.
5. Controlling caching
  1. The Cache-Control header
    1. The Cache-Control HTTP/1.1 general-header field is used to specify directives for caching mechanisms in both requests and responses. Use this header to define your caching policies with the variety of directives it provides.
  2. Expiration
    3. The most important directive here is max-age=<seconds>, which is the maximum amount of time in which a resource will be considered fresh. This directive is relative to the time that the response was sent by the server, and overrides the Expires header (if set).
  3. Note that a stale resource is not evicted or ignored; when the cache receives a request for a stale resource, it forwards this request with an If-None-Match header to check if it is in fact still fresh. If so, the server returns a 304 (Not Modified) header without sending the body of the requested resource, saving some bandwidth.

### Cache validation
1. When a cached resource's expiration time has been reached, the resource is either validated or fetched again. Validation can only occur if the server provided either a strong validator or a weak validator.
2. Revalidation is triggered when the user presses the Reload button. It is also triggered during normal browsing if the cached response includes the "Cache-Control: must-revalidate" header. You can also use the cache validation preferences in the Advanced->Cache preferences panel, which offers the option to force a validation each time a resource is loaded.

## Using HTTP cookies
1. An HTTP cookie (web cookie, browser cookie) is a small piece of data that a server sends to a user's web browser. The browser may store the cookie and send it back to the same server with later requests. Typically, an HTTP cookie is used to tell if two requests come from the same browser—keeping a user logged in, for example. It remembers stateful information for the stateless HTTP protocol.
2. Cookies are mainly used for three purposes:
  1. Session management: Logins, shopping carts, game scores, or anything else the server should remember
  2. Personalization: User preferences, themes, and other settings
  3. Tracking: Recording and analyzing user behavior

3. Cookies were once used for general client-side storage. While this made sense when they were the only way to store data on the client, modern storage APIs are now recommended. Cookies are sent with every request, so they can worsen performance (especially for mobile data connections). Modern APIs for client storage are the Web Storage API (localStorage and sessionStorage) and IndexedDB.

4. After receiving an HTTP request, a server can send one or more Set-Cookie headers with the response. The browser usually stores the cookie and sends it with requests made to the same server inside a Cookie HTTP header. You can specify an expiration date or time period after which the cookie shouldn't be sent. You can also set additional restrictions to a specific domain and path to limit where the cookie is sent.
5. Then, with every subsequent request to the server, the browser sends all previously stored cookies back to the server using the Cookie header

## A typical HTTP session
1. In client-server protocols, like HTTP, sessions consist of three phases:
  1. The client establishes a TCP connection (or the appropriate connection if the transport layer is not TCP).
  2. The client sends its request, and waits for the answer.
  3. The server processes the request, sending back its answer, providing a status code and appropriate data.
2. As of HTTP/1.1, the connection is no longer closed after completing the third phase, and the client is now granted a further request: this means the second and third phases can now be performed any number of times.
  
3. Establishing a connection
  1. In client-server protocols, it is the client which establishes the connection. Opening a connection in HTTP means initiating a connection in the underlying transport layer, usually this is TCP.
  2. With TCP the default port, for an HTTP server on a computer, is port 80. Other ports can also be used, like 8000 or 8080. The URL of a page to fetch contains both the domain name, and the port number, though the latter can be omitted if it is 80. See Identifying resources on the Web for more details.
  3. The client-server model does not allow the server to send data to the client without an explicit request for it. To work around this problem, web developers use several techniques: ping the server periodically via the XMLHTTPRequest, fetch() APIs, using the WebSockets API, or similar protocols.

4. Sending a client request
  1. Once the connection is established, the user-agent can send the request (a user-agent is typically a web browser, but can be anything else, a crawler, for example). A client request consists of text directives, separated by CRLF (carriage return, followed by line feed), divided into three blocks:
    1. The first line contains a request method followed by its parameters:
      * the path of the document, as an absolute URL without the protocol or domain name
      * the HTTP protocol version
    2. Subsequent lines represent an HTTP header, giving the server information about what type of data is appropriate (for example, what language, what MIME types), or other data altering its behavior (for example, not sending an answer if it is already cached). These HTTP headers form a block which ends with an empty line.
    3. The final block is an optional data block, which may contain further data mainly used by the POST method.

5. Structure of a server response
After the connected agent has sent its request, the web server processes it, and ultimately returns a response. Similar to a client request, a server response is formed of text directives, separated by CRLF, though divided into three blocks:

The first line, the status line, consists of an acknowledgment of the HTTP version used, followed by a response status code (and its brief meaning in human-readable text).
Subsequent lines represent specific HTTP headers, giving the client information about the data sent (for example, type, data size, compression algorithm used, hints about caching). Similarly to the block of HTTP headers for a client request, these HTTP headers form a block ending with an empty line.
The final block is a data block, which contains the optional data.

## Connection management in HTTP/1.x
1. Connection management is a key topic in HTTP: opening and maintaining connections largely impacts the performance of Web sites and Web applications. In HTTP/1.x, there are several models: short-lived connections, persistent connections, and HTTP pipelining.
2. Two newer models were created in HTTP/1.1. The persistent-connection model keeps connections opened between successive requests, reducing the time needed to open new connections. The HTTP pipelining model goes one step further, by sending several successive requests without even waiting for an answer, reducing much of the latency in the network.
3. HTTP/2 adds additional models for connection management.
4. A related topic is the concept of HTTP connection upgrades, wherein an HTTP/1.1 connection is upgraded to a different protocol, such as TLS/1.0, WebSocket, or even HTTP/2 in cleartext.
  
### Short-Lived
1. The original model of HTTP, and the default one in HTTP/1.0, is short-lived connections. 
1. short-lived: a new one created each time a request needed sending, and closed once the answer had been received. this means a TCP handshake happens before each HTTP request, and these are serialized.
2. The TCP handshake itself is time-consuming, but a TCP connection adapts to its load, becoming more efficient with more sustained (or warm) connections. Short-lived connections do not make use of this efficiency feature of TCP, and performance degrades from optimum by persisting to transmit over a new, cold connection.
3. This model is the default model used in HTTP/1.0 (if there is no Connection header, or if its value is set to close). In HTTP/1.1, this model is only used when the Connection header is sent with a value of close.

2. This simple model held an innate limitation on performance: opening each TCP connection is a resource-consuming operation. Several messages must be exchanged between the client and the server. Network latency and bandwidth affect performance when a request needs sending. Modern Web pages require many requests (a dozen or more) to serve the amount of information needed, proving this earlier model inefficient.
5. Unless dealing with a very old system, which doesn't support a persistent connection, there is no compelling reason to use this model.

### Persistent Connections
1. Short-lived connections have two major hitches: the time taken to establish a new connection is significant, and performance of the underlying TCP connection gets better only when this connection has been in use for some time (warm connection). 
2. Alternatively this may be called a keep-alive connection.
3. A persistent connection is one which remains open for a period of time, and can be reused for several requests, saving the need for a new TCP handshake, and utilizing TCP's performance enhancing capabilities. This connection will not stay open forever: idle connections are closed after some time (a server may use the Keep-Alive header to specify a minimum time the connection should be kept open).
4. Persistent connections also have drawbacks; even when idling they consume server resources, and under heavy load, DoS attacks can be conducted. In such cases, using non-persistent connections, which are closed as soon as they are idle, can provide better performance
5. In HTTP/1.1, persistence is the default, and the header is no longer needed (but it is often added as a defensive measure against cases requiring a fallback to HTTP/1.0).

### HTTP Pipelining
1. Pipelining is complex to implement correctly
2. By default, HTTP requests are issued sequentially. The next request is only issued once the response to the current request has been received. As they are affected by network latencies and bandwidth limitations, this can result in significant delay before the next request is seen by the server.
3. Pipelining is the process to send successive requests, over the same persistent connection, without waiting for the answer. This avoids latency of the connection. Theoretically, performance could also be improved if two HTTP requests were to be packed into the same TCP message.
4. Not all types of HTTP requests can be pipelined: only idempotent methods, that is GET, HEAD, PUT and DELETE, can be replayed safely. Should a failure happen, the pipeline content can be repeated.
5. 


  
  


  
  






# Difference between connections and connectionless
1. IP is a connectionless protocol. That means that there is no relationship between the packets sent – they are all individuals.
  2. IP is known as a best-effort protocol, but it does not guarantee packet delivery. Packets may be lost, corrupted, out of order, or duplicated. IP does nothing to correct those conditions. TCP does.
3. TCP makes a connection-oriented protocol on top of connectionless IP because the packets are related. Mostly they must be delivered in a certain order.
  4. Now actually, TCP takes an entire message and splits it into packets for IP to transfer. The message will be reassembled in the correct order. But messages may be delivered out of order. Thus an application protocol must ensure the messages are also delivered in correct order. Mostly that will be handled in middleware (the session layer in OSI).
  5. Also related to connections are sequence numbers – these implement the guarantees of TCP. Sequence numbers make sure all packets are received in the correct order and discard duplicates. Connectionless protocols have no way of doing this.
  6. But note a session-layer protocol might also be needed to ensure correct order of messages, since TCP only guarantees that within a message.
5. Note that UDP is a very thin transport layer over TCP. While IP does machine-to-machine delivery, transport layer protocols are process-to-process. They add ports to identify the receiving process on the receiving machine.
6. Another important point is that each network layer is independent in its connection-oriented or connectionless aspect. Connectionless protocols may be implemented over connection-oriented and as we have seen, connection-oriented TCP can be implemented over connectionless IP.



# Transmission control protocol (TCP)
1. TCP is a connection-oriented protocol over an IP network. Connection is established and terminated using a handshake. All packets sent are guaranteed to reach the destination in the original order and without corruption through:
   * Sequence numbers and checksum fields for each packet
   * Acknowledgement packets and automatic retransmission
1. Each application is assigned a unique TCP port number to enable delivery to the correct application on hosts where many applications are running.
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

## TCP Streams
1. It’s a continuous byte stream. You write data in-order, and it’s received in-order. The lower levels of the stack handle packetizing, sending, receiving, reordering, reassembling, etc. And, because it’s a reliable transport, it even handles retransmitting data if portions got lost.
## Reliable Datagrams
1. We want to send reliable datagrams (as compared to UDP) so we need use shared context between sender and receiver to build a reliable transport over an unreliable transport. This "shared context" is called a "connection".
2. The definition of a connection, as I understand it:
  1. Data is transmitted and received in-order, from the application’s view.
  1. There’s a notion of “session”, with a set-up and tear-down phase.
3. In my strawman reliable datagram example, all those pieces are present:
  1. The datagram’s contents are in-order even if the datagram is packetized.
  1. The sender and receiver must perform some minimal handshake to establish a mutual context, so that reordering, reassembly, and retransmits can happen.
  1. There’s even a teardown phase, where the sender learns everything it sent was received successfully, so it can retire its context.
And thus, the shared context is what embodies the working details of the connection.
4. The notion of connection resides in the shared context both ends of the link have to maintain so they can remain synchronized across multiple packets, and abstract away the packetization, reordering, and lossiness of the network, for the duration of a “session,” whether that’s a single datagram or a long-lived stream.
5. And that’s the point. The connection exists in the notion of a “session” and “shared context” that allow the sender and receiver to ferry a large block of data coherently from one side to the other.




#### Examples
Examples: World Wide Web (Apache TCP port 80), e-mail (SMTP TCP port 25 Postfix MTA), File Transfer Protocol (FTP port 21) and Secure Shell (OpenSSH port 22) etc.	

# User datagram protocol (UDP)

1. UDP is connectionless. It’s stateless, really. You package up your datagram, and lob it onto the network. If it arrives at the destination, and an app is listening, then it’s delivered to the listening app.
2. If I send a UDP packet out to the network, I have no idea whether the remote side received it. And if the remote side sends me a reply, they have no idea whether I received that reply.There’s nothing in the protocol to tell you anything about delivery. If the UDP packet makes it to the other side, and the other side is listening, then the other side gets to hear about it.
3. Instead of treating communications between computers like writing to files, what if we want to send and receive packets directly?
4. When you a send a data or message, you don’t know if it’ll get there, it could get lost on the way. There may be corruption while transferring a message.
5. Packets are sent individually and are guaranteed to be whole if they arrive. One packet per one read call.
6. UDP is connectionless. Datagrams (analogous to packets) are guaranteed only at the datagram level. Datagrams might reach their destination out of order or not at all. UDP does not support congestion control. Without the guarantees that TCP support, UDP is generally more efficient.
    * Not all data may be present. If the data is incomplete, don't resend
    * UDP can broadcast, sending datagrams to all devices on the subnet. This is useful with DHCP because the client has not yet received an IP address, thus preventing a way for TCP to stream without the IP address.
7. UDP is less reliable but works well in real time use cases such as VoIP, video chat, streaming, and realtime multiplayer games.
8. Use UDP over TCP when:
      * You need the lowest latency
      * Late data is worse than loss of data
      * You want to implement your own error correction
9. It's built on top of IP, but unlike TCP, instead of adding lots of features and complexity, UDP is a very thin layer over IP.
10. With UDP we can send a packet to a destination IP address (eg. 112.140.20.10) and port (say 52423), and it gets passed from computer to computer until it arrives at the destination or is lost along the way.
11. On the receiver side, we just sit there listening on a specific port (eg. 52423) and when a packet arrives from any computer (remember there are no connections!), we get notified of the address and port of the computer that sent the packet, the size of the packet, and can read the packet data.
12. Like IP, UDP is an unreliable protocol. In practice however, most packets that are sent will get through, but you’ll usually have around 1-5% packet loss, and occasionally you’ll get periods where no packets get through at all (remember there are lots of computers between you and your destination where things can go wrong…)
13. There is also no guarantee of ordering of packets with UDP. You could send 5 packets in order 1,2,3,4,5 and they could arrive completely out of order like 3,1,2,5,4. In practice, packets tend to arrive in order most of the time, but you cannot rely on this!
14. UDP also provides a 16 bit checksum, which in theory is meant to protect you from receiving invalid or truncated data, but you can’t even trust this, since 16 bits is just not enough protection when you are sending UDP packets rapidly over a long period of time. Statistically, you can’t even rely on this checksum and must add your own.
15. In certain situations UDP is used because it allows broadcast packet transmission. This is sometimes fundamental in cases like DHCP protocol, because the client machine hasn't still received an IP address (this is the DHCP negotiaton protocol purpose) and there won't be any way to establish a TCP stream without the IP address itself.
## IP fragmentation
1. A UDP datagram could get fragmented at the IP layer. The IP layer can perform some limited reordering and reassembly, but if any fragment gets lost, the whole IP packet gets lost, and thus the whole datagram gets lost. IP reassembly is a best-effort deal. Usually you want to send UDP with Don’t Fragment set to 1.




### Examples
Examples: Domain Name System (DNS UDP port 53), streaming media applications such as IPTV or movies, Voice over IP (VoIP), Trivial File Transfer Protocol (TFTP) and online multiplayer games etc

## TCP vs. UDP

### TCP:
1. Connection based
1. Guaranteed reliable and ordered
1. Automatically breaks up your data into packets for you
1. Makes sure it doesn't send data too fast for the internet connection to handle (flow control)
1. Easy to use, you just read and write data like its a file

### UDP:

1. No concept of connection, you have to code this yourself
1. No guarantee of reliability or ordering of packets, they may arrive out of order, be duplicated, or not arrive at all!
1. You have to manually break your data up into packets and send them
1. You have to make sure you don't send data too fast for your internet connection to handle
1. If a packet is lost, you need to devise some way to detect this, and resend that data if necessary
1. You can't even rely on the UDP checksum so you must add your own
2. Some latency-sensitive applications such as videoconferencing and Voice over IP use UDP
   * UDP does not perform flow control and does not retransmit lost packets, it avoids some of the reasons for variable network delays (although it is still suscepti‐ ble to switch queues and scheduling delays)
   * UDP is a good choice in situations where delayed data is worthless.

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
1. Internet Protocol (IP) operates at the internetwork layer (Layer 3). Its PDUs are called packets, and IP is responsible for delivering them from a origin host to a destination host, usually across the boundaries between the multiple smaller networks that make up the Internet. Each device that is directly connected to the Internet has a unique IP address, which is used to locate the device as the recipient of packets.
1. Here there is no concept of connection, packets are simply passed from one computer to the next. You can visualize this process being somewhat like a hand-written note passed from one person to the next across a crowded room, eventually, reaching the person it’s addressed to, but only after passing through many hands.

There is also no guarantee that this note will actually reach the person it is intended for. The sender just passes the note along and hopes for the best, never knowing whether or not the note was received, unless the other person decides to write back!

1. since no one computer knows the exact sequence of computers to pass the packet along to so that it reaches its destination quickly. Sometimes IP passes along multiple copies of the same packet and these packets make their way to the destination via different paths, causing packets to arrive out of order and in duplicate.

This is because the internet is designed to be self-organizing and self-repairing, able to route around connectivity problems rather than relying on direct connections between computers.

1. 
# Remote procedure call (RPC)
1. RPC, a session layer (or application layer in TCP/IP layer model) protocol, is an inter-process communication that allows a computer program to cause a subroutine or procedure to execute in another address space (commonly on another computer on a shared network), without the programmer explicitly coding the details for this remote interaction. That is, the programmer writes essentially the same code whether the subroutine is local to the executing program, or remote. In an Object-Oriented Programming context, RPC is also called remote invocation or remote method invocation (RMI).

1. In an RPC, a client causes a procedure to execute on a different address space, usually a remote server. The procedure is coded as if it were a local procedure call, abstracting away the details of how to communicate with the server from the client program. Remote calls are usually slower and less reliable than local calls so it is helpful to distinguish RPC calls from local calls. Popular RPC frameworks include Protobuf, Thrift, and Avro.
   * Stub procedure: a local procedure that marshals the procedure identifier and the arguments into a request message, and then to send via its communication module to the server. When the reply message arrives, it unmarshals the results.
   * Google Protobuf, 
   * Facebook Thrift: User case: Hbase/Cassandra/Hypertable/Scrib/
   * Apache Avro: Avro is heavily used in the hadoop ecosystem and based on dynamic schemas in Json. It features dynamic typing, untagged data, and no manually-assigned field IDs.

1. Generally speaking, RPC is internally used by many tech companies for performance issues, but it is rather hard to debug and not flexible. So for public APIs, we tend to use HTTP APIs, and are usually following the RESTful style.

## Apache/FB Thrift
1.  Thrift, a cross-language framework for handling RPC, including serialization/deserialization, protocol transport, and server creation.
1. For example, one issue we ran into was that internal service owners were constantly re-inventing the same features again and again — such as transport compression, authentication, and counters — to track the health of their servers. Engineers were also spending a lot of time trying to eke out more performance from their services.
2. Over time, we found that parallel processing of requests from the same client and out-of-order responses solved many of the performance issues. The benefits of the former are obvious; the latter helps avoid application-level, head-of-line blocking.
3. Head-of-line blocking in computer networking is a performance-limiting phenomenon that occurs when a line of packets is held up by the first packet. Examples include input buffered network switches, out-of-order delivery and multiple requests in HTTP pipelining.
4. Service requests that go between data centers are dynamically compressed based on the size of the message, while in-rack requests skip compression (and thus avoid the CPU hit).
5. To improve asynchronous workload performance, we updated the base Thrift transport to be folly’s IOBuf class, a chained memory buffer with views similar to BSD’s mbuf or Linux’s sk_buff.
   * To reduce the performance impact of allocating new buffers, we allocate constant-sized buffers from JEMalloc to hit the thread-local buffer cache as often as possible. Hitting the thread-local cache was an impressive performance improvement — for the average Thrift server, it’s just as fast as reusing or pooling buffers, without any of the complicated code. These buffers are then chained together to become as large as needed, and freed when not needed, preventing some memory issues seen in previous Thrift servers where memory was pooled indefinitely
6. To allow for per-request attributes and features, we introduced a new THeader protocol and transport. The THeader format is very similar to HTTP headers — each request passes along headers that the server can interpret. With some clever programming, it was possible to make the THeader format backward compatible with all the previous Thrift transports and protocols.
7. Apache Thrift has become a ubiquitous piece of software for backend and embedded systems operating at scale.

### RPC is a request-response protocol:
1. Client program - Calls the client stub procedure. The parameters are pushed onto the stack like a local procedure call.
1. Client stub procedure - Marshals (packs) procedure id and arguments into a request message.
1. Client communication module - OS sends the message from the client to the server.
1. Server communication module - OS passes the incoming packets to the server stub procedure.
1. Server stub procedure - Unmarshalls the results, calls the server procedure matching the procedure id and passes the given arguments.
1. The server response repeats the steps above in reverse order.

### Sample RPC calls:
```
GET /someoperation?data=anId

POST /anotheroperation
{
  "data":"anId";
  "anotherdata": "another value"
}
```
1. RPC is focused on exposing behaviors. RPCs are often used for performance reasons with internal communications, as you can hand-craft native calls to better fit your use cases.

### Choose a native library (aka SDK) when:
1. You know your target platform.
1. You want to control how your "logic" is accessed.
1. You want to control how error control happens off your library.
1. Performance and end user experience is your primary concern.
1. HTTP APIs following REST tend to be used more often for public APIs.

### Disadvantage(s): RPC
1. RPC clients become tightly coupled to the service implementation.
1. A new API must be defined for every new operation or use case.
1. It can be difficult to debug RPC.
1. You might not be able to leverage existing technologies out of the box. For example, it might require additional effort to ensure RPC calls are properly cached on caching servers such as Squid.

### Why REST for internal use and not RPC?
1. RPC is a nearly transparent mechanism that makes communication between two processes nearly painless- in theory. I can see the appeal of REST if you want to provide an API to clients beyond your control- everyone can do REST while RPC is more complex (if you don't have a matching RPC library for your platform of choice, you're probably fucked- but if you have control over front *and* backend, you should be able to pick a well-supported mechanism on both ends).
2. 

# Representational State Transfer (REST)
1. REST is an architectural style enforcing a client/server model where the client acts on a set of resources managed by the server. The server provides a representation of resources and actions that can either manipulate or get a new representation of resources. All communication must be stateless and cacheable.

### There are four qualities of a RESTful interface:
1. Identify resources (URI in HTTP) - use the same URI regardless of any operation.
1. Change with representations (Verbs in HTTP) - use verbs, headers, and body.
1. Self-descriptive error message (status response in HTTP) - Use status codes, don't reinvent the wheel.
1. HATEOAS (HTML interface for HTTP) - your web service should be fully accessible in a browser.

### Sample REST calls:
```
GET /someresources/anId

PUT /someresources/anId
{"anotherdata": "another value"}
```

1. REST is focused on exposing data. It minimizes the coupling between client/server and is often used for public HTTP APIs. REST uses a more generic and uniform method of exposing resources through URIs, representation through headers, and actions through verbs such as GET, POST, PUT, DELETE, and PATCH. **Being stateless, REST is great for horizontal scaling and partitioning.**

### Disadvantage(s): REST
1. With REST being focused on exposing data, it might not be a good fit if resources are not naturally organized or accessed in a simple hierarchy. For example, returning all updated records from the past hour matching a particular set of events is not easily expressed as a path. With REST, it is likely to be implemented with a combination of URI path, query parameters, and possibly the request body.
1. REST typically relies on a few verbs (GET, POST, PUT, DELETE, and PATCH) which sometimes doesn't fit your use case. For example, moving expired documents to the archive folder might not cleanly fit within these verbs.
1. Fetching complicated resources with nested hierarchies requires multiple round trips between the client and server to render single views, e.g. fetching content of a blog entry and the comments on that entry. For mobile applications operating in variable network conditions, these multiple roundtrips are highly undesirable.
1. Over time, more fields might be added to an API response and older clients will receive all new data fields, even those that they do not need, as a result, it bloats the payload size and leads to larger latencies.

## RPC and REST calls comparison
1. why should I considered REST’s request style (resource oriented) better than RPC’s (operation oriented)? Is RPC’s request style so evil? Is REST’s the panacea?
1. Both RPC and REST use HTTP protocol which is a request/response protocol. A basic HTTP request consists of:
   * A verb (or method)
   * A resource (or endpoint)
1. Each HTTP verb:
   * Has a meaning
   * Is idempotent or not: A request method is considered “idempotent” if the intended effect on the server of multiple identical requests with that method is the same as the effect for a single such request.
      * GET, PUT, DELETE
   * Is safe or not: Request methods are considered “safe” if their defined semantics are essentially read-only.
      * GET
   * Is cacheable or not
      * GET
      * POST/PATCH(Only cacheable if response contains explicit freshness information)

#### RPC
1. when I talk about RPC I talk about WYGOPIAO: What You GET Or POST Is An Operation
2. With this type of RPC, you expose operations to manipulate data through HTTP as a transport protocol. As far as I know, there are no particular rules for this style but generally:
   * The endpoint contains the name of the operation you want to invoke.
   * This type of API generally only uses GET and POST HTTP verbs.
#### REST
1. With a REST API you expose data as resources that you manipulate through HTTP protocol using the right HTTP verb
2. The endpoint contains the resource you manipulate. Many use the CRUD analogy to explain REST requests principles. The HTTP verb indicates what you want to do (Create/Read/Update/Delete) with that resource.

3. REST is an architectural approach and means that a RESTful system has the following properties:
   * It is client/server: the business logic is decoupled from presentation. So you can change one without impacting the other. The cons, it adds negligible latency, but who cares, the web is the platform and everything is client/server.
   * It is stateless: All messages exchanged between client and server has all the context needed to know what to do with the message. This visibility has several benefits: 
      * you can route a message where ever you want depending on its contents and any server can service a request. So you an just scale your server by creating several instances of it.
      * You dont need to send all messages from the same client or user to the same server. And if you want to supertune the backend, you can rout messages to different server depending on the message. For example, having a CPU intensive request to one server and a memory intensive one to another. 
      * The cons: the client is sending all messages with redundant information. This adds bandwidth and again, negligible latency.
   * It is cacheable, so if you were worried about latency you save bandwidth cacheing responses from the server.
   * It has a uniform interface based on hypermedia (you know, that HATEOAS thing). The great thing about this is that you can greatly improve decoupling between client and server. If server responses contain hypermedia to all referenced resources and available actions within the context of the last request, the client does not need to know much about the server but an entry point and a few conventions about the hypermedia. Properly implemented, you could change many things in the server side without rewriting a single line in the client.
   * And finally, it's layered, like an onion. You can put several layers of components between client and server, for routing purposes, load balancing, cacheing or whatever you need. Of course it adds latency but also lots of flexibility. And if you change a layer, only the previous layer could be impacted, so propagation of change effects is limited.
1. In general the only cons are related to latency in request processing times and bandwidth usage. But its a great general purpose architecture that provides:
   * great flexibility
   * lower maintenance costs
   * high scalability
   * simplicity

#### Caching
1. I’ve often seen (http) caching used as a killer reason to choose REST over RPC.
But after reading HTTP RFCs, I do not agree with this argument (maybe I missed something). Of course if your RPC API only use POST for all requests, caching may be a little tricky to handle (but not impossible). If you use GET and POST wisely, your RPC API will be able to obtain the same level of cacheability as a REST API.

#### Predictability and semantic
1. With RPC the semantic relies (mostly) on the endpoint and there are no global shared understanding of its meaning. 
2. With RPC you rely on your human interpretation of the endpoint’s meaning to understand what it does but you can therefore have a fine human readable description of what is happening when you call this endpoint.
3. With REST the semantic relies (mostly) on the HTTP verb. The verb’s semantic is globally shared.
4. REST is more predictable than RPC as it relies on the shared semantic of HTTP verbs. You don’t know what happen exactly but you have a general idea of what you do.
### RPC Problems
1. Focus on language (fit the distributed system to the language, not the other way around)
1. "Make it look local" (and cope with failure and latency as exceptions rather than the rule)
1. intended to be language-independent, but still has "function calls" across languages as main ingredient
1. IDL (Interface Description Language) boilerplate
1. Illusion of type safety

1. The fundamental problem with RPC is coupling. RPC clients become tightly coupled to service implementation in several ways and it becomes very hard to change service implementation without breaking clients:
   * Clients are required to know procedure names;
      * Procedure parameters order, types and count matters. It's not that easy to change procedure signatures(number of arguments, order of arguments, argument types etc...) on server side without breaking client implementations;
      * RPC style doesn't expose anything but procedure endpoints + procedure arguments. It's impossible for client to determine what can be done next.

1. On the other hand in REST style it's very easy to guide clients by including control information in representations(HTTP headers + representation). For example:
   * It's possible (and actually mandatory) to embed links annotated with link relation types which convey meanings of these URIs;
   * Client implementations do not need to depend on particular procedure names and arguments. Instead, clients depend on message formats. This creates possibility to use already implemented libraries for particular media formats (e.g. Atom, HTML, Collection+JSON, HAL etc...)
   * It's possible to easily change URIs without breaking clients as far as they only depend on registered (or domain specific) link relations;
   * It's possible to embed form-like structures in representations, giving clients the possibility to expose these descriptions as UI capabilities if the end user is human;
   * Support for caching is additional advantage;
   * Standardised status codes;

#### When RPC might be better than REST?
1. In general, RPC offers far more of a language integration than REST. As you mentioned, this comes with a number of problems in terms of scale, error handling, type safety, etc., especially when a single distributed system involves multiple hosts running code written in multiple languages. However, after having written business systems that use RPC, REST, and even both simultaneously, I've found that there are some good reasons to choose RPC over REST in certain cases.

1. Here's the cases where I've found RPC to be a better fit:
   * Tight coupling. The (distributed) components of the system are designed to work together, and changing one will likely impact all of the others. It is unlikely that the components will have to be adapted to communicate with other systems in the future.
   * Reliable communication. The components will communicate with each other either entirely on the same host or on a network that is unlikely to experience latency issues, packet loss, etc.. (This still means you need to design your system to handle these cases, however.)
   * Uniform language. All (or mostly all) components will be written in a single language. It is unlikely that additional components written in a different language will be added in the future.

1. Regarding the point about IDL, in a REST system you also have to write code that converts the data in the REST requests and responses to whatever internal data representation you are using. IDL sources (with good comments) can also serve as documentation of the interface, which has to be written and maintained separately for a REST API.

1. The above three items often occur when you are looking to build one component of a larger system. In my experience, these components are often ones where their subsystems need to be able to fail independently and not cause the total failure of other subsystems or the entire component. Many systems are written in Erlang to accomplish these goals as well, and in some cases Erlang may be a better choice than writing a system in another language and using RPC just to gain these benefits.


# Gossip Protocol
1. Gossip protocol is a message protocol that allows state sharing in distributed systems. Most modern systems use this peer-to-peer protocol to disseminate information to all the members in a network or cluster.
2. This protocol is used in a decentralized system that does not have any central node to keep track of all nodes and know if a node is down or not. what key ranges they are responsible for, and so on (this is basically a copy of the hash ring).
3. Not synchronous
## how does a node know every other node’s current state in a decentralized distributed system?
3. The simplest way to do this is to have every node maintain heartbeats with every other node.
4. O(N^2) messages get sent to every tick (N being the number of nodes), which is an expensive operation in any sizable cluster.
5. (Cassandra) Each node initiates a gossip round every second to exchange state information about itself and other nodes with one other random nodes (e.g. 3 nodes in Cassandra). This means that any new event eventually propagates through the system, and all nodes quickly learn about all other nodes in a cluster.
6. Meta data of the state of cluster
   * IP Address of the node
   * Heartbeat state: 
      * When the node started (generation in Cassandra)
      * timestamp of current gossip session (version in Cassandra)
   * Application state
      * Current status of node: normal, leaving, joining the cluster
      * what data center this node is in
      * what rack this node is in
      * schema number changes when schema changes
      * load information of current node
      * Severity: I/O pressure of current node
7. In a gossip session, a node will send:
   * IP of each node it knows
   * Heartbeat state
   * Application state
   * If a node sees newer information, it sends it back
1. Constant rate of network traffic
2. Minimal compared to data streaming, hints
3. doesn't cause network spike


## Seed Node
1. The gossip protocol can result in a logical partition of the cluster in a particular scenario.
2. An administrator joins node A to the ring and then joins node B to the ring. Nodes A and B consider themselves part of the ring, yet neither would be immediately aware of each other. To prevent these logical partitions, some distributed systems use the concept of seed nodes. Seed nodes are fully functional nodes and can be obtained either from a static configuration or a configuration service. This way, all nodes are aware of seed nodes. Each node communicates with seed nodes through gossip protocol to reconcile membership changes. Therefore, logical partitions are highly unlikely.









