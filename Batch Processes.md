# Overview
1. In reality, integrating disparate systems is one of the most impor‐ tant things that needs to be done in a nontrivial application.
2. systems that store and process data can be grouped into two broad categories:
   * Systems of record
      * A system of record, also known as source of truth, holds the authoritative version of your data.
   * Derived Data Systems
      * Data in a derived system is the result of taking some existing data from another system and transforming or processing it in some way
      * A classic example is a cache
      * Denormalized values, indexes, and materialized views also fall into this category. 
      * In recommendation systems, predictive summary data is often derived from usage logs

1. Not all systems make a clear distinction between systems of record and derived data in their architecture, but it’s a very helpful distinction to make, because it clarifies the dataflow through your system: it makes explicit which parts of the system have which inputs and which outputs, and how they depend on each other.

# Batch Processing with Unix Tools
## Simple Log Analysis
### Chain of commands versus custom program
#### Sorting versus in-memory aggregation
1. The Ruby script keeps an in-memory hash table of URLs, where each URL is mapped to the number of times it has been seen. The Unix pipeline example does not have such a hash table, but instead relies on sorting a list of URLs in which multiple occurrences of the same URL are simply repeated.
2.  For small to mid-sized websites, the required memory is small enough for in-memory hash table
3.  Otherwise, the sorting approach has advantage that it make efficient use of disks. It's the same principle as "SSTables and LSM-Trees":
   * chunks of data can be sorted in memory and written out to disk as segment files, and then multiple sorted segments can be merged into a larger sorted file. Mergesort has sequential access patterns that perform well on disks.
   * The simple chain of Unix commands (sort spill to disk, parallelize across cores) easily scales to large datasets, without running out of memory. The bottleneck is likely to be the rate at which the input file can be read from disk.

## The Unix Philosophy
1. This approach—automation, rapid prototyping, incremental iteration, being friendly to experimentation, and breaking down large projects into manageable chunks— sounds remarkably like the Agile and DevOps movements of today. Surprisingly
2. What does Unix do to enable this composability?
### A uniform Interface
1. In Unix, that interface is a file (or, more precisely, a file descriptor). A file is just an ordered sequence of bytes.
   * By convention, many (but not all) Unix programs treat this sequence of bytes as ASCII text.
3. Another example of a uniform interface is URLs and HTTP
### Separation of logic and wiring
1. Another characteristic feature of Unix tools is their use of standard input (stdin) and standard output (stdout). If
2. the program doesn’t know or care where the input is coming from and where the output is going to. (One could say this is a form of loose coupling, late binding [15], or inversion of control [16].)
3. However, there are limits to what you can do with stdin and stdout. Programs that need multiple inputs or outputs are possible but tricky
### Transparency and experimentation
1. The input files to Unix commands are normally treated as immutable
2. You can end the pipeline at any point, pipe the output into less, and look at it to see if it has the expected form
3. You can write the output of one pipeline stage to a file and use that file as input to the next stage. This allows you to restart the later stage without rerunning the entire pipeline.
4. However, the biggest limitation of Unix tools is that they run only on a single machine—and that’s where tools like Hadoop come in.


# MapReduce and Distributed Filesystems
1. A single MapReduce job is comparable to a single Unix process: it takes one or more inputs and produces one or more outputs.
2. In Hadoop’s implementation of Map‐ Reduce, that filesystem is called HDFS, Object storage services such as Amazon S3 are similar:
   * One difference is that with HDFS, computing tasks can be scheduled to run on the machine that stores a copy of a particular file, whereas object stores usually keep storage and computation separate. Reading from a local disk has a performance advantage if network bandwidth is a bottleneck. Note however that if erasure coding is used, the locality advantage is lost, because the data from several machines must be combined in order to reconstitute the original file
1. HDFS is based on the shared-nothing principle, in contrast to the shared-disk approach of Network Attached Storage (NAS) and Storage Area Network (SAN) architectures.
   * Shared-disk storage is implemented by a central‐ ized storage appliance, often using custom hardware and special network infrastruc‐ ture such as Fibre Channel.
1. HDFS consists of a daemon process running on each machine, exposing a network service that allows other nodes to access files stored on that machine. A central server called the NameNode keeps track of which file blocks are stored on which machine
2. In order to tolerate machine and disk failures, file blocks are replicated on multiple machines.

## MapReduce Job Execution
1. MapReduce is a programming framework with which you can write code to process large datasets in a distributed filesystem like HDFS
2. Steps:
   * Read a set of input files, and break it up into records.
   * Map: Call the mapper function to extract a key and value from each input record.
   * Sort all of the key-value pairs by key: the output from the mapper is always sorted before it is given to the reducer.
   * Call the reducer function to iterate over the sorted key-value pairs.
### Distributed execution of MapReduce
1. The MapReduce scheduler tries to run each mapper on one of the machines that stores a replica of the input file, provided that machine has enough spare RAM and CPU resources to run the map task. This principle is known as **putting the com‐ putation near the data**: it saves copying the input file over the network, reducing network load and increasing locality.
2. While the number of map tasks is determined by the number of input file blocks, the number of reduce tasks is configured by the job author (it can be different from the number of map tasks)
3. To ensure that all key-value pairs with the same key end up at the same reducer, the framework uses a hash of the key to determine which reduce task should receive a particular key-value pair
4. The key-value pairs must be sorted, but the dataset is likely too large to be sorted with a conventional sorting algorithm on a single machine. Instead, the sorting is performed in stages. 
   * First, each map task partitions its output by reducer, based on the hash of the key. Each of these partitions is written to a sorted file on the mapper’s local disk, using a technique similar to what we discussed in SSTabels and LSM-Trees.
   * The reducers connect to each of the mappers and download the files of sorted key-value pairs for their partition. The process of partitioning by reducer, sorting, and copying data partitions from mappers to reducers is known as the **shuffle**
1. The reducer is called with a key and an iterator that incrementally scans over all records with the same key (which may in some cases not all fit in memory)
### MapReduce workflows
1. it is very common for MapReduce jobs to be chained together into workflows
2. Chained MapReduce jobs are therefore less like pipelines of Unix commands
3. various workflow schedulers for Hadoop have been developed, including Oozie, Azkaban, Luigi, Airflow, and Pinball
## Reduce-Side Joins and Grouping
1. When a MapReduce job is given a set of files as input, it reads the entire content of all of those files; a database would call this operation a full table scan.
2. If you only want to read a small number of records, a full table scan is outrageously expensive compared to an index lookup as in DB query.
   * If the DB query involves joins, it may require multiple index lookups
1. When we talk about joins in the context of batch processing, we mean resolving all occurrences of some association within a dataset. For
   * For example, we assume that a job is processing the data for all users simultaneously, not merely looking up the data for one particular user (which would be done far more efficiently with an index)
1. In order to achieve good throughput in a batch process, the computation must be (as much as possible) local to one machine. Making random-access requests (e.g. to DB) over the network for every record you want to process is too slow. 
2. Thus, a better approach would be to take a copy of the user database and to put it in the same distributed filesystem as the log of user activity events.
### Sort-merge joins
1. Since the reducer processes all of the records for a particular user ID in one go, it only needs to keep one user record in memory at any one time, and it never needs to make any requests over the network. This algorithm is known as a sort-merge join, since mapper output is sorted by key, and the reducers then merge together the sorted lists of records from both sides of the join
### Bringing related data together in the same place
1. One way of looking at this architecture is that mappers “send messages” to the reduc‐ ers. When a mapper emits a key-value pair, the key acts like the destination address to which the value should be delivered. all key-value pairs with the same key will be delivered to the same desti‐ nation (a call to the reducer)
2. Using the MapReduce programming model has separated the physical network com‐ munication aspects of the computation (getting the data to the right machine) from the application logic (processing the data once you have it)
3. MapReduce trans‐ parently retries failed tasks without affecting the application logic

### Group By
1. Besides joins, another common use of the “bringing related data to the same place” pattern is grouping records by some key (as in the GROUP BY clause in SQL)
2. The simplest way of implementing such a grouping operation with MapReduce is to set up the mappers so that the key-value pairs they produce use the desired grouping key.
3. Another common use for grouping is collating all the activity events for a particular user session, in order to find out the sequence of actions that the user took—a pro‐ cess called sessionization
### Handling skew
1. The pattern of “bringing all records with the same key to the same place” breaks down if there is a very large amount of data related to a single key. 
2. Such disproportionately active database records are known as linchpin objects [38] or hot keys.
3. If a join input has hot keys, there are a few algorithms you can use to compensate. For example, the skewed join method in Pig.
4. We can spread the work of handling the hot key over several reducers, which allows it to be parallelized better, at the cost of having to replicate the other join input to multiple reducers. The **sharded join** method in Crunch is similar, but requires the hot keys to be specified explicitly rather than using a sampling job. This technique is also very similar to one we discussed in “Skewed Workloads and Relieving Hot Spots” on page 205, using randomization to alleviate hot spots in a partitioned data‐ base.

## Map-Side Joins
1. The reduce-side approach has the advantage that you do not need to make any assumptions about the input data: whatever its properties and structure, the mappers can prepare the data to be ready for joining. However, the downside is that all that sorting, copying to reducers, and merging of reducer inputs can be quite expensive
2. if you can make certain assumptions about your input data, it is possible to make joins faster by using a so-called map-side join. 
3. This approach uses a cut-down MapReduce job in which there are no reducers and no sorting. 
4. Instead, each mapper simply reads one input file block from the distributed filesystem and writes one output file to the filesystem—that is all.
### Broadcast hash joins
1. The simplest way of performing a map-side join applies in the case where a large dataset is joined with a small dataset. In particular, the small dataset needs to be small enough that it can be loaded entirely into memory in each of the mappers
2. the word broadcast reflects the fact that each mapper for a partition of the large input reads the entirety of the small input (so the small input is effectively “broadcast” to all partitions of the large input), and the word hash reflects its use of a hash table. 
3. This join method is supported by Pig (under the name “replicated join”), Hive (“MapJoin”), Cascading, and Crunch. It is also used in data warehouse query engines such as Impala
4. Instead of loading the small join input into an in-memory hash table, an alternative is to store the small join input in a read-only index on the local disk
### Partitioned hash joins
1. If the inputs to the map-side join are partitioned in the same way, then the hash join approach can be applied to each partition independently.
2. If the partitioning is done correctly, you can be sure that all the records you might want to join are located in the same numbered partition, and so it is sufficient for each mapper to only read one partition from each of the input datasets. This has the advantage that each mapper can load a smaller amount of data into its hash table.
3. This approach only works if both of the join’s inputs have the same number of parti‐ tions, with records assigned to partitions based on the same key and the same hash function.
### Map-side merge joins
1. Another variant of a map-side join applies if the input datasets are not only parti‐ tioned in the same way, but also sorted based on the same key
### MapReduce workflows with map-side joins
1. When the output of a MapReduce join is consumed by downstream jobs, the choice of map-side or reduce-side join affects the structure of the output.
## The Output of Batch Workflows
1. Where does batch processing fit in? It is not transaction processing, nor is it analyt‐ ics. It is closer to analytics, in that a batch process typically scans over large portions of an input dataset. However, a workflow of MapReduce jobs is not the same as a SQL query used for analytic purposes (see “Comparing Hadoop to Distributed Data‐ bases” on page 414). The output of a batch process is often not a report, but some other kind of structure.

### Building search indexes
### Key-value stores as batch process output
1. use the client library for your favorite database directly within a mapper or reducer, and to write from the batch job directly to the database server, one record at a time. but it is a bad idea for several reasons:
   * making a network request for every single record is orders of magnitude slower than the normal throughput of a batch task
   * If all the mappers or reducers concurrently write to the same output database, with a rate expected of a batch process, that database can easily be overwhelmed
   * writing to an external system from inside a job produces externally visible side effects that cannot be hidden in all-or-nothing
5. A much better solution is to build a brand-new database inside the batch job and write it as files to the job’s output directory in the distributed filesystem, just like the search indexes in the last section. and they can be loaded in bulk into servers that handle read-only queries

### Philosophy of batch process outputs
1. By treat‐ ing inputs as immutable and avoiding side effects (such as writing to external data‐ bases), batch jobs not only achieve good performance but also become much easier to maintain:
   * If you introduce a bug into the code and the output is wrong or corrupted, you can simply roll back to a previous version of the code and rerun the job, and the output will be correct again. Or, even simpler, you can keep the old output in a different directory and simply switch back to it. Databases with read-write trans‐ actions do not have this property:
   * feature development can proceed more quickly than in an environment where mistakes could mean irreversible damage. This principle of minimizing irreversibility
   * If a map or reduce task fails, the MapReduce framework automatically re- schedules it and runs it again on the same input. This automatic retry is only safe because inputs are immutable and outputs from failed tasks are discarded by the MapReduce framework.
   * The same set of files can be used as input for various different jobs, including monitoring jobs that calculate metrics and evaluate whether a job’s output has the expected characteristics (for
   * Like Unix tools, MapReduce jobs separate logic from wiring (configuring the input and output directories), which provides a separation of concerns and ena‐ bles potential reuse of code
1. 
## Comparing Hadoop to Distributed Databases

# Beyond MapReduce
## Materialization of Intermediate State
## Graphs and Iterative Processing
## High-Level APIs and Languages

