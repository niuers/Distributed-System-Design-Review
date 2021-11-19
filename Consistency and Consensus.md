# Overview
1. The word consistency is terribly overloaded: 
   * replica consistency and the issue of eventual consistency that arises in asynchronously replicated systems (see “Problems with Repli‐ cation Lag” on page 161).
   * Consistent hashing is an approach to partitioning that some systems use for reba‐ lancing (see “Consistent Hashing” on page 204).
   * In the CAP theorem (see Chapter 9), the word consistency is used to mean linearizability (see “Linearizability” on page 324).
   * In the context of ACID, consistency refers to an application-specific notion of the database being in a “good state.”

# Consistency Guarantees
# Linearizability
## What Makes a System Linearizable?
## Relying on Linearizability
## Implementing Linearizable Systems
## The Cost of Linearizability

# Ordering Guarantees
## Ordering and Causality
## Sequence Number Ordering
## Total Order Broadcast
# Distributed Transactions and Consensus
## Atomic Commit and Two-Phase Commit (2PC)
## Distributed Transactions in Practice
## Fault-Tolerant Consensus
## Membership and Coordination Services

## Consistency Patterns

### Weak consistency
1. After a write, reads may or may not see it. A best effort approach is taken. 
2. This approach is seen in systems such as memcached. Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games. 
For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

### Eventual consistency
1. After a write, reads will eventually see it (typically within milliseconds). Data is replicated asynchronously.
2. This approach is seen in systems such as DNS and email. Eventual consistency works well in highly available systems.

### Strong consistency
1. After a write, reads will see it. Data is replicated synchronously.
2. This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.

