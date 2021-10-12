
# Caching
1. .html: Craiglist saves data as .html and re-use them to improve performance instead of in XML, MySQL and generate pages dynamically
   * It increases storage cost
   * You also have to store the same data somewhere in DB if you allow edit. So there's redundancy.
   * Another redundancy: same HTML tags in every single page 
   * It's very hard to change the layout etc. of these pages now.
3. MySQL Query Cache: disabled-by-default since MySQL 5.6 (2013) as it is known to not scale with high-throughput workloads on multi-core machines.
4. memcached
   * You can run out of RAM for the cache
   * We can expire objects based on when they are put in (FIFO), or recently used. 
