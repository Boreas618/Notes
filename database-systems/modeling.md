# ER Modeling

**Entity** and **Entity Set**

An entity set is a collection of entities of the same type.

<img src="https://p.ipic.vip/1mhtty.png" alt="Screenshot 2023-03-05 at 7.00.34 PM.png" style="zoom: 35%;" />

<img src="https://p.ipic.vip/jzw23u.jpg" alt="Screenshot 2023-03-05 at 7.35.51 PM.png" style="zoom: 30%;" />

The same entity set can participate in the following:

- different relationship sets, or even
- different “roles” in the same set

<img src="https://p.ipic.vip/0kjwc6.jpg" alt="Screenshot 2023-03-05 at 7.40.03 PM.png" style="zoom:50%;" />

Take this pic as an example:

There are 2 relationships: **works in** and **reports to**

And the employees can have different roles: supervisor and subordinate 

**Key Constraints**

“Many” is represented by a line.

<img src="https://p.ipic.vip/81gpvg.png" alt="Screenshot 2023-03-05 at 7.51.46 PM.png" style="zoom:50%;" />

This is a one-to-many scenario.

**Participation Constraint**

Participation constraint explores whether all entities of one entity set take part in a relationship. If yes this is a **total** participation, otherwise it is partial. Total participation says thta each entity takes part in “**at least one**” relationship, and is represented by a bold line.

<img src="https://p.ipic.vip/j923ci.jpg" alt="Screenshot 2023-03-05 at 7.59.17 PM.png" style="zoom:50%;" />

The bold line is **total**.

**Weak Entities**

A weak entity can be identified uniquely only by considering (the primary key of) another (owner) entity. They are represented as “bold” rectangle.

Each weak entity has one and only one strong entity to depend on(key constraint)

Weak entity set must have total participation in this relationship set. Such relationship is called **identifying**.

Weak entities have only a “partial key”(dashed underline) and they are identified uniquely only when considering the primary key of the owner entity.

<img src="https://p.ipic.vip/06tyq0.png" alt="Screenshot 2023-03-05 at 8.18.07 PM.png" style="zoom:50%;" />

**Ternary Relationships(As opposed to Binary Relationships)**

In general, we can have **n**-ary relationships, and relationships can have attributes.

<img src="https://p.ipic.vip/0i3dmv.jpg" alt="Screenshot 2023-03-05 at 8.24.16 PM.png" style="zoom: 35%;" /><img src="https://p.ipic.vip/aghcki.png" alt="Screenshot 2023-03-05 at 8.26.52 PM.png" style="zoom:25%;" />

They are not the same!!! 

**Special attribute type: Multi-valued attributes**

Multi-valued attributes can have multiple (finite set of) values of the same type.

**Special attribute type: Composite attributes**

Have a structure inside

<img src="https://p.ipic.vip/ypppfo.jpg" alt="Screenshot 2023-03-05 at 8.28.59 PM.png" style="zoom:25%;" /><img src="https://p.ipic.vip/0rilse.jpg" alt="Screenshot 2023-03-05 at 8.30.30 PM.png" style="zoom:25%;" />

**Example**: Should “address” be an attribute of Employees or an entity (related to Employees)? 

**Answer**: Depends upon how we want to use address information, and the semantics of the data: If we have several addresses per employee, address must be an entity

ER design is subjective. There are often many ways to model a given scenario.

Summary:

Key constaints: one → many

Partcipation constraints: bold line

# Relational Model

Data Modes: Relational, ER, O-O, Network, Hierarchy

In relational model, we have rows and columns; We have keys and foreign keys to link relations.

Relational database is a set of relations. 

Relation is made up of 2 parts:

- Schema

  Specify the name of relation, plus name and type of each column(attribute)

  Example: Students(sid: string, name: string, gpa: real)

- Instance: A table with rows(Tuples or records) and columns(Attributes or fields)

  $\#rows = cardinality$ 

  $\#fields=degree(or\space arity)$

You can think of a relation as a set of rows or tuples. No order among rows.

In logical design, we translate ER model to relational model. Entity set becomes a relation. Attributes become attributes of the relation.

<img src="https://p.ipic.vip/bkzb2o.png" alt="Screenshot 2023-03-12 at 4.51.31 PM.png" style="zoom:25%;" />

## Keys

Keys are a way to assocaite tuples in different relations. Keys are one form if integerity constraint(IC). 

Primary keys cannot depulicate.

A primary key is a column or set of columns in a table that uniquely identifies each row or record in the table.

A foreign key is a column or set of columns in a table that refers to the primary key or a unique key in another table.

### Primary keys

A set of fields is a **superkey** if **no two distinct tuples can have same values in all key fields. 

A set of fields is a key for a realtion if it is a superkey and no subset of the fields is a superkey(minimal subset)

Out of all keys one is chosen to be the primary key of the relation. Other keys are called candidate keys.

Each relation has a primary key.

A primary key is not necessarily one column.

### Foreign keys

If all foreign key constraints are enforced in a DBMS, we say referential integrity is achieved.

![Screenshot 2023-03-12 at 5.19.25 PM.png](https://p.ipic.vip/4rbyg5.png)

In the table Enrolled, sid is a foreign key pointing to sid in Students. 

```sql
CREATE TABLE Enrolled
(sid CHAR(20),
cid CHAR(20),
grade CHAR(2),
PRIMARY KEY (sid, cid),
FOREIGN KEY (sid) REFERENCES Students
)
```

### Integerity Constraints

Definition: condition that must be true for any instance of the database; e.g., domain constraints

ICs are specified when schema is defined.

ICs are checked when relations are modified.

## Translating ER to Logical and Physical Model

**Composite attributes**: need to be unpacked when converting to relation models.

**Many to Many**: Keys for each participating entity set (as foreign keys). This set of attributes forms a superkey of the relation.

![Screenshot 2023-03-12 at 5.32.34 PM.png](https://p.ipic.vip/bbtivl.png)

```sql
CREATE TABLE Works_In
(ssn CHAR(11),
did INTEGER,
since DATE,
PRIMARY KEY(ssn, did),
FOREIGN KEY(ssn) REFERENCES Employee,
FOREIGN KEY(did) REFERENCES Department)
```

**Many to One:** 

Primary key from the many side becomes a foreign key on the one side.

For example, many: managers → one: departments

**Total participation:**

We specify total participation with key words NOT NULL.

```sql
CREATE TABLE Department(
	did INTEGER NOT NULL,
	dname CHAR(20) NOT NULL,
	budget FLOAT NULL,
	ssn CHAR(11) NOT NULL,
	since DATE NULL,
	PRIMARY KEY(did),
	FOREIGN KEY(ssn) REFERENCES Employee
		ON DELETE NO ACTION
)
```

**Weak Entity:**

```sql
CREATE TABLE Dependent(
	dname CHAR(20) NOT NULL,
	age INTEGER NULL,
	cost DECIMAL(7,2) NOT NULL,
	ssn CHAR(11) NOT NULL,
	PRIMARY KEY (dname, ssn),
	FOREIGN KEY (ssn) REFERENCES Employees
		ON DELETE CASCADE
)
```

Exam:

- Translate conceptual (ER) into logical & physical design
- Understand integrity constraints
- Use DDL of SQL to create tables with constraints

# Modeling with Crow's Foot Notation

Derived attributes imply that their values can be derived from some other attributes in the database. They do not need to be stored physically.

<img src="https://p.ipic.vip/dtg5qi.png" alt="Screenshot 2023-03-14 at 11.52.17 AM.png" style="zoom:50%;" /><img src="https://p.ipic.vip/pbgwfo.jpg" alt="Screenshot 2023-03-14 at 11.52.26 AM.png" style="zoom:50%;" />

Strong Entity and Weak Entity

<img src="https://p.ipic.vip/d6ifs6.jpg" alt="Screenshot 2023-03-14 at 11.58.08 AM.png" style="zoom:50%;" />

Note that the strong entities are linked by dash lines.

---

Convert from Conceptual to Logical design

Tasks checklist (from conceptual to logical):

- Flatten composite and multi-valued attributes
- Resolve many-may relationnships
- Resolve onne-many relationships

---

How to read a relationnship?

The customer has zero many accounts and each account is attached to and only one customer.

---

To deal with multi-valued attributes: 

- flatten the attribute
- use a look-up table one-many approach

<img src="https://p.ipic.vip/ijtjus.jpg" alt="Screenshot 2023-03-14 at 12.11.47 PM.png" style="zoom:50%;" /><img src="https://p.ipic.vip/auikdf.jpg" alt="Screenshot 2023-03-14 at 12.11.57 PM.png" style="zoom:50%;" />

- many-to-many approach

  Just share roles with others

  **associative entities**

  grab key from both sides of the relationship

---

To deal with one-one relationship

The rule is the OPTIMAL side of the relationship gets the foreign key.

![Screenshot 2023-03-14 at 12.23.42 PM.png](https://p.ipic.vip/tp871o.jpg)

---

**Summary of Binary Relationships From Conceptual to Logical**

- One-to-Many

  Primary key on the **one** side becomes a foreign key on the **many** side. **IN CASE OF CROW’S FOOT.**

- Many-to-Many

  Create an Associative Entity with the primary keys of the two entites it relates to as the combined primary key.

- One-to-One

  Need to decide where to put the foreign key

  The primary key on the mandatory side becomes a foreign key on the optional side.

  If two optional or two mandatory, pick one arbitrarily.

---

**Unary Relationship**

Operate in the same way as binary relationships

- One-to-One

  Put a Foreign key in the relation

- One-to-Many

  Put a Foreign key in the relation

- Many-to-Many

  Generate an Assoicative Entity

  Put two Foreign keys in the Associative Entity

  - Need 2 different names for the Foreign keys
  - Both Foreign keys become the *combined* key of the
    Associative Entity

Implementation:

```sql
CREATE TABLE Person(
	ID INT NOT NULL,
	Name VARCHAR(100) NOT NULL, 
	DateOfBirth DATE NOT NULL, 
	SpouseID INT,
	PRIMARY KEY (ID),
	FOREIGN KEY (SpouseID) REFERENCES Person (ID)
	ON DELETE RESTRICT
	ON UPDATE CASCADE);
```

<img src="https://p.ipic.vip/4pzb5j.png" alt="Screenshot 2023-03-14 at 1.20.43 PM.png" style="zoom:25%;" />

---

**Map an Identifying Relationship**

Foreign Key goes into the relationship at the crow’s foot end. The foreign key becomes **part of the primary key.**

---

Exam:

- Need to be able to draw conceptual, logical and physical disgrams
- Create table SQL statements