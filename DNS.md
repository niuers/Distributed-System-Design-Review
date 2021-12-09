# Overview

1. Domain Name System (DNS) translates a domain name such as www.example.com to an IP address.
2. DNS is hierarchical, with a few *authoritative servers* at the top level. Your router or ISP provides information about which DNS server(s) to contact when doing a lookup. Lower level DNS servers cache mappings, which could become stale due to DNS propagation delays. DNS results can also be cached by your browser or OS for a certain period of time, determined by the time to live (TTL).
1. Services such as CloudFlare and Route 53 provide managed DNS services.

# DNS Servers
1. When users type domain names such as ‘google.com’ or ‘nytimes.com’ into web browsers, DNS is responsible for finding the correct IP address for those sites. Browsers then use those addresses to communicate with origin servers or CDN edge servers to access website information. This all happens thanks to DNS servers: machines dedicated to answering DNS queries.
2. In a typical DNS query without any caching, there are four servers that work together to deliver an IP address to the client: 
   * Recursive resolvers, 
   * Root nameservers, 
   * TLD nameservers, and 
   * Authoritative nameservers.
4. The DNS recursor (or DNS resolver) is a server that receives the query from the client, and then interacts with other DNS servers to hunt down the correct IP. Once the resolver receives the request from the client, the resolver then actually behaves as a client itself, querying the other three types of DNS servers in search of the right IP.
   * First the resolver queries the root nameserver. The root server is the first step in translating (resolving) human-readable domain names into IP addresses. The root server then responds to the resolver with the address of a top-level domain (TLD) DNS server (such as .com or .net) that stores the information for its domains.
   * Next the resolver queries the TLD server. The TLD server responds with the IP address of the domain’s authoritative nameserver. 
   * The recursor then queries the authoritative nameserver, which will respond with the IP address of the origin server.
7. The resolver will finally pass the origin server IP address back to the client. Using this IP address, the client can then initiate a query directly to the origin server, and the origin server will respond by sending website data that can be interpreted and displayed by the web browser.

# DNS Load Balancer Routing Methods
1. This is similar to the routing algorithms used in load balancer

## Weighted round robin
1. Prevent traffic from going to servers under maintenance. 
1. Balance between varying cluster sizes
   * A/B testing

## Latency-based
   * To use latency-based routing, you create latency records for your resources in multiple AWS Regions. When Route 53 receives a DNS query for your domain or subdomain (example.com or acme.example.com), it determines which AWS Regions you've created latency records for, determines which region gives the user the lowest latency, and then selects a latency record for that region. Route 53 responds with the value from the selected record, such as the IP address for a web server.
   * For example, suppose you have ELB load balancers in the US West (Oregon) Region and in the Asia Pacific (Singapore) Region. You created a latency record for each load balancer. Here's what happens when a user in London enters the name of your domain in a browser:

1. DNS routes the query to a Route 53 name server.
1. Route 53 refers to its data on latency between London and the Singapore region and between London and the Oregon region.
1. If latency is lower between the London and Oregon regions, Route 53 responds to the query with the IP address for the Oregon load balancer. If latency is lower between London and the Singapore region, Route 53 responds with the IP address for the Singapore load balancer.

## Geolocation-based
1. Geolocation routing lets you choose the resources that serve your traffic based on the geographic location of your users, meaning the location that DNS queries originate from. For example, you might want all queries from Europe to be routed to an ELB load balancer in the Frankfurt region.
1. When you use geolocation routing, you can localize your content and present some or all of your website in the language of your users. 
2. You can also use geolocation routing to restrict distribution of content to only the locations in which you have distribution rights. 
3. Another possible use is for balancing load across endpoints in a predictable, easy-to-manage way, so that each user location is consistently routed to the same endpoint.

4. Geolocation works by mapping IP addresses to locations. 
   * However, some IP addresses aren't mapped to geographic locations, so even if you create geolocation records that cover all seven continents, Amazon Route 53 will receive some DNS queries from locations that it can't identify. 
   * You can create a default record that handles both queries from IP addresses that aren't mapped to any location and queries that come from locations that you haven't created geolocation records for. 
   * If you don't create a default record, Route 53 returns a "no answer" response for queries from those locations.

## Disadvantage(s): DNS
1. Accessing a DNS server introduces a slight delay, although mitigated by caching described above.
1. DNS server management could be complex and is generally managed by governments, ISPs, and large companies.
1. DNS services have recently come under DDoS attack, preventing users from accessing websites such as Twitter without knowing Twitter's IP address(es).


# DNS Architecture
1. With DNS, the host names reside in a database that can be distributed among multiple servers, decreasing the load on any one server and providing the ability to administer this naming system on a per-partition basis. 
2. DNS supports hierarchical names and allows registration of various data types in addition to host name-to-IP address mapping used in HOSTS files. Because the DNS database is distributed, its potential size is unlimited and performance is not degraded when more servers are added.

## DNS domain names
1. The Domain Name System is implemented as a **hierarchical and distributed database** containing various types of data, including host names and domain names. The names in a DNS database form a hierarchical tree structure called the domain namespace. Domain names consist of individual labels separated by dots, for example: mydomain.microsoft.com.

2. A fully qualified domain name (FQDN) uniquely identifies the host’s position within the DNS hierarchical tree by specifying a list of names separated by dots in the path from the referenced host to the root. 
3. DNS clients and servers use queries as the fundamental method of resolving names in the tree to specific types of resource information. This information is provided by DNS servers in query responses to DNS clients, which then extract the information and pass it to a requesting program for resolving the queried name. In the process of resolving a name, keep in mind that DNS servers often function as DNS clients, querying other servers in order to fully resolve a queried name.
4. The five categories used to describe DNS domain names by their function in the namespace
   * Root Domain, e.g. the single period at the end of the name, '.'
   * Top-level domain, e.g. '.com'
   * Second-level domain, e.g. 'microsoft.com'
   * Subdomain, e.g. 'subdomain.microsoft.com'
   * Host or Resources name, e.g. 
      * Names that represent a leaf in the DNS tree of names and identify a specific resource. Typically, the leftmost label of a DNS domain name identifies a specific computer on the network. For example, if a name at this level is used in a host (A) resource record, it is used to look up the IP address of computer based on its host name.
      * e.g. ““host-a.example.microsoft.com.”, where the first label (“host-a”) is the DNS host name for a specific computer on the network.
## Resource records
1. A DNS database consists of resource records (RRs). Each RR identifies a particular resource within the database.
3. Components
   * NS record (name server) - Specifies the DNS servers for your domain/subdomain.
   * MX record (mail exchange) - Specifies the mail servers for accepting messages.
   * A record (address) - Points a name to an IP address.
   * CNAME (canonical) - Points a name to another name or CNAME (example.com to www.example.com) or to an A record.

## Distributing the DNS database: zone files and delegation
1. A DNS database can be partitioned into multiple zones. A zone is a portion of the DNS database that contains the resource records with the owner names that belong to the contiguous portion of the DNS namespace. Zone files are maintained on DNS servers. A single DNS server can be configured to host zero, one, or multiple zones.
1. Each zone is anchored at a specific domain name referred to as the zone’s root domain. A zone contains information about all names that end with the zone’s root domain name. A DNS server is considered authoritative for a name if it loads the zone containing that name. 
2. The first record in any zone file is a Start of Authority (SOA) RR. The SOA RR identifies a primary DNS name server for the zone as the best source of information for the data within that zone and as an entity processing the updates for the zone.
3. A name within a zone can also be delegated to a different zone that is hosted on a different DNS server. Delegation is a process of assigning responsibility for a portion of a DNS namespace to a DNS server owned by a separate entity. 
4. Such delegation is represented by the NS resource record that specifies the delegated zone and the DNS name of the server authoritative for that zone. Delegating across multiple zones was part of the original design goal of DNS.
5. The primary reasons to delegate a DNS namespace include:
   * A need to delegate management of a DNS domain to a number of organizations or departments within an organization.
   * A need to distribute the load of maintaining one large DNS database among multiple DNS servers to improve the name resolution performance as well as create a DNS fault-tolerant environment.
   * A need to allow for a host’s organizational affiliation by including the host in appropriate domains.
1. The name server (NS) RRs facilitate delegation by identifying DNS servers for each zone and the NS RRs appear in all zones. Whenever a DNS server needs to cross a delegation in order to resolve a name, it will refer to the NS RRs for DNS servers in the target zone.

## Replicating the DNS database
1. There can be multiple zones representing the same portion of the namespace. Among these zones there are three types:
   * Primary: Primary is a zone to which all updates for the records that belong to that zone are made.
   * Secondary: A secondary zone is a read-only copy of the primary zone.
   * Stub: A stub zone is a read-only copy of the primary zone that contains only the resource records that identify the DNS servers that are authoritative for a DNS domain name.
1. Any changes made to the primary zone file are replicated to the secondary zone file. DNS servers hosting a primary, secondary, or stub zone are said to be authoritative for the DNS names in the zone.
1. Because a DNS server can host multiple zones, it can therefore host both a primary zone (which has the writeable copy of a zone file) and a separate secondary zone (which obtains a read-only copy of a zone file). 
2. A DNS server hosting a primary zone is said to be the primary DNS server for that zone, and a DNS server hosting a secondary zone is said to be the secondary DNS server for that zone.

### Zone transfer
1. The process of replicating a zone file to multiple DNS servers is called zone transfer. 
2. Zone transfer is achieved by copying the zone file from one DNS server to a second DNS server. Zone transfers can be made from both primary and secondary DNS servers.
1. A master DNS server is the source of the zone information during a transfer. The master DNS server can be a primary or secondary DNS server. If the master DNS server is a primary DNS server, then the zone transfer comes directly from the DNS server hosting the primary zone. If the master server is a secondary DNS server, then the zone file received from the master DNS server by means of a zone transfer is a copy of the read-only secondary zone file.
1. The zone transfer is initiated in one of the following ways:
   * The master DNS server sends a notification (RFC 1996) to one or more secondary DNS servers of a change in the zone file.
   * When the DNS Server service on the secondary DNS server starts, or the refresh interval of the zone has expired (by default it is set to 15 minutes in the SOA RR of the zone), the secondary DNS server will query the master DNS server for the changes.

### Types of zone file replication
1. There are two types of zone file replication. 
   * The first, a full zone transfer (AXFR), replicates the entire zone file. 
   * The second, an incremental zone transfer (IXFR), replicates only records that have been modified.

1. BIND and Windows NT 4.0 DNS, support full zone transfer (AXFR) only. 
2. There are two types of the AXFR: 
   * One requires a single record per packet, 
   * The other allows multiple records per packet. 

# Querying the database
1. DNS queries can be sent from a DNS client (resolver) to a DNS server, or between two DNS servers.
1. A DNS query is merely a request for DNS resource records of a specified resource record type with a specified DNS name. For example, a DNS query can request all resource records of type A (host) with a specified DNS name.

1. There are two types of DNS queries that can be sent to a DNS server:
   * Recursive
   * Iterative
1. A recursive query forces a DNS server to respond to a request with either a failure or a success response. DNS clients (resolvers) typically make recursive queries. With a recursive query, the DNS server must contact any other DNS servers it needs to resolve the request. When it receives a successful response from the other DNS server (or servers), it then sends a response to the DNS client. The recursive query is the typical query type used by a resolver querying a DNS server and by a DNS server querying its forwarder, which is another DNS server configured to handle requests forwarded to it.
   * When a DNS server processes a recursive query and the query cannot be resolved from local data (local zone files or cache of previous queries), the recursive query must be escalated to a root DNS server. Each standards-based implementation of DNS includes a cache file (or root server hints) that contains entries for the root DNS servers of the Internet domains. (If the DNS server is configured with a forwarder, the forwarder is used before a root server is used.)
1. An iterative query is one in which the DNS server is expected to respond with the best local information it has, based on what the DNS server knows from local zone files or from caching. This response is also known as a referral if the DNS server is not authoritative for the name. If a DNS server does not have any local information that can answer the query, it simply sends a negative response. A DNS server makes this type of query as it tries to find names outside of its local domain (or domains) (when it is not configured with a forwarder). It might have to query a number of outside DNS servers in an attempt to resolve the name.
   * e.g. queries between Name Server and Root Server/TLD Server etc.

# Time-to-Live for resource records
1. The Time-to-Live (TTL) value in a resource record indicates a length of time used by other DNS servers to determine how long to cache information for a record before expiring and discarding it. For example, most resource records created by the DNS Server service inherit the minimum (default) TTL of one hour from the start of authority (SOA) resource record, which prevents extended caching by other DNS servers.
1. A DNS client resolver caches the responses it receives when it resolves DNS queries. These cached responses can then be used to answer later queries for the same information. The cached data, however, has a limited lifetime specified in the TTL parameter returned with the response data. 
2. TTL ensures that the DNS server does not keep information for so long that it becomes out of date. TTL for the cache can be set on the DNS database (for each individual resource record, by specifying the TTL field of the record and per zone through the minimum TTL field of the SOA record) as well as on the DNS client resolver side by specifying the maximum TTL the resolver allows to cache the resource records.
3. There are two competing factors to consider when setting the TTL. 
   * The first is the accuracy of the cached information, and 
   * The second is the utilization of the DNS servers and the amount of network traffic. 
4. If the TTL is short, then the likelihood of having old information is reduced considerably, but it increases utilization of DNS servers and network traffic, because the DNS client must query DNS servers for the expired data the next time it is requested. If the TTL is long, the cached responses could become outdated, meaning the resolver could give false answers to queries. At the same time, a long TTL decreases utilization of DNS servers and reduces network traffic because the DNS client answers queries using its cached data.
5. If a query is answered with an entry from cache, the TTL of the entry is also passed with the response. This way the resolvers that receive the response know how long the entry is valid. The resolvers honor the TTL from the responding server; they do not reset it based on their own TTL.
6. Consequently, entries truly expire rather than live in perpetuity as they move from DNS server to DNS server with an updated TTL.
