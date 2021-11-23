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
## Reduce-Side Joins and Grouping
## Map-Side Joins
## The Output of Batch Workflows
## Comparing Hadoop to Distributed Databases

# Beyond MapReduce
## Materialization of Intermediate State
## Graphs and Iterative Processing
## High-Level APIs and Languages

