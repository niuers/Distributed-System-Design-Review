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
## The Unix Philosophy

# MapReduce and Distributed Filesystems
## MapReduce Job Execution
## Reduce-Side Joins and Grouping
## Map-Side Joins
## The Output of Batch Workflows
## Comparing Hadoop to Distributed Databases

# Beyond MapReduce
## Materialization of Intermediate State
## Graphs and Iterative Processing
## High-Level APIs and Languages

