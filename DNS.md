# Domain Name System

1. A Domain Name System (DNS) translates a domain name such as www.example.com to an IP address.
2. DNS is hierarchical, with a few authoritative servers at the top level. Your router or ISP provides information about which DNS server(s) to contact when doing a lookup. Lower level DNS servers cache mappings, which could become stale due to DNS propagation delays. DNS results can also be cached by your browser or OS for a certain period of time, determined by the time to live (TTL).

3. Components
   * NS record (name server) - Specifies the DNS servers for your domain/subdomain.
   * MX record (mail exchange) - Specifies the mail servers for accepting messages.
   * A record (address) - Points a name to an IP address.
   * CNAME (canonical) - Points a name to another name or CNAME (example.com to www.example.com) or to an A record.
4. Services such as CloudFlare and Route 53 provide managed DNS services. Some DNS services can route traffic through various methods:

## Routing Methods
1. This is similar to the routing algorithms used in load balancer

### Weighted round robin
1. Prevent traffic from going to servers under maintenance. 
1. Balance between varying cluster sizes
   * A/B testing
### Latency-based
   * To use latency-based routing, you create latency records for your resources in multiple AWS Regions. When Route 53 receives a DNS query for your domain or subdomain (example.com or acme.example.com), it determines which AWS Regions you've created latency records for, determines which region gives the user the lowest latency, and then selects a latency record for that region. Route 53 responds with the value from the selected record, such as the IP address for a web server.
   * For example, suppose you have ELB load balancers in the US West (Oregon) Region and in the Asia Pacific (Singapore) Region. You created a latency record for each load balancer. Here's what happens when a user in London enters the name of your domain in a browser:

1. DNS routes the query to a Route 53 name server.
1. Route 53 refers to its data on latency between London and the Singapore region and between London and the Oregon region.
1. If latency is lower between the London and Oregon regions, Route 53 responds to the query with the IP address for the Oregon load balancer. If latency is lower between London and the Singapore region, Route 53 responds with the IP address for the Singapore load balancer.


### Geolocation-based
1. Geolocation routing lets you choose the resources that serve your traffic based on the geographic location of your users, meaning the location that DNS queries originate from. For example, you might want all queries from Europe to be routed to an ELB load balancer in the Frankfurt region.
1. When you use geolocation routing, you can localize your content and present some or all of your website in the language of your users. 
2. You can also use geolocation routing to restrict distribution of content to only the locations in which you have distribution rights. 
3. Another possible use is for balancing load across endpoints in a predictable, easy-to-manage way, so that each user location is consistently routed to the same endpoint.

4. Geolocation works by mapping IP addresses to locations. 
   * However, some IP addresses aren't mapped to geographic locations, so even if you create geolocation records that cover all seven continents, Amazon Route 53 will receive some DNS queries from locations that it can't identify. 
   * You can create a default record that handles both queries from IP addresses that aren't mapped to any location and queries that come from locations that you haven't created geolocation records for. 
   * If you don't create a default record, Route 53 returns a "no answer" response for queries from those locations.





### Disadvantage(s): DNS
1. Accessing a DNS server introduces a slight delay, although mitigated by caching described above.
1. DNS server management could be complex and is generally managed by governments, ISPs, and large companies.
1. DNS services have recently come under DDoS attack, preventing users from accessing websites such as Twitter without knowing Twitter's IP address(es).
