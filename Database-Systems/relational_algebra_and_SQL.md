# Relational Algebra

- Selection($\sigma$): selects a subset of rows from relation
- Projection($\pi$): Retains only wanted colums from relation
- Cross-product($\times$): Allows us to combine two relations
- Set-difference($-$): Tuples in one relation, but not in the other
- Union($\cup$): Tuples in one relation and/or in the other

## Projection

Retains only attributes that are in the projection list

Also elimates the duplicate

## Selection

$\sigma_{rating>8}(S)$

Operations can be combined $\pi_{sname,rating}(\sigma_{rating>8}(S_2))$

## Union and Set-difference

These operations take two input relations, which must be **union-compatible.**

- Same number of fields
- Corresponding fields have the same type

## Compound Operator: Intersection

Intersection retains rows that appear in both sets

## Cross Product & Join

Cross product combines two relations:

- Each row of one input is merged with each row from another input
- Output is a new relation with all attributes of *both* inputs

The renaming operator: $\rho(C(1\rightarrow sid1, 5\rightarrow sid2), S1\times S2)$

And Here is cross join:

```sql
SELECT * FROM Rel1, Rel2; 
```

Joins are compound operators involving cross product, selection and sometimes projection.

Most common type: natural join $R\Join S$, a cross product that matches rows where attributes that appear in both relations have equal values( and we omit duplicate attributes). And there’s a projection in the final output which means duplicate attributes will be eliminated.

**Condition Join(or theta-join)** is a cross product with a condition.

$R\Join_C S= \sigma_C(R\times S)$

**Equi-Join** is a special case of condition join where condition $C$ only contains equalities.

Not same as natural join.

![Screenshot 2023-03-27 at 4.27.53 PM.png](https://p.ipic.vip/39wdpe.png)

Notes:

- Write Department $\Join$ Sale $\Join$ Item instead of Department $\Join$ Item $\Join$ Sale. The Department and Item do not share the same attributes.

- Find the employees whose salary is less than half that of their managers.

  $\rho(Employee\rightarrow TeamMember)$

  $\rho(Employee\rightarrow Boss)$

  $TeamMember \Join_{TeamMember. BossID = Boss.EmployeeID}Boss$

  $\sigma_{TeamMember.Salary<0.5\times Boss.Salary}(TeamMember \Join_{TeamMember. BossID = Boss.EmployeeID}Boss)$

# SQL

## Creating Table

```sql
CREATE TABLE Customer (
	CustomerID smallint auto_increment,
	CustFirstName varchar(100),
	CustMiddleName varchar(100),
	CustLastName varchar(100) NOT NULL,
	BussinessName varchar(200),
	CustType enum('Personal', 'Company') NOT NULL,
	PRIMARY KEY (CustomerID)
);
```

## Inset Data

```sql
INSERT INTO Customer(CustFirstName, CustLastName, CustType) VALUES ("Peter", "Smith", "Personal");

INSERT INTO Customer(DEFAULT, "James", NULL, "Jones", "JJ Enterprises", "Company");
```

MySQL doesn’t discard duplicates. To remove them use DISTINCT in front of the projection list

LIKE clause:

```sql
LIKE "REG_EXP"
```

% represents zero, one or multiple charactres.

_ represents a single character.

All of the aggregate functions except for `COUNT(*)` ignore null values and return null if all values are null. `COUNT(*)` counts the number of records.

**Limit and Offset**

Limit number limits the output size

Offset number skips ‘number’ records

**Join tables together**

```sql
SELECT * FROM Rel1, Rel2; 
```

```sql
SELECT * FROM Department NATRUAL JOIN Sale
```

The natural join automatically eliminates duplicate columns, while the inner join does not.

**Outer join: include records that don’t match the join from the other table**

## Set Operations

`UNION` shows all rows returned from teh queries(or tables)

`INTESECT` shows rows that are common in the queries(or the tables)

`UNION/INTESECT ALL` if you want duplicate rows in the results you need to use `ALL` keyword.

Union compatible are essentially two tables or expressions that have the same number of columns and corresponding columns are of same type.

## Sub-Query Comparison Operators

`IN / NOT IN` Used to test whether the attribute is IN/ NOT IN the subquery list

`ANY` True if any value returned meets the condition

`ALL` True if all values returned meet the condition

`EXISTS` True if the subquery returns one or more records

```sql
SELECT * FROM Buyer
WHERE BuyerID IN (SELECT BuyerID FROM offer WHERE ArtefactID = 1)
```

`JOIN` is a bit faster than nesting queries.

```sql
SELECT * FROM Buyer WHERE EXISTS
(SELECT * FROM Offer WHERE Buyer.BuyerID = Offer.BuyerID AND ArtefactID = 1)
```

Check the records in the Buyer table one by one.

`ALL` must satisfy all inner conditions

```sql
SELECT empno sal
FROM emp
WHERE sal > ALL(200,300,400)
```

## More on `INSERT`

Inserting records from a table

```sql
INSERT INTO NewEmployee
	SELECT * FROM Employee
```

Multiple record inserts:

```sql
INSERT INTO Employee VALUES
	(DEFAULT, "A", "A's Addr", "2012-02-02", NULL, "S"),
	(DEFAULT, "B", "B's Addr", "2012-02-02", NULL, "S"),
	(DEFAULT, "C", "C's Addr", "2012-02-02", NULL, "S");
```

```sql
INSERT INTO Employee
	(Name, Address, DateHired, EmployeeType)
	VALUES
		("D", "D's Addr", "2012-02-02", "C"),
		("E", "E's Addr", "2012-02-02", "C"),
		("F", "F's Addr", "2012-02-02", "C");
```

## The UPDATE Statement

Changes existing data in tables

Order of statement is important

Specifying a `WHERE` clause is important

```sql
UPDATE Hourly
	SET HourlyRate = HourlyRate * 1.10;
```

`CASE`

```sql
UPDATE Salaried
SET AnnualSalary = 
	CASE
		WHEN AnnualSalary <= 100000
		THEN AnnualSalary * 1.05
		ELSE AnnualSalary * 1.10
	END;
```

## Views

```sql
CREATE VIEW nameofview AS validsqlstateemnt
```

## More DDL Commands

`ALTER` Allows us to add or remove attributes (columns) from a relation (table)

```sql
ALTER TABLE TableName ADD AttributeName AttributeType
ALTER TABLE TableName DROP AttributeName
```

`RENAME`

```sql
RENAME TABLE CurrentTableName TO NewTableName
```

`TRUNCATE`

Same as `DELETE * FROM table`

Faster but cannot `ROLL BACK` a `TRUNCATE` command

`DROP`

Porentially dangerous

Kills a relation - removes the data, removes the relation

`DROP TABLE TableName`