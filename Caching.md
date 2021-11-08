
# Caching
1. Caching improves page load times and can reduce the load on your servers and databases. In this model, the dispatcher will first lookup if the request has been made before and try to find the previous result to return, in order to save the actual execution.
1. Databases often benefit from a uniform distribution of reads and writes across its partitions. Popular items can skew the distribution, causing bottlenecks. Putting a cache in front of a database can help absorb uneven loads and spikes in traffic.
   * DB access bottlenecks caused by too many concurrent requests

1. Please never do file-based caching, it makes cloning and auto-scaling of your servers just a pain. 
2. Improve performance (response time by reducing data access latency): reading from memory is 50-200x faster than reading from disk
3. Offload persistent storages by reducing number of trips to data sources
4. avoid the cost of repeatedly creating objects
5. share objects between threads
6. 
7. save money: can serve the same amount of traffic with fewer resources
8. Caching consists of: precalculating results (e.g. the number of visits from each referring domain for the previous day), pre-generating expensive indexes (e.g. suggested stories based on a user’s click history), and storing copies of frequently accessed data in a faster backend (e.g. Memcache instead of PostgreSQL.
9. In practice, caching is important earlier in the development process than load-balancing, and starting with a consistent caching strategy will save you time later on. It also ensures you don’t optimize access patterns which can’t be replicated with your caching mechanism or access patterns where performance becomes unimportant after the addition of caching (I’ve found that many heavily optimized Cassandra applications are a challenge to cleanly add caching to if/when the database’s caching strategy can’t be applied to your access patterns, as the datamodel is generally inconsistent between the Cassandra and your cache).

## Types
1. Application vs. database caching
   * There are two primary approaches to caching: application caching and database caching (most systems rely heavily on both).
1. Local Cache
   * Simple, performant, no serilization/deserialization overhead
   * Not fault-tolerant
   * Scale is difficult
   * products: EhCache, Google Guava, 
3. Replicated Cache
   * Each node access data locally
   * Best read performance, fault tolerant, linear performance scalability for reads
   * Poor write performance, additional network load, poor and limited scalability for writes, memory consumption
   * Products: InfiniteSpan, EhCache+Terracota, Oracle Coherence
5. Distributed Cache
   * A cache that partitions its data among all cluster nodes
   * resolving  known limitations of replicated cache on write
   * Data consistency and data loss
   * Failover involves promoting backup data to primary storage
   * Advantages: linear performance scalability for reads and writes; fault-tolerant
   * Increased latency of reads (roundtrip network, serialization/deserialization overhead)
   * Products: MongoDB, Redis, Cassandra
7. Remote Cache
   * A cache that is located remotely and should be accessed by a client
   * The client-server mode from most replicated/distributed cache
   * MemCached
   * 
9. Near Cache
   * A hybrid cache, it typically fronts a distributed cache or a remote cache with a local cache
   * It's best used for read-only data
   * Increases memory usage since the near cache items need to be stored in the memory of the member
   * reduces consistency
## IMDG (In-Memory Data Grid)
1. It is the in-memory distributed cache plus:
   * ability to support co-location of computations of with data in a distributed context and move computations to data
   * Distributed MPP processing based on standard SQL and/or MapReduce, that allows to effectively compute over data stored in-memory across the cluster
1. It's developed to respond to growing complexities of data processing
2. Adding distributed SQL and/or MapReduce type processing required a complete re-thinking of distributed caches, as the focus has shifted from pure data management to hybrid data and compute management.
3. 
### Client caching
Caches can be located on the client side (OS or browser), server side, or in a distinct cache layer.
### CDN
CDNs are considered a type of cache.

### Web server caching
1. Reverse proxies and caches such as Varnish can serve static and dynamic content directly. Web servers can also cache requests, returning responses without having to contact application servers.
1. Varnish, Squid, Nginx, 

### Database Cache
Your database usually includes some level of caching in a default configuration, optimized for a generic use case. Tweaking these settings for specific usage patterns can further boost performance.


### APllication
1. In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage. Since the data is held in RAM, it is much faster than typical databases where data is stored on disk. RAM is more limited than disk, so cache invalidation algorithms such as least recently used (LRU) can help invalidate 'cold' entries and keep 'hot' data in RAM.

1. Redis has the following additional features:
   * Persistence option
   * Built-in data structures such as sorted sets and lists
1. There are multiple levels you can cache that fall into two general categories: database queries and objects:
   * Row level
   * Query-level
   * Fully-formed serializable objects
   * Fully-rendered HTML
Generally, you should try to avoid file-based caching, as it makes cloning and auto-scaling more difficult.

#### Caching at the database query level
1. Whenever you query the database, hash the query as a key and store the result to the cache. This approach suffers from expiration issues:
   * Hard to delete a cached result with complex queries
   * If one piece of data changes such as a table cell, you need to delete all cached queries that might include the changed cell

#### Caching at the object level
1. See your data as an object, similar to what you do with your application code. Have your application assemble the dataset from the database into a class instance or a data structure(s):
   * Remove the object from cache if its underlying data has changed
   * Allows for asynchronous processing: workers assemble objects by consuming the latest cached object

1. Suggestions of what to cache:
   * User sessions
   * Fully rendered web pages
   * Activity streams
   * User graph data



## HTTP Caching
### CDN, Akamai

## Generate Static Content

### Precompute content
1. Homegrown + cron or Quartz
2. Spring Batch
3. Gearman
4. Hadooop
5. Google Data Protocal
6. Amazon Elastic MapReduce

## Distributed Caching
### Summary
1. It has built-in functionality to replicate data, shard data across servers, and locate proper server for each key.
2. It can have active/passive caches
   * We can warm-up the passive caches to fill the data when we bring them active to avoid 'cache-missing' in the initial time

### **Eviction Policies**
* We need to prevent stale data, and to save resources we only cache most valuable data
* TTL: set a time period before a cache entry is deleted automatically, used to prevent stale data
* Bounded FIFO
* Bounded LIFO
* Explicit Cache Invalidation
* LRU/LFU (least frequently used)
  * FB Thundering herd problem: where some popular user updates a post, we dump the old post, update the DB, and before we update the cache with the new post, thousands of requests coming in, cause 'cache miss' and DB issues.
  * They implement a lease a kind of backup cache that serves the old data while the new one is being updated

### Caching Strategies
1. When to update the cache: since you can only store a limited amount of data in cache, you'll need to determine which cache update strategy works best for your use case.


#### Cache Aside - most common
1. The application is responsible for reading and writing from storage. The cache does not interact with storage directly. The application does the following:
   * Look for entry in cache, resulting in a cache miss
   * Load entry from the database
   * Add entry to cache
   * Return entry
1. Memcached is generally used in this manner.
1. Subsequent reads of data added to cache are fast. Cache-aside is also referred to as **lazy loading**. Only requested data is cached, which avoids filling up the cache with data that isn't requested.
1. Disadvantage(s): cache-aside
   * Each cache miss results in three trips, which can cause a noticeable delay.
   * Data can become stale if it is updated in the database. This issue is mitigated by setting a **time-to-live (TTL)** which forces an update of the cache entry, or by using write-through.
   * When a node fails, it is replaced by a new, empty node, increasing latency.
   * It might have blocking behavior
   * Cache-aside may be perferrable when there are multiple cache updates triggered to the same storage from different cache servers

#### Read Through
1. The application treats cache as the main data store and reads data from it
2. 
#### Write-through  (for write heavy)
1. The application uses the cache as the main data store, reading and writing data to it, while the cache is responsible for reading and writing to the database:
   * Application adds/updates entry in cache
   * Cache synchronously writes entry to data store
   * Return
   * Consistency vs. Performance
1. Write-through is a slow overall operation due to the write operation, but subsequent reads of just written data are fast. Users are generally more tolerant of latency when updating data than reading data. Data in the cache is not stale.
1. Disadvantage(s): write through
   * When a new node is created due to failure or scaling, the new node will not cache entries until the entry is updated in the database. Cache-aside in conjunction with write through can mitigate this issue.
   * Most data written might never be read, which can be minimized with a TTL.

#### Write-behind (for write heavy)
1. In write-behind, the application does the following:
   * Add/update entry in cache Use event-queue to write to DB
   * Asynchronously write entry to the data store, improving write performance
   * Consistency vs. Performance
1. It can deliver considerablely higher throughput and reduced latency compared to write-through
2. Implication of write-behind is that DB updates occur outside of cache transaction
3. Disadvantage(s): write-behind
   * There could be data loss if the cache goes down prior to its contents hitting the data store.
   * It is more complex to implement write-behind than it is to implement cache-aside or write-through.
   * It can conflict with an external update

#### Refresh-ahead
1. You can configure the cache to automatically and asynchronously refresh(reload) any recently accessed cache entry from the cache loader prior to its expiration.
2. Refresh-ahead can result in reduced latency vs read-through if the cache can accurately predict which items are likely to be needed in the future.
1. Disadvantage(s): refresh-ahead
   * Not accurately predicting which items are likely to be needed in the future can result in reduced performance than without refresh-ahead.


6. Replication
7. Peer-To-Peer P2P
   * Decentralized
   * No special nodes
   * Nodes can join and leave as they please
8. Products
   * memcached
      * Very fast, simple, key-value, 
      * Not replicated, so 1/N chance for local access in cluster
   * EHCache
   * JBoss Cache
   * OSCache
9. You can run out of RAM for the cache   

### Cache Consistency
1. Writes: make sure for user who's writing data get that fresh data immediately
   * FB posts wrong image, don't want too long delay between the time user posts image and user sees the image
3. While caching is fantastic, it does require you to maintain consistency between your caches and the source of truth (i.e. your database), at risk of truly bizarre applicaiton behavior. Solving this problem is known as cache invalidation.
4. If you’re dealing with a single datacenter, it tends to be a straightforward problem, but it’s easy to introduce errors if you have multiple codepaths writing to your database and cache (which is almost always going to happen if you don’t go into writing the application with a caching strategy already in mind). At a high level, the solution is: each time a value changes, write the new value into the cache (this is called a write-through cache) or simply delete the current value from the cache and allow a read-through cache to populate it later (choosing between read and write through caches depends on your application’s details, but generally I prefer write-through caches as they reduce likelihood of a stampede on your backend database).

Invalidation becomes meaningfully more challenging for scenarios involving fuzzy queries (e.g if you are trying to add application level caching in-front of a full-text search engine like SOLR), or modifications to unknown number of elements (e.g. deleting all objects created more than a week ago).

In those scenarios you have to consider relying fully on database caching, adding aggressive expirations to the cached data, or reworking your application’s logic to avoid the issue (e.g. instead of DELETE FROM a WHERE..., retrieve all the items which match the criteria, invalidate the corresponding cache rows and then delete the rows by their primary key explicitly).


## Cache Disadvantages
1. Memory size is limited and can become unacceptablely large
1. Synchronization/Consistency: Need to maintain consistency between caches and the source of truth such as the database through cache invalidation.
1. Durability: Cache invalidation is a difficult problem, there is additional complexity associated with when to update the cache.
1. Need to make application changes such as adding Redis or memcached.
2. Only work for IO bound applications ? 
3. Scalability? 



## Website Cache
1. .html: Craiglist saves data as .html and re-use them to improve performance instead of in XML, MySQL and generate pages dynamically
   * It increases storage cost
   * You also have to store the same data somewhere in DB if you allow edit. So there's redundancy.
   * Another redundancy: same HTML tags in every single page 
   * It's very hard to change the layout etc. of these pages now.

## Cached Database Queries
1. Whenever you do a query to your database, you store the result dataset in cache.
   * MySQL Query Cache: disabled-by-default since MySQL 5.6 (2013) as it is known to not scale with high-throughput workloads on multi-core machines.
1. This pattern has several issues. 
   * The main issue is the expiration. It is hard to delete a cached result when you cache a complex query. 
   * When one piece of data changes you need to delete all cached queries who may include that table cell.

## Cached Objects
1. It makes asynchronous processing of different parts of the object possible

### Some ideas of objects to cache:
1. user sessions (never use the database!)
1. fully rendered blog articles
1. activity streams
1. user<->friend relationships 


### Redis
1. Redis can do several hundreds of thousands of read operations per second when being hosted on a standard server. 
2. Also writes, especially increments, are very, very fast.
3. Redis has extra database-features like **persistence** and the built-in data structures like lists and sets. So you can use it as a Database.

### Amazon ElastiCache
1. It is a web service that makes it easy to set up, manage, and scale a distributed in-memory data store or cache environment in the cloud
2. Amazon ElastiCache works with both the Redis and Memcached engines
3. 
