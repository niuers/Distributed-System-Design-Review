# Scaling Memcache at Facebook

## Infrastructure Requirements at FB
1. Near real-time communication
2. Aggregate content on-the-fly from multiple sources
3. Be able to access and update very popular shared content
4. Scale to process millions of user requests per second
5. Support rapid deployment of new features

## Memcached at FB
1. Trillions of Items
2. Billions of requests per second
3. Network attached in-memory hash table
   * Support LRU based eviction
4. The problems of high fan-out, and sheer volume of communications at FB, the abilities to fetch a lot of small data very quickly, dictate a need of very fast, scalable, in-memory key-value store.
5. We partition the data in cache into 'pools', some are high-turn data sets, may stay in the cache for hours, a few days, other data can stay longer because the cost of fetching is high.
## Issues
### Single Front-End Cluster
1. Read heavy workload
2. wide fan-out
3. handling failure

### Multiple Front-End Clusters
1. Controlling data replication
2. Data consistency

### Multiple Geographic Data Centers
1. Data Consistency


### Why Separate Cache? 
1. High fan-out and multiple rounds of data fetching


## Scaling MemCache in 4 Easy Steps

1. If we need more read capacity

### A few Memcache Servers
10s of servers and millions of operations per second

1. How do we store data?
   * Demand filled look aside cache
      * If miss, get from DB, then re-fill the cache
   * Common case is data is available in cache
2. Handling Updates
   * Memcache needs to be invalidated (by web server request) after the DB write
   * Prefer deletes to sets
      * Idempotent
      * Demand filled
   * Up to web application to specify which keys to invalidate after DB update
3. Problems with look-aside cache
   * Stale Sets: MC and DB inconsistency
      * Extend Memcache protocl with 'leases'
   * Thudering herds: 
4. We also need to provision our memcache in the worst case scenario,  (tail distribution case)
### many Memcache servers in one cluster
100s of servers and 10s of millions of operations per second
A cluster is a set of web servers plus a set of cache servers.

1. Items  are distributed across Memcache servers through consistent hashing on the key
2. Individual items are rarely accessed very frequently so over replication doesn't make sense
3. All web servers talk to all memcache servers
   * Accessing 100s of memcache servers are common to process a user request
5. Big problems with All-to-All communication
   * within a very short period of time, all web servers will access all cache servers simutaneously, this puts enormous strain on network infrstructure
6. Incast congestion problem
   * When we do wide parallel fetch to many cache servers, the responses go back to web server, they overwhelm shared networking resources at the client side, not the server side, and you start to see packet drops.
   * Solution: limit the number of outstanding requests with a sliding window
      * Larger window cause results in more congestion
      * Smaller window results in more roundtrips in the network
### Many Memcache servers in multiple Front-End clusters
1000s of servers and 100s of millions of operations per second
1. All-to-All limits horizontal scaling
2. Multiple memcache clusters front one DB installation (all clusters connect to one storage cluster)
   * Have to keep the cache consistent
   * have to manage over-replication of data
3. Solution: DBs invalidate caches
   * Cached data must be invalidated after DB updates
   * Solution: 
      * Tail the MySQL commit log and issue deletes based on transactions that have been committed
      * Allows caches to be resynchronized if there's a problem
4. Invalidation Pipeline
   * There are too many packets in the system
   * We add a 'memcache routers' before the DB instances, which send out the 'delete' requests to Front-End clusters, each FE cluster has its own 'memcache routers', which broadcast out internally.
   * Now the inter-cluster bandwith is much smaller than the intra-cluster bandwidth. We minimize bandwith and maximize packet density
   * Each stage buffers deletes in case downstream component is down
   *  


### Geographically distributed clusters
1000s of servers and 1 billions of operations per second
They have a single master set of DBs. 
1. Primarily motivation is fault tolerance
2. Writes in non-master
   * Database update directly in master (even it means to cross-region), then  we delete from local memcache
   * Race between DB replication (MySQL replication) process and subsequent DB read (in case of miss on cache) process 
   * It could potentially set stale value on cache
   * Solution: Set a remote markers, set a special flag to indicate whether a race is likely

## Lessons Learned
1. Push the complexity into the client side library (residing on every web server) whenever possible
   * Notice there's no server-server communication in the system
3. Operational efficiency is as important as performance
   * e.g. multiple routing when we do invalidation pipeline simplifies cluster management, it can be slower because we go through multiple steps, but it keeps the configuration tight, local,easy to deal with.
5. Separating cache and persistent store allows them to be scaled independently
6. 
