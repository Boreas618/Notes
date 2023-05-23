# Definitions

**Distributed Database**: ﻿﻿a single logical database physically spread across multiple computers in multiple locations that are connected by a data communications link

* appears to users as though it is one database

**Decentralized Database**: a collection of independent databases which are not networked together as one logical database

* appears to users as though many databases

**Advantages of ditributed DBMS**

* Good fit for geographically distributed organizations/ users

* Data located near site with greatest demand

* Faster data access (to local data)

* Faster data processing

  Workload is splited amongst physical servers

* Allows modular growth

  Thanks to horizontal scalability

* Increased reliability and availability

  Less danger of a single-point of failure(SPOF), **if** data is replicated

* Supports database recovery

  When data is replicated across mutiple sites

**Objectives of distributed DBMS**

**Location transparency**: a user does not need to know where particular data are stored

**Local autonomy**: a node can continue to function for local users if connectivity to the network is lost

**Funtions of a ditrbuted DBMS**

* Locate data with a distributed catalog (meta data)
* Determine location from which to retrieve data and process query components
* DBMS translation between nodes with different local DBMSs (using middleware)
* Data consistency (via multiphase commit protocols) 
* Global primary key control 
* Scalability 
* Security, concurrency, query optimization, failure recovery

# Distribution options

When distributing data around the world, the data can be partitioned ot replicated.

**Data replication**: the process of duplicating data to different nodes. 

**Data partitioning**: the process of partitioning data into subsets that are shipped to different nodes. 

Many real-life systems use a combination of two (e.g. distribute data and keep some replicas around usually 3)

**Advantages of republication**

* High reliability due to redundant copies of data 

* Fast access to data at the location where it is most accessed 

* May avoid complicated distributed integrity routines 

  Replicated data is refreshed at scheduled intervals 

* Decoupled nodes don't affect data availability 

  Transactions proceed even if some nodes are down

* Reduced network traffic at prime time

  If updates can be delayed

* This is currently popular as a way of achieving high availability for global systems

**Disadvantages**

* Need more storage space

* Data Integrity

* Takes time for update operations
* Network communication

----

**Horizontal partitioning**: Table rows distributed across nodes (sides) 

**Vertical partitioning**: Table columns distributed across nodes (sides)

**Advantages of partitioning**

* Data stored close to where it is used (NBA in US, ARL in AUS)
* Local access optimization
* Only relevant data is stored locally
* Unions across partitions

**Disadvantages of partitioning** 

* Accessing data across partitions

* No data replication backup vulnerability (SPOF)

----

**Trade-offs**

Availability vs Consistency 

The CAP theorem says we need to decide whether to make data always available **OR** always consistent 

Synchronous vs Asynchronous updates 

Are changes immediately visible everywhere (great BUT expensive) or later propagated (less expensive faster, but seeing stale data)?

# Sync and Async Updates

**Synchronous updates**: Data is continuously kept up to date

Ensures data integrity and minimizes the complexity of knowing where the most recent copy of data is located.

Slow response time and high network usage

**Asynchronous updates**: Some delay in propagating data updates to remote databases

Acceptable response time

More complex to plan and design

# The CAP Theorem

You can only have two of three of Consistency, Partition Tolerance and Availability.

**Consistency**: everyone always sees the same data

**Availability**: System stays up when nodes fail

**Partition Tolerance**: System stays up when network between nodes fail

