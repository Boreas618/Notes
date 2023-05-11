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

A **serial schedule**: run all the operations of one transaction to completion before beginning the operations of next transaction

If we ﬁnd a schedule whose results are equivalent to a serial schedule, we call the schedule **serializable**.

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
