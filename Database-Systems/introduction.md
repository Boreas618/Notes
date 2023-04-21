# Introduction to Database Systems

**Data VS Information**

Information is data presented in context.

**Metadata**

Can include: structure, rules and constraints

Ensure consistency and meaning

**Database**

A large, integrated, structured collection of data

Entites and relations 

Database systems manage data in a structured way.

Rows&Colimns forming relations

Keys&Foreign keys to link relations

---

**Advantages of Databases vs File processing Systems**

![Screenshot 2023-02-27 at 1.25.07 PM.png](https://p.ipic.vip/wji4m1.png)

# Database development process

<img src="https://p.ipic.vip/2wi6b9.jpg" alt="Screenshot 2023-02-27 at 1.28.19 PM.png" style="zoom:50%;" />

## Conceptual Design

Construction of a model of the data used in the database - independent of all physical considerations.

Data Models **ER Diagram (Entity Relation Diagram)**

![Screenshot 2023-02-27 at 1.50.41 PM.png](https://p.ipic.vip/8lkmsv.jpg)

Big boxes: entities

Blue diamond: mandatory

White diamond: optional

## Logical Design

Construction of a (relational) model of the data based on the conceptual design

JSON or XML?

Independent of a specific database and other physical considerations

![Screenshot 2023-02-27 at 2.20.23 PM.png](https://p.ipic.vip/i1t8bk.png)

![Screenshot 2023-02-27 at 2.25.03 PM.png](https://p.ipic.vip/5m4v8c.png)

Example:

![Screenshot 2023-02-27 at 2.27.38 PM.png](https://p.ipic.vip/ldb882.jpg)

## Phydical Design

For a specific DBMS

Describes:

- Basic relations(data types)
- File organisation
- Indexes

Types help the DBMS store and use information efficiently

- Can make assumptions in computation
- Consistency is guaranteed

Minimise storage space

Need to consider

- Can you store all possible values
- Can the type you choose support the data manipulation required

Selection of types may improve data integrity

**Data Dictionary**

![Screenshot 2023-02-27 at 2.41.53 PM.png](https://p.ipic.vip/l4acnj.png)

### Physical Design Decisions

**(1)How to Store “Look Up”**

![Screenshot 2023-02-27 at 2.49.03 PM.png](https://p.ipic.vip/nbxymy.png)

Achieve data integerity in the left side. (Predefine the code in the look-up table)

There’s a trade-off between speed and space(and possibly integrity of data)

**(2)To De-Normalise or Not**
