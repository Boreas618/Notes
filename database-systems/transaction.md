# Transaction

Transactions guarantee the **ACID** properties to avoid the problems:

* Atomicity: A transaction ends in two ways: it either commits or aborts. Atomicity means that either all actions in the Xact happen, or none happen.
* Consistency: If the DB starts out consistent, it ends up consistent at the end of the Xact.
* Isolation: Execution of each Xact is isolated from that of others. In reality, the DBMS will interleave actions of many Xacts and not execute each in order of one after the other. The DBMS will ensure that each Xact executes as if it ran by itself.
* Durabilty: If a Xact commits, its eﬀects persist. The eﬀects of a committed Xact must survive failures.

## Concurrency Control

Four transaction operations:

* Begin
* Read
* Write
* Commit
* Abort

SQL keywords `BEGIN`, `COMMIT`, `ROLLBACK`.

A **serial schedule**: run all the operations of one transaction to completion before beginning the operations of next transaction

If we ﬁnd a schedule whose results are equivalent to a serial schedule, we call the schedule **serializable**.

We can interleave the execution of operations based on concurrency control algorithms such as locking or time stamping. There are several methods of achieving concurrency control:

* Locking (Most common)
* Time stamping
* Optimistic Concurrency Control

### Lock

Guarantees exclusive use of a data item to a current transaction

* T1 acquires a lock prior to data access; the lock is released when the transaction is complete
* T2 does not have access to data item currently being used by T1
* T2 has to wait until T1 releases the lock

A lock manager is responsible to prevent another transaction from reading inconsistent data.

* Database-level lock

  The entire database is locked. It is good for batch processing but unnsuitable for multi-user DBMSs

  T1 and T2 can not access the same database concurrently even if they use different tables

* Table-level lock

  Not suitable for highly Muti-user DBMSs

* Page-level lock

  An entire disk page is locked

  Not commonly used now

* Row-level lock

  Very high overhead (Every row has a lock now)

  Highly popular

* Field-level lock

  Most flexible lock but requires an extremely high level of overhead

Binary locks has only two states: **locked** or **unlocked**. It eliminates the "Lost Update" problem. But it sis considered too restrictive to yield optimal concurrency, as it locks even for two READs (When no update is being down)

The alternative is to allow both **Shared** and **Exclusive** locks. (Often called Read and Write locks)

**Exclusive lock**:

Access is reserved for the transaction that locked the object 

Must be used when transaction intends to WRITE 

Granted if and only if no other locks are held on the data item 

**Shared lock**:

Other transactions are also granted Read access 

Issued when a transaction wants to READ data, and no Exclusive lock is held on that data item 

Multiple transactions can each have a shared lock on the same data item if they are all just reading it 

**Deadlock**

Condition that occurs when two transactions wait for each other to unlock data 

T1 locks data item X, then wants Y 

T2 locks data item Y then wants X 

Each waits to get a data item which the other transaction is already holding 

Could wait forever if not dealt with

Only happens with exclusive locks

Deadlocks are dealt with by:

* Prevention
* Detection

### Optimistic 

Based on the assumption that the majority of database operations do not conflict 

Transaction is executed without restrictions or checking 

Then when it is ready to commit, the DBMS checks whether any of the data it read has been altered (if so, roll back)

To find a serializable schedule without running through the entire schedule, we can start by looking for **conflicting operations**. The conficting operations:

* The operations are from diﬀerent transactions
* Both operations operate on the same resource
* At least one operation is a write

When two schedules order their conﬂicting operations in the same way the schedules are said to be **conﬂict equivalent**, which is a stronger condition than being equivalent.

We call a schedule that is conﬂict equivalent to some serial schedule **conﬂict serializable**.

If a schedule S is **conﬂict serializable** then it implies that is serializable. Not all serializable schedules are conﬂict serializable.

### Conflict Dependency Graph

* One node per Xact

* Edge from $$T_i$$ to $$T_j$$ if:
  * an operation $$O_i$$ of $$T_i$$ conﬂicts with an operation $$O_j$$ of $$T_j$$
  * $$O_i$$ appears earlier in the schedule than $$O_j$$-1

## Logging Transactions

Allow us to restore the database to a previous consistent state.

If a transaction cannot be completed, it must be aborted and any changes rolled back.

To enable this, DBMS tracks all updates to data.

This transaction log contains: 

1. A record for the beginning of the transaction
2. For each SQL statement 
   * operation being performed (update, delete, insert) 
   * objects affected by the transaction 
   * “before” and "after" values for updated fields 
   * pointers to previous and next transaction log entries 

3. The ending (COMMIT) of the transaction
