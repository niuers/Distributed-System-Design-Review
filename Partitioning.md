
# Overview
## Purpose of Partitioning
1. Scalability: a large dataset can be distributed across many disks, and the query load can be distributed across many processors
2. 

## Problems to Consider
1. How to rebalance data when add or remove nodes?
2. How to route requests to the right partitions and execute queries?

## Comparison
Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
Products | MongoDB, Elasticsearch, SolrCloud, HBase, Bigtable, Cassandra, Riak, CouchBase | 3


# Partitioning and Replication
1. Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodes.
2. The choice of partitioning scheme is mostly independent of the choice of replication scheme.

# Partitioning of Key-Value Data
# Partitioning and Secondary Indexes
# Rebalancing Partitions
# Request Routing
