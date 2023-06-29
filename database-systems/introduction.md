# Introduction to Database Systems

**Data VS Information:** Information is data presented in context.

**Metadata** can include: structure, rules and constraints. Ensure consistency and meaning.

**Database** is a large, integrated, structured collection of data/

**Entities and relations:** Database systems manage data in a structured way.

Rows\&Columns forming relations. Keys\&Foreign keys to link relations.

**Advantages of Databases vs File processing Systems**

*   **Data independence**

    separation of data and program, application logic&#x20;

    central data repository, central management&#x20;
*   **Minimal data redundancy**&#x20;

    redundancy can be controlled (normalization)&#x20;
*   **Improved data consistency**&#x20;

    single store: no disagreements, update problems, less storage space&#x20;
*   I**mproved data sharing**&#x20;

    data is shared, a corporate resource, not a necessity for an application&#x20;

    external users can be allowed access multiple views of data, arbitrary views of data&#x20;
*   **Reduced program maintenance**&#x20;

    data structure can change without application data changing&#x20;
*   **Novel ad hoc data access** 'without programming'

    SQL

# Database development process

<img src="https://p.ipic.vip/2wi6b9.jpg" alt="Screenshot 2023-02-27 at 1.28.19 PM.png" data-size="original">

## Conceptual Design

Construction of a model of the data used in the database - independent of all physical considerations.

Data Models **ER Diagram (Entity Relation Diagram)**

![Screenshot 2023-02-27 at 1.50.41 PM.png](https://p.ipic.vip/8lkmsv.jpg)

Big boxes: entities

Blue diamond: mandatory

White diamond: optional

## Logical Design

Construction of a (relational) model of the data based on the conceptual design

Independent of a specific database and other physical considerations

![Screenshot 2023-02-27 at 2.20.23 PM.png](https://p.ipic.vip/i1t8bk.png)

Data modeling involves two distinct stages: conceptual design and logical design. The former stage aims to identify the highest-level relationships between different entities, while the latter provides a detailed description of data attributes without considering their physical implementation in the database. Conceptual design is more abstract and focuses on main entities and their relationships, whereas logical design specifies all attributes and primary keys within each entity.

Example:

Consider a library system that comprises authors, books, and publishers. A conceptual design would illustrate the relationships between these entities as follows:&#x20;

`Author writes --> Book published by --> Publisher.`

However, this design does not include their attributes or keys. To address this issue, we need to create a logical design that includes the following details:

1. Author (author\_id\*, name, email)
2. Book (book\_id\*, title, genre, year)
3. Publisher (publisher\_id\*, name, address)
4. Writes (author\_id\*, book\_id\*)
5. PublishedBy (book\_id\*, publisher\_id\*)

This revised version provides a clear and concise explanation of the two designs while maintaining coherence.

## Physical Design

For a specific DBMS

Describes:

* Basic relations(data types)
* File organization
* Indexes

Types help the DBMS store and use information efficiently

* Can make assumptions in computation
* Consistency is guaranteed

Minimize storage space

Need to consider

* Can you store all possible values
* Can the type you choose support the data manipulation required

Selection of types may improve data integrity

**Data Dictionary**

![Screenshot 2023-02-27 at 2.41.53 PM.png](https://p.ipic.vip/l4acnj.png)

### Physical Design Decisions

**(1)How to Store “Look Up”**

![Screenshot 2023-02-27 at 2.49.03 PM.png](https://p.ipic.vip/nbxymy.png)

Achieve data integerity in the left side. (Predefine the code in the look-up table)

There’s a trade-off between speed and space(and possibly integrity of data)

**(2)To De-Normalise or Not**
