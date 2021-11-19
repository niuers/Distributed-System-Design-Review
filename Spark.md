# Overview

1. Programming directly in Hadoop MapReduce is difficult
1. Performance bottlenecks, or batch not fitting use cases
   * Three or more times teh data is put to the HDD (write followed by read) during a single MR job
   * HIVE query as SQL tool translates to 3-5 MR jobs
3. Better support iterative jobs, e.g. ML

# Spark Advantages
1. Lazy Computations
   * optimize the job before execution
1. In-memory data caching, 
   * scan HDD only once, then scan your RAM
1. Efficient Pipeline
   * Avoids the data hitting HDD by all means

# Two Main Abstractions of Spark
## RDD (Resilient Distributed Dataset)
1. Simpmle View: A collections of data items split into partitions and stored in memory on worker nodes of the cluster
2. Complex View: 
   * RDD is an interface for data transformation
   * RDD refers to the data stored either in persisted store (HDFS, Cassandra, Hbase, etc.) or in cache (memory, memory+disk, disks only, etc.) or in another RDD
   * Partitions are re-computed on failure or cache eviction
   * Meta stored for interface
      *  partitions: set of data splits associated with this RDD
      *  dependencies: list of parent RDDs involved in this computation
      *  compute: function to compute partition of RDD given the parent partitions from the dependencies
      *  preferred locations: where is the best location to put computations on this partition (data locality)
      *  partitioner: how the data is split into partitions
1. Two classes of operations
   * Transformation
   * Actions

### Transformation
1. Transformation only causes meta data change
### Actions
## DAG
1. Sequence of computations performed on data
2. Node: RDD partition
3. Edge: transformation on top of data
4. Acyclic: can't return to the older partition
5. Direct: transformation is an action that transitions data partition state (from A to B)
6. 

# Spark Architecture
## Spark Cluster
1. Put detailed graph here ???
### Driver
1. SparkContext
2. Driver translates RDD into the execution graph
3. Split graph into stages
4. Schedules tasks and controls their execution
5. Stores metadata about all the RDDs and their partitions
6. 
### Worker
1. Executors inside worker node
   * Stores the data in cache in JVM heap or on HDDs
   * Reads/Writes data from external sources
#### Executor Memory
1. JVM Heap/Safe/Shuffle/Unroll/Storage
2. Python Memory/Safe
3. Cache: 
   * Spark considers memory as a cache with LRU eviction rules. If "disk" is involved, data is evicted to disks
5. Runs tasks

# 
1. Application: Single instance of SparkContext (thread-safe) that stores some data processing logic and can schedule series of jobs; sequentially or in parallel,
2. Jobs: Complete set of transformation on RDD that finishes with action or data saving, triggered by the driver application
3. Stage: Set of transformations that can be pipelined and executed by a single independent worker. Usually it is app the transformations between 'read', 'shuffle', 'action', 'save'.
4. Task: Execution of the stage on a single data partition. Basic unit of scheduling.
5. 
## Shuffle in Spark
1. Hash Shuffle: before 1.2
2. Sort Shuffle
3. Tungsten Sort: optimized

## DataFrame
1. DataFrame is RDD with schema,  internally the same as RDD
2. Data stored in row-columnar format, 
3. Each column in each partition stores min-max values for partition pruning
4. Allows better compression ratio than RDD
5. faster performance for smaller subsets of columns
