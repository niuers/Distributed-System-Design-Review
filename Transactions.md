# Overview

1. A transaction is a way for an application to group several reads and writes together into a logical unit.
2. Conceptually, all the reads and writes in a transaction are executed as one operation: either the entire transaction succeeds (commit) or it fails (abort, rollback).
3. By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these safety guarantees).

# The Slippery Concept of a Transaction
## ACID
## Single-Object and Multi-Object Operations

# Weak Isolation Levels
## Read Committed
## Snapshot Isolation and Repeatable Read
## Preventing Lost Updates
## Write Skew and Phantoms

# Serializability

## Actual Serial Execution
## Two-Phase Locking (2PL)
## Serializable Snapshot Isolation (SSI)

