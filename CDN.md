# Overview of Content delivery networks (CDN)

1. [CDN are used to solve the latency issue](https://www.imperva.com/learn/performance/what-is-cdn-how-it-works)
   * In all cases however, the delay duration is impacted by the physical distance between you and that website’s hosting server.
   * A CDN’s mission is to virtually shorten that physical distance, the goal being to improve site rendering speed and performance.
   * To minimize the distance between the visitors and your website’s server, a CDN stores a cached version of its content in multiple geographical locations (a.k.a., **points of presence**, or PoPs). Each PoP contains a number of caching servers responsible for content delivery to visitors within its proximity.
3. It can be very important to improve the performance by pushing the popular videos (youtube) to CDN, and reduce latency.
   * Flash crowd problem for popular websites, request load overwhelms some aspect of the site’s infrastructure, such as the frontend Web server, network equipment, or bandwidth, or (in more advanced sites) the back-end transaction-processing infrastructure.
   * By caching content at the Internet’s edge, we reduce demand on the site’s infrastructure and provide faster service for users, whose content comes from nearby servers.
   * 

1. A content delivery network (CDN) is a globally distributed network of proxy servers, serving content from locations closer to the user. Generally, static files such as HTML/CSS/JS, photos, and videos are served from CDN, although some CDNs such as Amazon's CloudFront support dynamic content. The site's DNS resolution will tell clients which server to contact.

1. Serving content from CDNs can significantly improve performance in two ways:
   * Users receive content from data centers close to them
   * Your servers do not have to serve requests that the CDN fulfills
## Modern CDNs can handle numerous IT tasks, helping you to:
1. Improve page load speed
1. Handle high traffic loads
1. Block spammers, scrapers and other bad bots
1. Localize coverage without the cost
1. Reduce bandwidth consumption
1. Load balance between multiple servers
1. Protect your website from DDoS attacks
1. Secure your application

## Make CDN Work
1. For a CDN to work, it needs to be the default inbound gateway for all incoming traffic. To make this happen, you’ll need to modify your root domain DNS configurations (e.g., domain.com) and those of your subdomains (e.g., www.domain.com, img.domain.com).
2. For your root domain, you’ll change its A record to point to one of the CDN’s IP ranges. For each subdomain, modify its CNAME record to point to a CDN-provided subdomain address (e.g., ns1.cdn.com). In both cases, this results in the DNS routing all visitors to your CDN instead of being directed to your original server.
3. Content delivery networks employ reverse proxy technology. Topology wise, this means CDNs are deployed in front of your backend server(s).
4. This position, on the edge of your network perimeter, offers several key advantages beyond a CDN’s innate ability to accelerate content delivery.
   * Website Security: The on-edge position also makes a CDN ideal for blocking DDoS floods, which need to be mitigated outside of your core network infrastructure.
   * Load Balancing: 



## Caching Method
1. Caching method: Most are origin pull ?



## Main Components
1. PoPs (Points of Presence)
   * CDN PoPs (Points of Presence) are strategically located data centers responsible for communicating with users in their geographic vicinity. Their main function is to reduce round trip time by bringing the content closer to the website’s visitor. Each CDN PoP typically contains numerous caching servers.
1. Caching servers: Caching servers are responsible for the storage and delivery of cached files. Their main function is to accelerate website load times and reduce bandwidth consumption. Each CDN caching server typically holds multiple storage drives and high amounts of RAM resources.
2. SSD/HDD + RAM: Inside CDN caching servers, cached files are stored on solid-state and hard-disk drives (SSD and HDD) or in random-access memory (RAM), with the more commonly-used files hosted on the more speedy mediums. Being the fastest of the three, RAM is typically used to store the most frequently-accessed items.







## Potential Problems
1. Operating servers in many locations poses many technical challenges, including: 
   * how to direct user requests to appropriate servers, 
   * how to handle failures, 
   * how to monitor and control the servers, and
   * how to update software across the system

### Traditional Approaches
1. Local clustering can improve fault-tolerance and scalability. If the data center or the ISP providing connectivity fails, however, the entire cluster is inaccessible to users. To solve this problem, sites can offer **mirroring** (deploying clusters in a few locations) and **multihoming** (using multiple ISPs to connect to the Internet). Clustering, mirroring, and multihoming are common approaches for sites with stringent reliability and scalability needs. These methods do not solve all connectivity problems, however, and they do introduce new ones:
   * It is difficult to scale clusters to thousands of servers.
   * With multihoming, the underlying network protocols — in particular the border gateway protocol (BGP)2 — do not converge quickly to new routes when connections fail.
   * Mirroring requires synchronizing the site among the mirrors, which can be difficult.
1. In all three cases, excess capacity is required
   * With clustering, there must be enough servers at each location to handle peak loads (which can be an order of magnitude above average loads); 
   * with multihoming, each connection must be able to carry all the traffic; and 
   * with mirroring, each mirror must be able to carry the entire load. 
1. Each of these solutions thus entails considerable cost, which could more than double a site’s initial infrastructure expense and ongoing operation costs
1. Deploying independent proxy caches throughout the Internet can address some of these bottlenecks. However, As a result, proxy caches have had limited success in improving Web sites’ scalability, reliability, and performance.
1. proxy caches cannot typically cache dynamic content. A proxy cache could not, for example, handle a largely static Web page if it contained an advertisement that changed according to each user’s profile.


#### Failures of Internet
Congestion and failures occur at many places, including
1. the “first mile” (which is partially addressed by multihoming the origin server),
1. the backbones,
1. peering points between network service providers, and
1. the “last mile” to the user.

## Architecture of Akamai
The system directs client requests to the nearest available server likely to have the requested content. It determines this as follows:
1. Nearest is a function of network topology and dynamic link characteristics: A server with a lower round-trip time is considered nearer than
one with a higher round-trip time. Likewise, a server with low packet loss to the client is nearer than one with high packet loss.
1. Available is a function of load and network bandwidth: A server carrying too much load or a data center serving near its bandwidth capacity is unavailable to serve more clients.
1. Likely is a function of which servers carry the content for each customer in a data center: If all servers served all the content — by roundrobin DNS, for example — then the servers’ disk and memory resources would be consumed by the most popular set of objects.

### Mapping
1. The direction of requests to content servers is referred to as mapping.
2. Akamai mapping uses a dynamic, fault-tolerant DNS system.
3. The mapping system resolves a hostname based on the service requested, user location, and network status; it also uses DNS for network load-balancing.
4. 

### Internet Topology
1. Internet routers use BGP messages to exchange network reachability information among BGP systems and compute the best routing path among the Internet’s autonomous systems.2 Akamai agents communicate with certain border routers as peers; the mapping system uses the resulting BGP information to determine network topology. The number of hops between autonomous systems is a coarse but useful measure of network distance. The mapping system combines this information with live network statistics — such as traceroute data3 — to provide a detailed, dynamic view of network structure and quality measures for different mappings.

### Network Services
1. Using ESI (Edge Side Includes) lets a content provider break a dynamic page into fragments with independent cacheability properties. These fragments are maintained as separate objects in the edge server’s cache and are dynamically assembled into Web pages in response to user requests.
1. Media packet delivery from the entry-point to the edge servers must be resilient to network failures and packet loss, and thus the entry point server must route packet flows around congested and failed links to reach the edge server. Further, the entry point and edge servers must deliver packets without significant delay or jitter because a late or out-of-order packet is useless in the playback. When necessary, Akamai uses information dispersal techniques that let the entry point server send data on multiple redundant paths, which lets the edge server construct a clean copy of the stream even when some paths are down or lossy

### Technique Challenges
#### Scalability
Akamai’s network must scale to support many geographically distributed servers, and many customers with differing needs. This presents the following
challenges.
1. Monitoring and controlling tens of thousands of widely distributed servers, while keeping monitoring bandwidth to a minimum.
1. Monitoring network conditions across and between thousands of locations, aggregating that information, and using it to generate new maps every few seconds. Success here depends on minimizing the overhead added to DNS to avoid long DNS lookup times. This lets us perform the calculations required to identify the optimal server offline, rather than making the user wait.
1. Dealing gracefully with incomplete and out-ofdate information. This requires careful design and iterative algorithm tuning.
1. Reacting quickly to changing network conditions and changing workloads.
1. Measuring Internet conditions at a fine enough granularity to attain high-probability estimates of end-user performance.
1. Managing, provisioning, and solving problems for numerous customers with varying needs, varying workloads, and varying amounts of content.
1. Isolating customers so they cannot negatively affect each other.
1. Ensuring data integrity over many terabytes of storage across the network. Because low-level (file system or disk) checks are inadequate to protect against possible errors — including those caused by operators and software bugs — we also perform end-to-end checks.
1. Collecting logs with information about user requests, processing these logs (for billing), and delivering accurate, timely billing information to customers. 
2. To meet the challenges of monitoring and controlling content servers, Akamai developed a distributed monitoring service that is resilient to temporary loss of information. To solve problems for customers, Akamai has customer-focused teams that diagnose problems and provide billing services.



#### Reliability
1. For customers still using cached DNS answers, two mechanisms help prevent denial of service: the DNS resolution can return multiple IP addresses, so that the client can try another address; or a live server at the site can assume the failed server’s IP address. To prevent problems when network failures make sites unreachable (due to router and switch problems, for example), the top-level (TL) DNS will identify local DNS servers at different sites to ensure that clients can reach a live DNS server.
2. 
#### Content Visibility and Control
1. Akamai’s network distributes and serves content for providers, who must retain control over their content as we serve it at the edge, and see real-time information about what content is served to whom and when. Providing this visibility and content control offers challenges in cache consistency, lifetime and integrity control, and several other areas.
2. Cache Consistency
   * TTL
   * Use different URL for each object version. Versioned objects typically have infinite TTLs.
   * 
3. To improve uncacheable objects’ performance, we introduce an edge server between the client and origin to split the client’s TCP connection into two separate connections — one from the client to the edge server and one from the edge server to the origin. Contrary to intuition, splitting the connection can deliver faster responses in some cases because the edge server can react to packet loss more quickly than the origin server, improving the connection’s bandwidth-delay product. We also map clients to edge servers that have low congestion and packet loss. Furthermore, the edge server can accept the origin server’s response faster than the client could, and can serve it from memory at the client’s pace. This frees up origin server resources to serve subsequent requests, reducing origin site demand even for uncacheable content. Finally, the edge server can maintain much longer persistent connections with the client than can an origin server; the origin need only maintain connections with relatively few Akamai edge servers

#### Lifetime Control
#### Authentication and authorization
#### Integrity control. 
A server must ensure that each client request receives the correct response, and also detect when origin servers issue incomplete responses and avoid caching those responses.
#### 


##### Cache consistency. 
1. When objects that the edge servers deliver are cacheable, we must address the consistency of cached content; when they are uncacheable, high-performance delivery is a challenge.
2. Web Caching
   * The Web has long used caching, first in browsers and then in forward proxies.
   * 5Web caching’s lack of success relative to other types of caching might be because Web proxy caches are not closely coordinated with the data sources; caching software is built and administered by organizations distinct from those that provide the data.
3. 


## Types of CDN
### Push CDNs
1. Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. You can configure when content expires and when it is updated. Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.
1. Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.

### Pull CDNs

1. Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. This results in a slower request until the content is cached on the CDN.
1. A time-to-live (TTL) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.
1. Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.

## Disadvantage(s): CDN
1. CDN costs could be significant depending on traffic, although this should be weighed with additional costs you would incur not using a CDN.
1. Content might be stale if it is updated before the TTL expires it.
1. CDNs require changing URLs for static content to point to the CDN.

### Product
1. Amazon CloudFront
2. 
