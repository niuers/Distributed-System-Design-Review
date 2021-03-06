# Overview

1. A transaction is a way for an application to group several reads and writes together into a logical unit.
  1. Conceptually, all the reads and writes in a transaction are executed as one operation: either the entire transaction succeeds (commit) or it fails (abort, rollback). If  it  fails,  the  application  can  safely  retry.
  2.  With  transactions,  errorhandling becomes much simpler for an application, because it doesn’t need to worryabout partial failure
2. Transactions are created to simplify the programming  model  for  applications accessing  a database. By using transactions, the application is free to ignore certain potential error scenarios and concurrency issues, because the database takes care of them instead (we call these safety guarantees).
4. Transactions are an abstraction layer that allows an application to pretend that certain concurrency problems and certain kinds of hardware and software faults don’t exist. A large class of errors is reduced down to a simple transaction abort, and the application just needs to try again.

# The Slippery Concept of a Transaction
1. Nonrelational (NoSQL) databases aimed to improve upon the relational status quo by offering a choice of new data models, ***and by including replication  and partitioning by default**. Transactions were the main casualty of this movement: many abandoned transactions entirely, or redefined the word to describe a much weaker set of guarantees than had previously been understood

## ACID
1.The safety guarantees provided by transactions are often described by the well-known acronym ACID, which stands for Atomicity, Consistency, Isolation, and Durability
1. Systems that do not meet the ACID criteria are sometimes called BASE, which stands for Basically Available, Soft State, and Eventual consistency

### Atomicity
1. In the context of ACID, atomicity is not about concurrency (it's covered under Isolation), ACID atomicity describes what happens if a client wants to make several writes, but a fault occurs after some of the writes have been processed. 
2. If the writes are grouped together into an atomic transaction, and the transaction cannot be completed (committed) due to a fault, then the transaction is aborted and the database must discard or undo any writes it has made so far in that transaction.
3. Atomicity  simplifies  this  problem:  if  a  transaction  was  aborted,  the application can be sure that it didn’t change anything, so it can safely be retried
4. The ability to abort a transaction on error and have all writes from that transaction discarded is the defining feature of ACID atomicity
5. The database saves you from having to worry about partial failure, by giving an all-or-nothing guarantee.

### Consistency
1. In the context of ACID, consistency refers to an application-specific notion of the database being in a “good state.”
2. The idea of ACID consistency is that you have certain statements about your data (invariants) that must always be true.  If a transaction starts with adatabase that is valid according to these invariants, and any writes during the transac‐tion preserve the validity, then you can be sure that the invariants are always satisfied.
3. However, this idea of consistency depends on the application’s notion of invariants, and it’s the application’s responsibility to define its transactions correctly so that they preserve consistency. This is not something that the database can guarantee.
  1. if  youwrite  bad  data  that  violates  your  invariants,  the  database  can’t  stop  you.  (Some  spe‐cific  kinds  of  invariants  can  be  checked  by  the  database,  for  example  using  foreignkey  constraints  or  uniqueness  constraints.  However,  in  general,  the  applicationdefines what data is valid or invalid—the database only stores it.)
5. Atomicity, isolation, and durability are properties of the database, whereas consistency (in the ACID sense) is a property of the application.

### Isolation
1. If read/write are accessing the same database records, you can run into concurrency problems (race conditions)
2. Isolation in the sense of ACID means that concurrently executing transactions are isolated from each other: they cannot step on each other’s toes.
3. The classic database textbooks formalize isolation as *serializability*, which means that each transaction can pretend that it is the only transaction running on the entire database. The database ensures that when the transactions have committed, the result is the same as if they had run serially (one after another), even though in reality they may have run concurrently.
1. However, in practice, serializable isolation is rarely used, because it carries a performance penalty. Some popular databases, such as Oracle 11g, don’t even implement it.
2. Concurrently running transactions shouldn’t interfere with each other. For example, if one transaction makes several writes, then another transaction should see either all or none of those writes, but not some subset.

### Durability
1. Durability is the promise that once a transaction has committed successfully, any data it has written will not be forgotten, even if there is a hardware fault or the database crashes.
2. In a single-node database, durability typically means that the data has been written to nonvolatile storage such as a hard drive or SSD. It usually also involves a write-ahead log or similar, which allows recovery in the event that the data structures on disk are corrupted.
3. In a replicated database, durability may mean that the data has been successfully copied to some number of nodes. In order to provide a durability guarantee, a database must wait until these writes or replications are complete before reporting a transaction as successfully committed.

## Single-Object and Multi-Object Operations
1. in ACID, atomicity and isolation describe what the database should do if aclient makes several writes within the same transaction
2. These definitions assume that you want to modify several objects (rows, documents,records)  at  once.  Such  multi-object  transactions  are  often  needed  if  several  pieces  ofdata need to be kept in sync.
3. Not all applications are susceptible to all those problems: an application with very simple access patterns, such as reading and writing only a single record, can probably manage without transactions.
4. However, for more complex access patterns, transactions can hugely reduce the number of potential error cases you need to think about.
5. in ACID, atomicity and isolation describe what the database should do if a client makes several writes within the same transaction
   * These definitions assume that you want to modify several objects (rows, documents, records) at once (e.g. check for unread email while having a counter of unread messages). Such multi-object transactions are often needed if several pieces of data need to be kept in sync.
6. Multi-object transactions require some way of determining which read and write operations belong to the same transaction. In relational databases, that is typically done based on the client’s TCP connection to the database server: on any particular connection, everything between a BEGIN TRANSACTION and a COMMIT statement is considered to be part of the same transaction.
7. On the other hand, many nonrelational databases don’t have such a way of grouping operations together. ven  if  there  is  a  multi-object  API  (for  example,  a  key-valuestore may have a multi-putoperation that updates several keys in one operation), thatdoesn’t necessarily mean it has transaction semantics: the command may succeed forsome keys and fail for others, leaving the database in a partially updated state.

### Single-Object Writes
1. Atomicity and isolation also apply when a single object is being changed. For example, imagine you are writing a 20 KB JSON document to a database
2. To avoid confusing, storage engines almost universally aim to provide atomicity and isolation on the level of a single object (such as a key-value pair) on one node. *Atomicity can be implemented using a log for crash recovery (see “Making B-trees reliable” on page 82), and isolation can be implemented using a lock on each object (allowing only one thread to access an object at any one time)*.
3. Single-Object Operations
   * Atomic increment: removes the need for a read-modify-write cycle, (2nd client reads the data before 1st client's write phase)
   * Compare-and-set: allows a write to happen only if the value has not been concurrently changed by someone else
1. These single-object operations are useful, as they can prevent lost updates when several clients try to write to the same object concurrently
2. However, they are not transactions in the usual sense of the word. 
3. A transaction is usually understood as a mechanism for grouping multiple operations on multiple objects into one unit of execution.

### The need for multi-object transactions
1. Many distributed datastores have abandoned multi-object transactions because they are difficult to implement across partitions, and they can get in the way in some scenarios where very high availability or performance is required.
2. However, in many other cases writes to several different objects need to be coordinated
   * In a relational data model, a row in one table often has a foreign key reference to a row in another table. Multi-object transactions allow you to ensure that these references remain valid.
   * In a document data model, the fields that need to be updated together are often within the same document, which is treated as a single object—no multi-object transactions are needed when updating a single document. However,document databases lacking join functionality also encourage denormalization. Transactions are very useful in this situation to prevent denormalized data from going out of sync.
   * The secondary indexes also need to be updated every time you change a value. without transaction isolation, it’s possible for a record to appear in one index but not another, because the update to the second index hasn’t happened yet.

### Handling errors and aborts
1. Without transactions, various error scenarios (processes crashing, network interruptions, power outages, disk full, unexpected concurrency, etc.) mean that data can become inconsistent in various ways.
1. A key feature of a transaction is that it can be aborted and safely retried if an error occurred.
2. Not all systems follow that philosophy, though. In particular, datastores with leader‐less replication (see “Leaderless Replication” on page 177) work much more on a “best effort” basis, which could be summarized as “the database will do as much as it can, and if it runs into an error, it won’t undo something it has already done”—so it’s the application’s responsibility to recover from errors.
3. Although retrying an aborted transaction is a simple and effective error handling mechanism, it isn’t perfect:
   * If the transaction actually succeeded, but the network failed while the server tried to acknowledge the successful commit to the client (so the client thinks it failed)
   * If the error is due to overload, retrying the transaction will make the problem worse, not better. To avoid such feedback cycles, you can limit the number of retries, use exponential backoff, and handle overload-related errors differently from other errors (if possible).
   * It is only worth retrying after transient errors (for example due to deadlock, isolation violation, temporary network interruptions, and failover); after a perma‐ nent error (e.g., constraint violation) a retry would be pointless.
   * If the transaction also has side effects outside of the database, those side effects may happen even if the transaction is aborted. If you want to make sure that several different systems either commit or abort together, two-phase commit can help
   * If the client process fails while retrying, any data it was trying to write to the database is lost.

# Weak Isolation Levels
1. Concurrency issues (race conditions) only come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.
2. In theory, transaction isolation should make your life easier by letting you pretend that no concurrency is happening: serializable isolation means that the database guarantees that transactions have the same effect as if they ran serially (i.e., one at a time, without any concurrency).
3. Serializable isolation has a performance cost. It’s therefore common for systems to use weaker levels of isolation, which protect against some concurrency issues, but not all. Even many popular relational database systems (which are usually considered “ACID”) use weak isolation.

## Read Committed
1. Read committed makes two guarantees:
   * When reading from the database, you will only see data that has been committed (no dirty reads).
      * This means that any writes by a transaction only become visible to others when that transaction commits (and then all of its writes become visible at once).
   * When writing to the database, you will only overwrite data that has been committed (no dirty writes)

### No Dirty Reads
1. One client reads another client’s writes before they have been committed. The read committed isolation level and stronger levels prevent dirty reads.
1. Imagine a transaction has written some data to the database, but the transaction has not yet committed or aborted. Can another transaction see that uncommitted data? If yes, that is called a dirty read.
2. Reasons to prevent dirty reads
   * If a transaction needs to update several objects, a dirty read means that another transaction may see some of the updates but not others.
   * If a transaction aborts, any writes it has made need to be rolled back

### No Dirty Writes
1. One client overwrites data that another client has written, but not yet committed. Almost all transaction implementations prevent dirty writes.
1. What happens if two transactions concurrently try to update the same object in a database? We don’t know in which order the writes will happen, but we normally assume that the later write overwrites the earlier write. However, what happens if the earlier write is part of a transaction that has not yet committed, so the later write overwrites an uncommitted value? This is called a dirty write.
2. Reasons to prevent dirty writes
   * If transactions update multiple objects, dirty writes can lead to a bad outcome. Conflicting writes from different transactions can be mixed up
   * However, read committed does not prevent the race condition between two counter increments. In this case, the second write happens after the first transaction has committed, so it’s not a dirty write.

### Implementing read committed
1. It is the default setting in Oracle 11g, PostgreSQL, SQL Server 2012, MemSQL, and many other databases
2. Most commonly, databases prevent dirty writes by using row-level locks: when a transaction wants to modify a particular object (row or document), it must first acquire a lock on that object. It must then hold that lock until the transaction is committed or aborted.
3. How do we prevent dirty reads? One option would be to use the same lock, and to require any transaction that wants to read an object to briefly acquire the lock and then release it again immediately after reading. This would ensure that a read couldn’t happen while an object has a dirty, uncommitted value (because during that time the lock would be held by the transaction that has made the write). However, one long-running write transaction can force many read-only transactions to wait until the long-running transaction has completed. 
4. More commonly, for every object that is written, the database remembers both the old committed value and the new value set by the transaction that currently holds the write lock. While the transaction is ongoing, any other transactions that read the object are simply given the old value. Only when the new value is committed do transactions switch over to reading the new value.

## Snapshot Isolation and Repeatable Read
1. Problems with Read Committed
   * **nonrepeatable read or read skew**: A client sees different parts of the database at different points in time. This issue is most commonly prevented with snapshot isolation, which allows a transaction to read from a consistent snapshot at one point in time. It is usually implemented with multi-version concurrency control (MVCC).
   * Think about an example of balance transfer from one account to another, a user read one account before incoming payment has arrived, and another account after outgoing payment has been made

1. This tempoary inconsistency can cause problem in
   * backups
   * Analytic queries and integrity checks
1. Snapshot isolation is the most common solution to this problem. The idea is that each transaction reads from a consistent snapshot of the database—that is, the transaction sees all the data that was committed in the database at the start of the transaction. Even if the data is subsequently changed by another transaction, each transaction sees only the old data from that particular point in time
2. Snapshot isolation is a boon for long-running, read-only queries such as backups and analytics.
3. it is supported by PostgreSQL, MySQL with the InnoDB storage engine, Oracle, SQL Server, and others
4. snapshot isolation is a very useful feature in databases that need to support both small, fast read-write transactions and large, long-running read-only transactions (e.g., for backups or analytics). It allows read-only transactions to see the database in a consistent state at a particular point in time, without locking and interfering with read-write transactions.

### Implementing snapshot isolation
1. snapshot isolation typically use write locks to prevent dirty writes
2. From a performance point of view, a key principle of snapshot isolation is readers never block writers, and writers never block readers. This allows a database to handle long-running read queries on a consistent snapshot at the same time as processing writes normally, without any lock contention between the two
3. The database must potentially keep several different committed versions of an object, because various in-progress transactions may need to see the state of the database at different points in time. Because it maintains several versions of an object side by side, this technique is known as multi-version concurrency control (MVCC)
   * If a database only needed to provide read committed isolation, but not snapshot iso‐ lation, it would be sufficient to keep two versions of an object: the committed version and the overwritten-but-not-yet-committed version.
   * A typical approach is that read committed uses a separate snapshot for each query, while snapshot isolation uses the same snapshot for an entire transaction.
   * 
#### Visibility rules for observing a consistent snapshot
1. When a transaction reads from the database, transaction IDs are used to decide which objects it can see and which are invisible.
2. By never updating values in place but instead creating a new version every time a value is changed, the database can provide a consistent snapshot while incurring only a small overhead

### Indexes and snapshot isolation
1. How do indexes work in a multi-version database? 
   * One option is to have the index simply point to all versions of an object and require an index query to filter out any object versions that are not visible to the current transaction. When


### Repeatable read and naming confusion
1. Snapshot isolation is a useful isolation level, especially for read-only transactions. 
2. However, many databases that implement it call it by different names. In Oracle it is called serializable, and in PostgreSQL and MySQL it is called repeatable read

## Preventing Lost Updates
1. **Lost Updates** : Two clients concurrently perform a read-modify-write cycle. One overwrites the other’s write without incorporating its changes, so data is lost. Some implemen‐ tations of snapshot isolation prevent this anomaly automatically, while others require a manual lock (SELECT FOR UPDATE).
1. The read committed and snapshot isolation levels we’ve discussed so far have been primarily about the guarantees of what a read-only transaction can see in the pres‐ ence of concurrent writes.
1. There are several other interesting kinds of conflicts that can occur between concur‐ rently writing transactions. The best known of these is the lost update problem
2. The lost update problem can occur if an application reads some value from the database, modifies it, and writes back the modified value (a **read-modify-write cycle**). If two transactions do this concurrently, one of the modifications can be lost, because the second write does not include the first modification. (We sometimes say that the later write clobbers the earlier write.)
   * Incrementing a counter or updating an account balance
   * Making a local change to a complex value, e.g., adding an element to a list within a JSON document
   * Two users editing a wiki page at the same time, where each user saves their changes by sending the entire page contents to the server, overwriting whatever is currently in the database
### Atomic write operations
1. Many databases provide atomic update operations, which remove the need to implement read-modify-write cycles in application code. They are usually the best solution if your code can be expressed in terms of those operations.
2. Atomic operations are usually implemented by taking an exclusive lock on the object when it is read so that no other transaction can read it until the update has been applied. This technique is sometimes known as cursor stability
3. Another option is to simply force all atomic operations to be executed on a single thread.
4. Not all writes can easily be expressed in terms of atomic operations—for example, updates to a wiki page involve arbitrary text editing.

### Explicit locking
1. the application explicitly lock objects that are going to be updated. Then the application can perform a read- modify-write cycle, and if any other transaction tries to concurrently read the same object, it is forced to wait until the first read-modify-write cycle has completed
2. This works, but to get it right, you need to carefully think about your application logic. It’s easy to forget to add a necessary lock somewhere in the code, and thus introduce a race condition
### Automatically detecting lost updates
1. Atomic operations and locks are ways of preventing lost updates by forcing the read- modify-write cycles to happen sequentially. An alternative is to allow them to execute in parallel and, if the transaction manager detects a lost update, abort the transaction and force it to retry its read-modify-write cycle.
2. An advantage of this approach is that databases can perform this check efficiently in conjunction with snapshot isolation
### Compare-and-set
1. The purpose of this operation is to avoid lost updates by allowing an update to happen only if the value has not changed since you last read it. If the current value does not match what you previously read, the update has no effect, and the read-modify-write cycle must be retried.
### Conflict resolution and replication
1. In replicated databases, preventing lost updates takes on another dimension: since they have copies of the data on multiple nodes, and the data can potentially be modified concurrently on different nodes, some additional steps need to be taken to prevent lost updates.
2. Locks and compare-and-set operations assume that there is a single up-to-date copy of the data. However, databases with multi-leader or leaderless replication usually allow several writes to happen concurrently and replicate them asynchronously, so they cannot guarantee that there is a single up-to-date copy of the data. Thus, techniques based on locks or compare-and-set do not apply in this context.
3. Instead, as discussed in “Detecting Concurrent Writes” on page 184, a common approach in such replicated databases is to allow concurrent writes to create several conflicting versions of a value (also known as siblings), and to use application code or special data structures to resolve and merge these versions after the fact.
4. Atomic operations can work well in a replicated context, especially if they are com‐ mutative (i.e., you can apply them in a different order on different replicas, and still get the same result)
5. On the other hand, the last write wins (LWW) conflict resolution method is prone to lost updates, Unfortunately, LWW is the default in many replicated databases.

## Write Skew and Phantoms
1. In the previous sections we saw dirty writes and lost updates, two kinds of race conditions that can occur when different transactions concurrently try to write to the same objects.

### Characterizing write skew
1. **Write skew**: A transaction reads something, makes a decision based on the value it saw, and writes the decision to the database. However, by the time the write is made, the premise of the decision is no longer true. Only serializable isolation prevents this anomaly.
1. It is neither a dirty write nor a lost update, because the two transactions are updating two different objects. 
2. You can think of write skew as a generalization of the lost update problem. Write skew can occur if two transactions read the same objects, and then update some of those objects (different transactions may update different objects). In the special case where different transactions update the same object, you get a dirty write or lost update anomaly (depending on the timing)
3. Possible Solutions
   * Atomic single-object operations don’t help, as multiple objects are involved
   * The automatic detection of lost updates that you find in some implementations of snapshot isolation unfortunately doesn’t help either: write skew is not auto‐ matically detected. 
   * Auto‐ matically preventing write skew requires true serializable isolation
   * If you can’t use a serializable isolation level, the second-best option in this case is probably to explicitly lock the rows that the transaction depends on.
### More examples of write skew
1. Meeting room booking system
2. Multiplayer game
3. Claiming a username
4. Preventing double-spending

### Phantoms causing write skew
1. **Phantom Reads**: A transaction reads objects that match some search condition. Another client makes a write that affects the results of that search. Snapshot isolation prevents straightforward phantom reads, but phantoms in the context of write skew require special treatment, such as index-range locks.
1. A SELECT query checks whether some requirement is satisfied by searching for rows that match some search condition (no existiing bookings for that room at that time)
2. Depending on the result of the first query, the application code decides how to continue
3. If the application decides to go ahead, it makes a write (INSERT, UPDATE, or DELETE) to the database and commits the transaction
   * The effect of this write changes the precondition of the decision of step 2.
   * In other words, if you were to repeat the SELECT query from step 1 after commiting the write, you would get a different result, because the write changed the set of rows matching the search condition
1. The steps may occur in a different order.
1. This effect, where a write in one transaction changes the result of a search query in another transaction, is called a phantom.
2. Snapshot isolation avoids phantoms in read-only queries, but in read-write transactions phantoms can lead to particularly tricky cases of write skew.
### Materializing conflicts
1. If the problem of phantoms is that there is no object to which we can attach the locks, perhaps we can artificially introduce a lock object into the database?
2. This approach is called materializing conflicts, because it takes a phantom and turns it into a lock conflict on a concrete set of rows that exist in the database
3. Unfortu‐ nately, it can be hard and error-prone to figure out how to materialize conflicts, and it’s ugly to let a concurrency control mechanism leak into the application data model.
4. A serializable isolation level is much preferable in most cases.

# Serializability
1. Serializable isolation is usually regarded as the strongest isolation level. It guarantees that even though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially, without any concurrency.
2. Most databases that provide serializability today use one of three techniques
   * Literally executing transactions in a serial order
   * Two-Phase Locking which for sev‐ eral decades was the only viable option
   * Optimistic concurrency control techniques such as serializable snapshot isolation
## Actual Serial Execution
1. The simplest way of avoiding concurrency problems is to remove the concurrency entirely: to execute only one transaction at a time, in serial order, on a single thread. This is possible because:
   * RAM became cheap enough that for many use cases is now feasible to keep the entire active dataset in memory
   * Database designers realized that OLTP transactions are usually short and only make a small number of reads and writes. By contrast, long-running analytic queries are typically read- only, so they can be run on a consistent snapshot (using snapshot isolation) outside of the serial execution loop.
1. VoltDB/H-Store, Redis, Datomic
2. However, its throughput is limited to that of a single CPU core. In order to make the most of that single thread, transactions need to be structured differently from their traditional form
### Encapsulating transactions in stored procedures
1. almost all OLTP applications keep transac‐ tions short by avoiding interactively waiting for a user within a transaction.
2. this means that a transaction is committed within the same HTTP request—a transaction does not span multiple requests. A new HTTP request starts a new trans‐ action.
3. transactions have continued to be executed in an interactive client/server style, one statement at a time.
4. For this reason, systems with single-threaded serial transaction processing don’t allow interactive multi-statement transactions. Instead, the application must submit the entire transaction code to the database ahead of time, as a stored procedure. 
5. Provided that all data required by a transaction is in memory, the stored procedure can execute very fast, without waiting for any network or disk I/O
#### Pros and cons of stored procedures
1. Each database vendor has its own language for stored procedures
2. Code running in a database is difficult to manage
3. A database is often much more performance-sensitive than an application server, because a single database instance is often shared by many application servers. A badly written stored procedure (e.g., using a lot of memory or CPU time) in a database can cause much more trouble.
4. With stored procedures and in-memory data, executing all transactions on a single thread becomes feasible. As they don’t need to wait for I/O and they avoid the over‐ head of other concurrency control mechanisms, they can achieve quite good throughput on a single thread.

#### Partitioning
1. Read-only transactions may execute elsewhere, using snapshot isola‐ tion, but for applications with high write throughput, the single-threaded transaction processor can become a serious bottleneck.
2. In order to scale to multiple CPU cores, and multiple nodes, you can potentially par‐ tition your data (see Chapter 6), which is supported in VoltDB.
   * but data with multiple secondary indexes is likely to require a lot of cross- partition coordination

#### Summary of Serial Execution Constraints
1. Every transaction must be small and fast, because it takes only one slow transac‐ tion to stall all transaction processing.
1. It is limited to use cases where the active dataset can fit in memory. Rarely accessed data could potentially be moved to disk, but if it needed to be accessed in a single-threaded transaction, the system would get very slow. We can do anti-caching here by abort the transaction, asynchronously fetch data and re-try.
1. Write throughput must be low enough to be handled on a single CPU core, or else transactions need to be partitioned without requiring cross-partition coordi‐ nation.
1. Cross-partition transactions are possible, but there is a hard limit to the extent to which they can be used.

## Two-Phase Locking (2PL)
1. Sev‐ eral transactions are allowed to concurrently read the same object as long as nobody is writing to it. But as soon as anyone wants to write (modify or delete) an object, exclusive access is required
2. In 2PL, writers don’t just block other writers; they also block readers and vice versa. Snapshot isolation has the mantra *readers never block writers, and writers never block readers*. , which captures this key difference between snapshot isolation and two-phase locking.
3. because 2PL provides serializability, it protects against all the race conditions dis‐ cussed earlier, including lost updates and write skew.

### Implementation of two-phase locking
1. The blocking of readers and writers is implemented by a having a lock on each object in the database. The lock can either be in shared mode or in exclusive mode.
2. two phases: the first phase (while the transaction is executing) is when the locks are acquired, and the second phase (at the end of the transaction) is when all the locks are released
3. Since so many locks are in use, it can happen quite easily that transaction A is stuck waiting for transaction B to release its lock, and vice versa. This situation is called deadlock.
### Performance of two-phase locking
1. Big downside: transaction throughput and response times of queries are significantly worse under two-phase locking than under weak isolation.
2. This is partly due to the overhead of acquiring and releasing all those locks, but more importantly due to reduced concurrency. By
3. databases running 2PL can have quite unstable latencies, and they can be very slow at high percentiles if there is contention in the workload. It
### Predicate locks
1. (mainly to avoid phantoms) It works similarly to the shared/exclusive lock described earlier, but rather than belonging to a particular object (e.g., one row in a table), it belongs to all objects that match some search condition
2. The key idea here is that a predicate lock applies even to objects that do not yet exist in the database, but which might be added in the future (phantoms). If two-phase locking includes predicate locks, the database prevents all forms of write skew and other race conditions, and so its isolation becomes serializable.

### index-range locks
1. Unfortunately, predicate locks do not perform well: if there are many locks by active transactions, checking for matching locks becomes time-consuming. For that reason, most databases with 2PL actually implement index-range locking (also known as next- key locking), which is a simplified approximation of predicate locking
2. 


## Serializable Snapshot Isolation (SSI)
1. On the one hand, we have implementations of serializability that don’t perform well (two- phase locking) or don’t scale well (serial execution). 
2. On the other hand, we have weak isolation levels that have good performance, but are prone to various race conditions (lost updates, write skew, phantoms, etc.)
3. SSI is promising: It provides full serializability, but has only a small performance penalty compared to snapshot isolation
4. Today SSI is used both in single-node databases (the serializable isolation level in PostgreSQL since version 9.1 [41]) and distributed databases (FoundationDB uses a similar algorithm).
### Pessimistic versus optimistic concurrency control
1. Two-phase locking is a so-called pessimistic concurrency control mechanism: it is based on the principle that if anything might possibly go wrong (as indicated by a lock held by another transaction), it’s better to wait until the situation is safe again before doing anything. It is like mutual exclusion, which is used to protect data struc‐ tures in multi-threaded programming.
2. Serial execution is, in a sense, pessimistic to the extreme: it is essentially equivalent to each transaction having an exclusive lock on the entire database (or one partition of the database) for the duration of the transaction.
3. By contrast, serializable snapshot isolation is an optimistic concurrency control tech‐ nique. Optimistic in this context means that instead of blocking if something poten‐ tially dangerous happens, transactions continue anyway, in the hope that everything will turn out all right. 
4. It performs badly if there is high con‐ tention (many transactions trying to access the same objects), as this leads to a high proportion of transactions needing to abort.
5. However, if there is enough spare capacity, and if contention between transactions is not too high, optimistic concurrency control techniques tend to perform better than pessimistic ones.
6. SSI is based on snapshot isolation—that is, all reads within a transaction are made from a consistent snapshot of the database. 
7. On top of snapshot isolation, SSI adds an algorithm for detecting serialization conflicts among writes and determining which transactions to abort

### Decisions based on an outdated premise
1. In snapshot isolation: the transaction is taking an action based on a premise (a fact that was true at the beginning of the transaction, e.g., “There are currently two doctors on call”). Later, when the transaction wants to commit, the original data may have changed—the premise may no longer be true.
2. How does the database know if a query result might have changed? There
   * Detecting reads of a stale MVCC object version (uncommitted write occurred before the read)
   * Detecting writes that affect prior reads (the write occurs after the read)
### Performance of serializable snapshot isolation
1. Compared to two-phase locking, the big advantage of serializable snapshot isolation is that one transaction doesn’t need to block waiting for locks held by another trans‐ action.
2. Like under snapshot isolation, writers don’t block readers, and vice versa. This design principle makes query latency much more predictable and less variable.
3. Compared to serial execution, serializable snapshot isolation is not limited to the throughput of a single CPU core
4. The rate of aborts significantly affects the overall performance of SSI. so SSI requires that read-write transactions be fairly short (long- running read-only transactions may be okay). However,SSI is probably less sensitive to slow transactions than two-phase locking or serial execution.
