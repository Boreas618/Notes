# Query Optimization

**Query Plan** is a tree, with relational algebra operators as nodes and access paths as leaves. Each operator labeled with a choice of algorithm.

```sql
SELECT sname 
FROM Sailors NATURAL JOIN Reserves
WHERE bid = 100 and rating > 5
```

<figure><img src="https://p.ipic.vip/344xej.png" alt="" width="375"><figcaption></figcaption></figure>

## Query Optimization

Query optimization steps:

1. Query first broken into "blocks"
2. Each block converted to relational algebra
3.  Then, for each block, several alternative query plans are considered

    Cross product is costly. We can convert cross product into natural join. We'd better reduce the amount of tuples needed in the join.

    We can also "push down" projection: $$\pi_{s.sname}(Sailors\Join_{s.sid=R.rid} Reserves)=\pi_{s.sname}(\pi_{sname,sid}(Sailors)\Join_{S.sid=R.rid}\pi_{sid}(Reserves))$$
4. Plan with the lowest estimated cost is selected

## Cost Estimation

For each plan considered, must estimate cost:

* Must estimate size of result for each operation in tree
*   Must estimate cost of each operation in plan tree

    Depends on input cardinalities

To decide on the cost, the optimizer needs information about the relations and indices involved. This information is stored in the system **catalogs**.

Catalogs typically contain at least:

* $$[R]$$ and $$|R|$$
* Distinct key values for each index
* Low/high values for each tree index
* Index height for each tree index
* Index pages for each index

Statistics in catalogs are updated periodically.

## Result size estimation

**Reduction factor** associated with each predicate reflects the impact of the predicate in reducing the result size. RF is also called selectivity.

Single table selection: $$Size=[R]\prod_{i=1..n}RF_i$$

Joins over $$k$$ tables: $$Size=\prod_{j=1..k}[R_j]\prod_{i=1..n}RF_i$$

**Col = value** $$RF=\frac{1}{\#Keys(Col)}$$

**Col > value** $$RF=\frac{High(Col)-value}{High(Col)-Low(Col)}$$

**Col < value** $$RF=\frac{value-Low(Col)}{High(Col)-Low(Col)}$$

**Col\_A=Col\_B (for joins)** $$RF=\frac{1}{Max(\#Keys(Col_A),\#Keys(Col_B))}$$

**No information** $$RF=\frac{1}{10}$$

## Enumeration of Alternative Plans

Two main cases:

* Single-relation plans
* Multiple-relation plans (joins)

### A Single Relation

For each available access path (file scan/ index) is considered, and the one with the lowest estimated cost is chosen.

* Heap scan is always one alternative
* Each index can be another alternative (if matching selection predicates)

1. Sequential scan of data file: **Cost=**$$[R]$$
2.  Index selection over a **primary key** (just a single tuple):

    **Cost(B+ Tree)** = $$Height(I)+1$$

    **Cost(Hash Index) =**$$ProbeCost(I) + 1$$ ProbeCost(I) \~ 1.2
3.  Clustered index matching one or more predicates:

    **Cost(B+ Tree)** = $$([I]+[R]) \times\prod_{i=1..n}RF_{i}$$

    **Cost(Hash Index)** = $$[R] \times\prod_{i=1..n}RF_i \times2.2$$
4.  Non-clustered index matching one or more predicates

    **Cost(B+ Tree)** = $$([I]+|R|) \times\prod_{i=1..n}RF_{i}$$

    **Cost(Hash Index)** = $$|R| \times\prod_{i=1..n}RF_i\times2.2$$

### Multiple Relations

Steps:

*   Select order of relations

    Maximum possible orderings = $$N!$$
* For each join, select **join algorithm**
* For each input relation, select access method

As number of joins increases, number of alternative plans grows rapidly. We need to restrict search space.

Fundamental decision in System R: only **left-deep join trees** are considered

<figure><img src="https://p.ipic.vip/1j8vmm.png" alt=""><figcaption></figcaption></figure>

The outcome of a previous join is immediately fed into the next join without writing to the disk. Hence, the cost of reading outer pages for the next join is discarded.

**Note that only the first reading cost is discarded.** For hash join, however, it writes the partition to the disk and read the disk again. This part of reading cost cannot be pruned. The cost is $$2\times[Outer]+3\times[Inner]$$

Cross products is instantly pruned.
