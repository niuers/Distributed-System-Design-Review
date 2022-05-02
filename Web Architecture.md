
# Overview
1. Tier is a logical separation of components in an application or a service. Here separation means physical separation at the component level, not the code level. 
1. A two-tier application involves a client and a server.
2. In a three-tier application, the user interface, application logic, and the database all lie on different machines and, thus, have different tiers. They are physically separated.
## Why the need for so many tiers?#
1. Two software design principles that are key to explaining this are the single responsibility principle and the separation of concerns.

## Layer vs. Tier
1. In the industry, layers of an application typically means the user interface layer, business layer, service layer, or the data access layer.
2. The difference between layers and tiers is that layers represent the conceptual organization of the code and its components, whereas, tiers represent the physical separation of components.

# Client-Server Architecture
1. The architecture works on a request-response model. The client sends the request to the server for information and the server responds with it.
2. The client holds our user interface. The user interface is the presentation part of the application. It’s written in HTML, JavaScript, CSS and is responsible for the look and feel of the application.
3. The user interface runs on the client. The client can be a mobile app, a desktop or a tablet like an iPad. It can also be a web-based console, running commands to interact with the backend server.

## There are primarily two types of clients:
### Thin client
1. Thin client is the client that holds just the user interface of the application. It has no business logic of any sort. For every action, the client sends a request to the backend server. 

### Thick client

## What is a web server?
1. The primary task of a web server is to receive the requests from the client and provide the response after executing the business logic based on the request parameters received from the client.
1. Every service, running online, needs a server to run. Servers running web applications are commonly known as the application servers.
2. Besides the application servers, there are also other kinds of servers with specific tasks assigned to them. These include the:
   * Proxy server
   * Mail server
   * File server
   * Virtual server

1. All the components of a web application need a server to run, be it a database, a message queue, a cache, or any other component. 
2. Server-side rendering: Often the developers use a server to render the user interface on the backend and then send the rendered data to the client. This technique is known as server-side rendering. 

# Communication Between the Client and the Server
## Request-response model#
1. The client and the server have a request-response model. The client sends the request and the server responds with the data.
1. HTTP protocol: The entire communication happens over the HTTP protocol
### REST API and API Endpoints
1. Speaking from the context of modern N-tier web applications, every client has to hit a REST endpoint to fetch the data from the backend.
1. The backend application code has a REST-API implemented. This acts as an interface to the outside world requests. Every request, be it from the client written by the business or the third-party developers that consume our data has to hit the REST-endpoints to fetch the data.
#### REST 
1. REST stands for Representational State Transfer. It’s a software architectural style for implementing web services. Web services implemented using the REST architectural style are known as the RESTful Web services.
#### REST API 
1. A REST API is an API implementation that adheres to the REST architectural constraints. It acts as an interface. The communication between the client and the server happens over HTTP. A REST API takes advantage of the HTTP methodologies to establish communication between the client and the server. REST also enables servers to cache the response that improves the application’s performance.
2. When implementing a REST API the client communicates with the backend endpoints. This entirely decouples the backend and the client code.

#### REST Endpoints
1. An API/REST/Backend endpoint means the URL of a service. For example, https://myservice.com/users/{username} is a backend endpoint for fetching the user details of a particular user from the service. The REST-based service will expose this URL to all its clients to fetch the user details using the above stated URL.

#### Decoupling clients and the backend service#
1. With the availability of the endpoints, the backend service does not have to worry about the client implementation. It just calls out to its multiple clients and says “Hey everyone! Here is the URL address of the resource/information you need. Hit it when you need it. Any client with the required authorization to access a resource can access it”.

### API gateway
1. The REST-API acts as a gateway, or a single-entry point into the system. It encapsulates the business logic and handles all the client requests, taking care of the authorization, authentication, sanitizing the input data, and other necessary tasks before providing access to the application resources.


# HTTP PULL AND PUSH
## HTTP PULL
1. for every response, there has to be a request first. The client sends the request and the server responds with the data. This is the default mode of HTTP communication, called the HTTP PULL mechanism.
2. The client pulls the data from the server whenever required. It keeps doing this over and over to fetch the updated data.
3. An important thing to note here is that every request to the server and the response to it consumes bandwidth. Every hit on the server costs the business money and adds more load on the server.
4. Excessive pulls by the clients have the potential to bring down the server.
5. There are two ways of pulling/fetching data from the server.
   * The first is sending an HTTP GET request to the server manually by triggering an event by clicking a button or any other element on the web page.
   * The other is fetching data dynamically at regular intervals by using AJAX without any human intervention.

### AJAX – Asynchronous JavaScript and XML
1. AJAX enables us to fetch the updated data from the server by automatically sending the requests over and over at stipulated intervals.
2. Upon receiving the updates, a particular section of the web page is updated dynamically by the callback method.
3. AJAX uses an XMLHttpRequest object for sending the requests to the server which is built-in the browser and uses JavaScript to update the HTML DOM.
4. AJAX is commonly used with the jQuery framework to implement the asynchronous behavior on the UI.
5. This dynamic technique of requesting information from the server after regular intervals is known as **polling**.

## HTTP PUSH
1. the client sends the request for particular information to the server just once. After the first request, the server keeps pushing the new updates to the client whenever they are available.
2. This is also known as a callback. The client phones the server for information. The server responds, “Hey!! I don’t have the information right now but I’ll call you back whenever it is available”.
3. A very common example of this is user notifications. We have them in almost every web application today. We get notified whenever an event happens on the backend.
4. When the application is a real-time application like an online multiplayer game, a LIVE sports app. When we need to reduce the number of client requests hitting the server every now and then, checking for new information.
5. Clients use Asynchronous JavaScript & XML (AJAX) to send requests to the server in the HTTP PULL based mechanism.
6. There are multiple technologies involved in the HTTP PUSH based mechanism such as:
   * Ajax Long polling
   * Web Sockets
   * HTML5 Event Source
   * Message Queues
   * Streaming over HTTP

### Time To Live (TTL)
1. In the regular client-server communication, which is HTTP PULL, there is a Time to Live (TTL) for every request in browser. It could be 30 secs to 60 secs, varying from browser to browser.
2. If the client doesn’t receive a response from the server within the TTL, the browser kills the connection and the client has to re-send the request hoping it receives the data from the server before the TTL ends again.
3. But what if we are certain that the response will take more time than the TTL set by the browser?

### Persistent connection
1. A persistent connection is a network connection between the client and the server that remains open for further requests and responses, as opposed to being closed after a single communication.
2. This facilitates HTTP PUSH-based communication between the client and the server.
3. When we are certain that the response time of the server will be more than the TTL set by the browser.
4. When the frequency of request-response is high, persistent connections avert the need to open and close a new connection, between the client and the server, every time a conversation happens. They do this by keeping the connection opened for a longer period of time.

#### Heartbeat interceptors
1. Now you might be wondering how is a persistent connection possible if the browser kills the open connections to the server every X seconds?
2. The connection between the client and the server stays open with the help of Heartbeat Interceptors.
3. These are just blank request responses between the client and the server to prevent the browser from killing the connection.
4. Isn’t this resource-intensive?

#### Resource intensive
1. Yes, it is. Persistent connections consume a lot of resources compared to the HTTP PULL behavior. However, there are use cases where establishing a persistent connection is vital to an application’s feature.
2. Long opened connections can be implemented by multiple techniques such as AJAX Long Polling, Web Sockets, Server-Sent Events, etc.

### Web Sockets
1. A Web Socket connection is preferred when we need a persistent bi-directional low latency data flow from the client to server and back.
2. Typical use-cases of these are messaging, chat applications, real-time social streams, and browser-based massive multiplayer games which have quite a number of read writes in comparison to a regular web app.
3. Web Sockets tech doesn’t work over HTTP. It runs over TCP.


### AJAX – Long polling
1. Long polling lies somewhere between AJAX and Web Sockets. In this technique instead of immediately returning the response, the server holds the response until it finds an update to be sent to the client.
2. The connection in long polling stays open a bit longer compared to polling. The server doesn’t return an empty response. If the connection breaks, the client has to re-establish the connection to the server.
3. Long polling can be used in simple asynchronous data fetch use cases when you do not want to poll the server every now and then.


### HTML5 Event-Source API and Server-Sent Events
1. The Server-Sent Events (SSE) implementation takes a bit of a different approach. Instead of the client polling for data, the server automatically pushes the data to the client whenever the updates are available. The incoming messages from the server are treated as events.
2. To implement server-sent events, the backend language should support the technology, and on the UI HTML5 Event-Source API is used to receive the data in-coming from the backend.
3. An important thing to note here is that once the client establishes a connection with the server, the data flow is in one direction only, that is from the server to the client.
4. SSE is ideal for scenarios like a real-time Twitter feed, displaying stock quotes on the UI, real-time notifications etc.
### Streaming over HTTP
1. Streaming over HTTP is ideal for cases where we need to stream large data over HTTP by breaking it into smaller chunks. This is possible with HTML5 and a JavaScript Stream API.
2. The technique is primarily used for streaming multimedia content, like large images, videos etc, over HTTP.


# Client-side rendering - How does a browser render a web page?
1. When a user requests a web page from the server and the browser receives the response, it has to render the response on the window in the form of an HTML page.For this, the browser has several components, such as the:
   * Browser engine
   * Rendering engine
   * JavaScript interpreter
   * Networking and the UI backend
   * Data storage etc.
1. The rendering engine constructs the DOM tree and renders and paints the construction. Naturally, all this activity needs a bit of time.
## Server-side Rendering
1. To avoid all this rendering time on the client, developers often render the UI on the server, generate HTML there and directly send the HTML page to the UI.
3. This technique is known as the server-side rendering. It ensures faster rendering of the UI, averting the UI loading time in the browser window because the page is already created and the browser doesn’t have to do much assembling or rendering work.
4. Static content can be rendered on the server in comparatively less time as opposed to being rendered on the client. It can also be cached for future requests.
5. 2. Server Side Includes SSI is a server-side scripting language used for clubbing the content of multiple web pages on the web server.


### Use cases for server-side & client-side rendering
1. The server-side rendering approach is perfect for delivering static content, such as WordPress blogs. It’s also good for SEO because the crawlers can easily read the generated content.
2. However, modern websites are highly dependent on AJAX. On such websites, content for a particular module or a section of a page has to be fetched and rendered on the fly.
3. Therefore, server-side rendering doesn’t help much.
4. A big downside to this is that, once the number of concurrent users on the website rises, it puts an unnecessary load on the server.
5. Client-side rendering works best for modern dynamic AJAX-based websites.


# Microservices
1. Remember the Single Responsibility and the Separation of Concerns principles? Both principles are applied in a microservices architecture.
2. Every component interacts with each other via a REST API Gateway interface.
3. No strong consistency: Strong consistency is hard to guarantee in a distributed environment. Things are eventually consistent across the nodes, and this limitation is due to the distributed design.
4. 

# Event-Driven Architecture
1. There are generally two kinds of processes in applications: CPU intensive and IO intensive. In the context of web applications, IO means events. 
2. Non-blocking architecture is also known as reactive or event-driven architecture. 
3. Technologies like NodeJS, frameworks in the Java ecosystem like Play, and Akka.io are non-blocking in nature and are built for modern high IO scalable applications.

## Webhooks
1. Webhooks are more like call-backs. It’s like, “I will call you when new information is available. You carry on with your work.”
2. Webhooks enable communication between two services without the middleware. They have an event-based mechanism.
3. To use the Webhooks, consumers register an HTTP endpoint with the service with a unique API Key. It’s like a phone number. Call me on this number, when an event occurs. I won’t call you anymore.
4. Whenever new information is available on the backend, the server fires an HTTP event to all the registered endpoints of the consumers, notifying them of the new update.
5. Browser notifications are a good example of Webhooks. Instead of visiting the websites every now and then for new info, the websites notify us when they publish new content.

# hexagonal architecture?#
1. The architecture consists of three components:
   * Ports
   * Adapters
   * Domain

1. The architectural pattern holds the domain at its core, meaning the business logic. On the outside, the outer layer has ports and adapters. Ports act like an API as an interface. All the input to the app goes through the interface.
2. 

# Peer-to-Peer Architecture
1. A P2P network is a network in which computers, also known as nodes, can communicate with each other without a central server. The absence of a central server rules out the possibility of a single point of failure. All the computers in the network have equal rights. A node acts as a seeder and a leecher at the same time. So, even if some of the computers/nodes go down, the network and the communication is still up.
2. A seeder is a node that hosts the data on its system and provides bandwidth to upload the data to the network, and a leecher is a node that downloads the data from the network.
3. The data is exchanged over TCP IP just like it happens over the HTTP protocol in a client-server model. The P2P design has an overlay network over TCP IP, which enables the users to connect directly. 
## Types of P2P networks
1. structured: In an unstructured network nodes/peers keep connecting with each other randomly. So, there is no structure, no rule. Just simply connect and grow the network.
   * In this architectural design, there is no indexing of the nodes. To search the data, we have to scan through each and every node in the network. This is O(n) in complexity where n is the number of nodes in the network. This is very resource-intensive.
   * Some of the unstructured network’s protocols are Gossip, Kazaa, and Gnutella.
3. unstructured
   * a structured P2P network holds the proper node indexing, or the topology. This makes it easier to search for specific data.This kind of network implements a distributed hash table to index the nodes.
   * BitTorrent
5. or a hybrid model
   * It is a network, involving both a peer to peer and a client-server model.
   * A P2P network offers more availability. 

### Decentralized social networks
1. They have servers, spread out across the globe, and hosted by individuals like you and me. Nobody has autonomous control over the network, and everybody has an equal say.



# federated network
1. a federated network has entities called servers or pods. A large number of nodes subscribe to the pods. There are several pods in the network that are linked to each other and share information with each other.
2. Pods facilitate node discovery. In a peer-to-peer network, there is no way of discovering other nodes, and we would just sit in the dark if it weren’t for a centralized node registry or something. The other way is to run a scan through the network to try to discover other nodes. This is a really time-consuming and tedious task. 
3. A pod in a federated architecture acts like a server to which a large number of nodes are subscribed.

# [API Key](https://www.freecodecamp.org/news/best-practices-for-building-api-keys-97c26eabfea9/)
1. [API keys are for projects, authentication is for users](https://cloud.google.com/endpoints/docs/openapi/when-why-api-key)
  1. API keys identify the calling project — the application or site — making the call to an API. API keys aren't as secure as authentication tokens (see Security of API keys), but they identify the application or project that's calling an API. 
    1. API keys are generally not considered secure; they are typically accessible to clients, making it easy for someone to steal an API key. Once the key is stolen, it has no expiration, so it may be used indefinitely, unless the project owner revokes or regenerates the key. While the restrictions you can set on an API key mitigate this, there are better approaches for authorization.
  3. Authentication tokens identify a user — the person — that is using the app or site.
    1.  User authentication — Securely verify that the calling user is who they claim to be.
    2.  User authorization — Check whether the user should have access to make this request.

2. When to use API keys
  1. An API may restrict some or all of its methods to require API keys. It makes sense to do this if:
    1. You do want to block anonymous traffic. API keys identify an application's traffic for the API producer, in case the application developer needs to work with the API producer to debug an issue or show their application's usage.
    2. You want to control the number of calls made to your API.
    3. You want to identify usage patterns in your API's traffic. You can see application usage in APIs & services.
    4. You want to filter logs by API key.
  2. API keys cannot be used for:
    1. Identifying individual users — API keys don't identify users, they identify projects.
    2. Secure authorization.
    3. Identifying the creators of a project.


