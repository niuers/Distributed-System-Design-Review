# Facebook TAO
1. TAO is FB's distributed data store for social graph
2. 
## Example Post
1. Photo Node
   * EXIF information on a photo: camera exposure, date/time the image was captured, and even GPS location
3. Text
4. Comment Node
5. Like Node
6. Other information: location, time, author

## Requirements of the Solution
1. Efficiency (reduce cost) at Scale
   * Needs to dynamically load/render a HTML post every time a person refresh it. So it has to fetch the data from graph, and combine them into HTML.
   * 1 billion queries/second into TAO
   * Many petabytes of data
1. Low Read Latency because of Dynamic resolution of data dependencies
   * Can't go from one DC to another in each of these query rounds (too slow)
   * So we always read from local DC
   * Have 500 reads for every write, so can tolerate high write latency
1. Timeliness of Writes
   * Read what you wrote consistency in each DC: easier for development of application
1. High Read Availability to render the page

## Efficiency at Scale and Low Read Latency
### Basic Configuration
1. Separate into different pieces so they can be sized and scaled independently
   * Web servers are stateless
   * Sharded caches
   * Sharded DBs
3. Scale the read by adding caches
4. Scale the storage and write capacity by adding MySQL DBs

### Issues 
1. The simple horizontal scale out starts to hit bottlenecks when you get to the size of data center or a set of buildings that are coupled together.
1. Web Servers
   * Inefficient Failure detection 
      * If every web server is trying to discover failures in the cache server independently so the number of errors that are required to discover failures gets larger as the top level gets bigger. 
   * Many Switch traversals
      * Even the servers are in the same building, the costs of going close/far are different
      * Switches that are actually cost effective only have a limited number of ports, so start to have inefficient usage there
1. Cache
   * Many  Open sockets
   * Lots of hot spots as shards get smaller and smaller
3. Solution
   * Have a subset of web servers talk to a subset of caches
      * Problem: if we just spray the query randomly from web server, the user query may not hit the same cache each time, and we have to re-populate the cache each time
      * We make sticky user sessions with web server so they always (for some time) hit the same cache
   * Problem: we distribute our read/write logic (in cache), it's harder to prevent **Thundering Herds** problem.
      * Distributed write is more complicate, has latency problem, 
   * Solution: Add another layer of cache (Leader Cache)
      * Responsible for coordinating the writes and updating the caches in the followers and protecting DBs against Thundering Herds.
      * Used to reduce the temperature of hot spots. 
## Timeliness of Writes
1. Want to distribute writes out everywhere as soon as possible
2. Want to provide read what you wrote consistency
3. Solution: Use a write-through cache instead of lookaside cache
4. The leader cache send request to follower cache to ask them to 'get a refill of something' (instead of sending the data to followers). This way we make the consistency messages **Idempotent**, so in multi-datacenter cases, we dont have to guarantee that the cache consistency messages are delivered only once, or in the correct order. If we have some sort of failure, we can replay cache consistency messages to get ourselves back to a good state.
5. MySQL asynchronous replication to keep the data in sync

## High Read Availability
1. Cost of fail over read ? 
2. MetaStable state: We have failures because of out of capacity. Where we may never have the opportunity to catch up.
3. What's the fail over path? 
