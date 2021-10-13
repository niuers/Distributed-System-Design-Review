
# Caching

1. Please never do file-based caching, it makes cloning and auto-scaling of your servers just a pain. 

## HTTP Caching
### Reverse Proxy
1. Varnish, Squid, Nginx, 
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
1. Write-through
   * Directly write to DB
3. Write-behind
   * Use even-queue to write to DB
4. Eviction Policies
   * TTL 
   * Bounded FIFO
   * Bounded LIFO
   * Explicit Cache Invalidation
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

### Memcached 
   * You can run out of RAM for the cache
   * We can expire objects based on when they are put in (FIFO), or recently used. 

### Redis
1. Redis can do several hundreds of thousands of read operations per second when being hosted on a standard server. 
2. Also writes, especially increments, are very, very fast.
3. Redis has extra database-features like **persistence** and the built-in data structures like lists and sets. So you can use it as a Database.
