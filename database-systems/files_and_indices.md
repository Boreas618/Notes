# Files and Indices

## Files

File is a collection of pages, each containing a collection of records.

![](https://p.ipic.vip/0gbmjt.jpg)

Many alternative file organizations exists, each is good for some situations, and not so good in others.

*   Heap Files have no particular order among the records.

    They are suitable when the typical access is a file scan retrieving all records.

    They are also feasible for inserting records.
*   Sorted Files: pages and records within pages are ordered by some condition

    Best for retrieval (of a range of records) in some order

    They are also feasible for finding and manipulating some records.
*   Index File Organizations

    Special data structure that has the fastest retrieval in some order

### Heap Files

Simplest file structure, contains records in no particular order. As file grows and shrinks, disk pages are allocated and de-allocated. Fastest for **inserts** compared to other alternatives

![](https://p.ipic.vip/0hkt58.jpg)

### Sorted Files

Similar structure like heap files (pages and records), but pages and records are ordered.

Fast for range queries, but hard for maintenance( each insert potentially reshuffles records )

![](https://p.ipic.vip/b0n49e.png)

***

DBMS model the cost of all operations

The cost is typically expressed in **the number of page accesses** (or disk **I/O operations** - to bring data from disk to memory)

1 page access (on disk) == 1 I/O (used interchangeably)

## Index

Sometimes, we want to retrieve records by specifying the values in one or more fields.

An index is a data structure built on top of data pages used for efficient search. The index is built over specific fields called **search key fields.**

**Any** subset of the fields of a relation can be the search key for an index on the relation.

_Search key_ is not the same as _key_.

An index contains a collection of **data entries**, and supports efficient retrieval pf **data records** matching a given search condition.

![Screenshot 2023-04-10 at 6.26.02 PM.png](https://p.ipic.vip/azub82.jpg)

## Index Classification

Classification based on various factors:

* Clustered vs. Unclustered
* Primary vs. Secondary
* Single Key vs. Composite
* Indexing technique: Tree-based, hash-based, other

### Clustered vs. Unclustered

If **order of data records** is the same as the **order of index data entrie**s, then the index is called clustered index. Otherwise is unclustered.

<figure><img src="https://p.ipic.vip/1f8d1f.jpg" alt="" width="375"><figcaption></figcaption></figure>

A data file can have a clustered index on at most **one search key combination**. (i.e. we cannot have multiple clustered indexes over a single table)

Cost of retrieving data records through index varies greatly based on whether index is clustered. (cheaper for clustered)

Clustering indexes are more expensive to maintain (require file reorganization), but are really efficient for **range search.**

Approximated cost of retrieving records found in range scan:

1. Clustered: cost $$\approx$$ $$\#$$ of pages in data file with matching records
2. Unclustered: cost$$\approx$$ $$\#$$ of matching index data entries (data records)

### Primary vs. Secondary Index

* **Primary** index includes the table’s **primary key**
* **Secondary** is any other index

Properties:

* Primary index never contains duplicates
* Secondary index may contain duplicates

### Composite Search Keys

An index can be built over a combination of search keys.

Data entries in index sorted by search keys.

Examples:

1. Index on $$<age, sal>$$
2. Index on $$<sal,age>$$

### Hash-based Index

Hash-based index

*   Represents index as a collection of buckets. Hash function maps the search key to the corresponding bucket.

    $$h(r.search\_key) =$$ bucket in which record $$r$$ belongs
* Good for **equality-based** selections
