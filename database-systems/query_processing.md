# Query Processing Overview

Main weapons are:

1. clever implementations techniques for operators
2. exploiting ‘quivalencies’ of relational operators
3. using cost models to choose among alternatives

### Workflow

<img src="https://p.ipic.vip/d5o8xl.jpg" alt="Screenshot 2023-04-17 at 5.26.23 PM.png" style="zoom:50%;" />

# Selections

**Schema for Examples**

Sailors(sid: integer, sname: string, rating: integer, age: real)

Reserves(sid: integer, bid: integer, day: dates, rname: string)

**Sailors(S)**

Each tuple is 50 tuples long, 80 tuples per page, 500 pages

**Reserves(R)**

Each tuple is 40 bytes long, 100 tuples per page, 1000 pages

## Simple Selections

The best way to preform a selection depends on:

1. avaliable indexes/access paths
2. expected **size of the result** (number of tuples and/or number of pages)

**Estimate result size (reduction factor)**

Size of result approximated as:

$$size\_of\_relation \times \prod(reduction\_factors)$$

**Reduction factor** is usually called **selectivity**. It estimates what portion of the relation will qualify for the given predicate, i.e. satisfy the given condition.

This is estimated by the optimizer.

### Alternatives for Simple Selections

1. With no index, unsorted:

   Must scan the whole relation, i.e. perform Heap Scan

   **Cost =** $$[R]$$

2. With no index, but file is sorted:

   **Cost =** binary search cost + $$[R_{condition}]$$

   **Cost =** $$\log_2([R]) + (RF\times [R])$$

3. With an index on selection attribute:

   Use index to find qualifying data entries, then retrieve corresponding data records

   Cost depends on the number of qualifying tuples
   
   Clustering is important when calculating the total cost
   
   1. Clustered index **Cost =** $$([I]+[R])\times RF$$
   2. Unclustered index **Cost =** $$([I]+|R|))\times RF$$

![Untitled](https://p.ipic.vip/k3hbnh.jpg)

## General Selection Conditions

Typically queires have multiple predicates (conditions)

Example: `day<8/9/94 AND rname='Paul' AND bid=5 AND sid=3`

A B+ tree index matches (a combination of) predicates that involve only attributes in a  **prefix of the search key (upon which we build the index).**

Index on `<a, b, c>` matches predicates on `(a, b, c)` `(a, b)` `(a)`

This implies that only reduction factors of the **matching predicates(or primary conjuncts)** that are part of the prefix will be used to determine the index cost.

## Selection Approach

1. Find the cheapest access path 

   An index or file scan with the **least estimated page I/O** 

2. Retrieve tuples using it 

   Predicates that match this index reduce the number of tuples retrieved (and impact the cost) 

3. Apply the predicates that don't match the index (if any) later on 

   These predicates are used to discard some retrieved tuples, but do not affect number of tuples/pages fetched (nor the total cost) 

   In this case selection over other predicates is said to be done “on-the-fly”

**Example: `day < 8/9/24 AND bid = 5 AND sid = 3`**

A **B+ tree** index **on day** can be used: $$RF = RF(day)$$ . Then `bid=5` and `sid=3` must be checked for each retrieved tuple on the fly

Similarly,  a **hash index on <bid, sid>** could be used $$\prod RF=RF(bid)\times RF(sid)$$. Then, `day<8/9/94` must be checked on the fly.

# Projections

Issue with projection is removing duplicates

```sql
SELECT DISTINCT R.sid, R.bid
FROM Reserves R;
```

Projection can be done based on **hashing** or **sorting.** By sorting, the duplicates are adjacent to each other. Bu hashing, the duplicates fall on the same bucket.

## Sorting

Basic approach is to use **sorting**

1. Scan R, extract only the needed attributes
2. Sort the result set(typically using external merge sort)
3. Remove adjacent duplicates

Usually, we bring data from disk to memory to sort it. But what if the data is too large to fit in memory? 

### A Simple Two-way Merge Sort

When sorting a file, several sorted subfiles are typically generated in intermediate steps. Each sorted file is called a **run**.

In the first pass, the pages in the file are read in one at a time. After a page is read in, **the records on it are sorted and the sorted page is written out**.

In subsequent passes, pairs of runs from the output of the preovious pass are read in and merged to produce runs that are twice as long.

If the number of pages in the input file is $$2^k$$, for some $$k$$, then:

Pass $$0$$ produces $$2^k$$ sorted runs of one page each.

Pass $$1$$ produces $$2^{k-1}$$ sorted runs of two pages each.

Pass $$2$$ produces $$2^{k-2}$$ sorted runs of four pages each.

…

Pass $$k$$ produces $$1$$ sorted run of $$2^k$$ pages.

In each pass, we read every page in the file, process it, and write it out. We have 2 I/Os per page, per pass. The overall cost is $$2N(\lceil \log_{2}^{N}\rceil+1)$$ I/Os.

Only three buffer pages in the memory is needed (Two inputs and oen output). In order to merge 2 **sorted runs,** we only need two buffer pages. We compare the records respectively. Once one page of one sorted run is run out, we can load a new page. Therefore, to merge 2 sorted runs, only 2 buffer pages are needed. Beyond 2 buffer pages, 1 output page is needed.

### External Merge Sort

Two optimizations as opposed to Two-way merge sort:

- In the initial “conquer pass”, we load B pages and sort them at a time.
- Merge more than 2 pages at a time. Because we need a output buffer page, we can merge $$B-1$$ pages at a time.

The number of I/Os needed is $$2N\times(1+\lceil \log _{B-1}^{\frac{N}{B}}\rceil)$$.

Sort runs: Make each B pages sorted (called runs)

Merge runs: Make multiple passes to merge runs

- Pass 2: Produce runs of length $$B(B-1)$$ pages
- Pass 3: Produce runs of length $$B(B-1)^2$$ pages
- …
- Pass P: Produce runs of length $$B(B-1)^P$$ pages

![Screenshot 2023-04-18 at 7.56.32 PM.png](https://p.ipic.vip/98kp15.jpg)

Projection with **external sort:**

1. Scan R, extract only the needed attributes
2. Sort the result set using EXTERNAL SORT
3. Remove adjacent duplicates

![Screenshot 2023-04-18 at 8.08.53 PM.png](https://p.ipic.vip/e31if3.png)

**WriteProjectedPages** = $$[R] \times PF$$

**PF: Projection Factor** says how much are we projecting, ratio with respect to all attributes ( e.g. keeping $$\frac{1}{4}$$  of attributes, or $$10\%$$ of all attributes )

**Sorting Cost = $$2\times$$ NumPasses$$\times$$ReadProjectedPages**

## Hashing

1. Scan $$R$$, extract only the needed attributes

2. Hash data into buckets

   Apply hash function $$h_1$$ to choose one of $$B-1$$ output buffers

3. Remove adjacent duplicates from a bucket

   2 tuples from different partitions guaranteed to be distinct

<img src="https://p.ipic.vip/quha8m.jpg" alt="Screenshot 2023-04-18 at 8.17.46 PM.png" style="zoom:50%;" />

### Projection based on External Hashing

**Partition** data into $$B-1$$ partitions with $$h_1$$ hash function

Load each partition, hash it with another hash function($$h_2$$) and eliminate duplicates

<img src="https://p.ipic.vip/nip3w9.png" alt="Screenshot 2023-04-18 at 8.21.46 PM.png" style="zoom:50%;" />

1. Partitioning phase

   - Read $$R$$ using one input buffer

   - For each tuple:

     Discard unwanted fields

     Apply hash function $$h_1$$ to choose one of $$B-1$$ output buffers

   - Result is $$B-1$$ partitions (of tuples with no unwanted fields)

     2 tuples from different partitions guaranteed to be distinct

2. Duplicate elimination phase

   - For each partition

     Read it and build an in-memory hash table -

     Using hash function $$h_2$$ on all fields 

     While discarding duplicates

   - If partition does not fit in memory

     Apply hash-based projection algorithm recursively to this partition

# Joins

To find matches between two lists of records, it is important to sort them first. Otherwise, the process can be fairly inefficient.

```sql
Table A       Table B
ID            ID
1             7
2             3
7             8
3             2
9             9
```

Join techniques we will cover:

- Nested-loops join
- Sorted-merge join
- Hash join

Cross product is very expensive. $$R \times S$$ followed by a selection is inefficient.

Consider two inputs of tables:

The left input is called the **outer input** and right input **is called the **inner inpu**t.

![Screenshot 2023-04-21 at 6.54.56 AM.png](https://p.ipic.vip/k1wgav.png)

Have nothing to do with inner/ outer joins!

**Join is associative and communicative**

## Simple Nested Loops Join

For each tuple in the outer relation R, we scan the entire inner relation S.

**Pseudo Code**:

```pseudocode
foreach tuple in R do
	foreach tuple s in S do
		if ri == sj then add<r, s> to result
```

**Cost** = $$[Outer]+|Outer|\times [Inner]$$

## Page-Oriented Nested Loops Join

For each **page** of R, get **each page** of S. Write out matching pairs of tuples $$<r,s>$$, where $$r$$ is in R page and S is in S-page.

Pseudo code:

```pseudocode
for each page bR in R do
	for each page bS in S do
		for each tuple r in bR do
			for each tuple in bS do
				if ri == sj then add<r, s> to result
```

**Cost** = $$[Outer]+[Outer]\times [Inner]$$

## Block Nested Loops Join

Page-oriented NL doesn’t exploit extra memory buffers

Use one page as an input buffer for scanning the inner S, one page as the output buffer, and use the remaining pages to hold ‘block’ of outer R.

For each matching tuple $$r$$ in R-block, $$s$$ in S-page, add $$<r,s>$$ to result. Then read next R-block,scan S, etc.

![Screenshot 2023-04-21 at 7.20.09 AM.png](https://p.ipic.vip/7wwbch.jpg)

**Cost** = $$[Outer]+NBlocks(Outer)\times [Inner]$$

$$\tiny{NBlocks(Outer)=\lceil\frac{[Outer]}{B-2}\rceil}$$

## Index Nested Loop Join

```pseudocode
foreach record ri in R:
	foreach record sj in S where condi(ri,sj) == true:
		yield <ri, sj>
```

**Cost** =  $$[R]+|R|\times(cost\,to\,look\,up\,matching\,records\,in\,S)$$

## Sort-Merge Join

It's cumbersom to handle the identical keys in $$R$$ and $$S$$.

```pseudocode
function sort_merge_join():
	sort(R)
	sort(S)
	p := first tuple of R
	q := first tuple of S
	result := []
	while !(either p or q reaches the end of its relation):
		if R[p].join_attribute < S[q].join_attribute:
  		increment(p)
  	else if R[p].join_attribute > S[q].join_attribute:
  		increment(q)
  	else (R[p].join_attribute == S[q].join_attribute):
    	mark(q)
    	while R[p].join_attribute == S[q].join_attribute:
    		result.add(join(R[p], S[q]))
      	increment(q)
    	restore(q)
  		increment(p)
 	return result

```

**Average cost =** $$Sort(R)+Sort(S)+[R]+[S]$$

**Worst case cost = ** $$Sort(R)+Sort(S)+|R|\times[S]$$

$$Sort(R)=2\times \#Passes\times [R]$$

## Hash join

**Requires** equality predicate.

### Naive Hash Join

construct a hash table with a size of $$B-2$$ pages for $$R$$. Look up records of $$S$$ in the hash table. The cost is $$[R]+[S]$$. The problem is that $$R$$ needs to be fit in the memory.

### Grace Hash Join

Two phases:

* Partition (divide)
* Build and Probe (conquer)

**Partition** both relations using hash function $$h_1$$. $$R$$ tuples in partition $$I$$ will only match $$S$$ tuples in partition $$I$$. If the partition doesn't fit in the memory, we recursively partiton it. Write the partition into the disk.

Read in a partition of tuples of $$R$$ and **build** a hash table using hash function $$h_2$$. Scan matching partition of S, **probe** hash table for matches.

In **Partition** phase, we read+write both relations.

In **Build and Probe** phase, we read both relations.

**Cost =** $$3\times [R]+3\times [S]$$

### Code Example for Grace Hash Join

```c++
#include <functional>
#include <unordered_map>
#include <vector>

using Record = std::pair<uint32_t, std::string>;
using Table = std::vector<Record>;

// Hash function for partitioning input tables
struct Hash1 {
    size_t operator()(const Record& r) const {
        // TODO: Implement your own hash function here
        // Use the join key value of the record to compute its hash value
        // Return a size_t value that represents the hash value
    }
};

// Hash function for constructing the hash table
struct Hash2 {
    size_t operator()(const Record& r) const {
        // TODO: Implement your own hash function here
        // Use the join key value of the record to compute its hash value
        // Return a size_t value that represents the hash value
    }
};

// Recursive partitioning function
void partition(const Table& table, std::vector<Table>& partitions, size_t depth) {
    // TODO: Implement the recursive partitioning function here
    // Use the Hash1 hash function to determine which partition each record belongs to
    // Continue partitioning until the desired depth is reached
}

// Grace hash join function
Table grace_hash_join(const Table& table1, const Table& table2) {
    // Partition the two input tables using the Hash1 hash function
    std::vector<Table> partitions1(depth);
    partition(table1, partitions1, depth);

    std::vector<Table> partitions2(depth);
    partition(table2, partitions2, depth);

    // Construct the hash table using the Hash2 hash function
    std::unordered_map<uint32_t, std::vector<std::string>, Hash2> hash_table;
    for (const auto& record : table2) {
        auto& bucket = hash_table[record.first];
        bucket.push_back(record.second);
    }

    // Perform the join operation using the partitioned tables and hash table
    Table result;
    for (const auto& partition : partitions1) {
        for (const auto& record1 : partition) {
            auto it = hash_table.find(record1.first);
            if (it != hash_table.end()) {
                for (const auto& record2 : it->second) {
                    result.emplace_back(record1.first, record1.second + record2);
                }
            }
        }
    }

    return result;
}

```

## General Join Conditions

Equalities over serveral attributes

For inequality conditions, hash join and sort merge join is not applicable. Block NL quite likely to be the best join method here.