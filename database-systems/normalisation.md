# Normalization

A technique used to remove undesired redundancy from databases ( Break one large table into serveral smaller tables )

A relation is normalized if all determinants are candidate keys.

## Anomalies

* Insertion Anomaly: A new course cannot be added until at least one student has enrolled (which comes first student or course?) 
* Deletion Anomaly: If student 425 withdraws, we lose all record of course C400 and its fee! 
* Update Anomaly: If the fee for course C200 changes, we have to change it in multiple records (rows), else the data will be inconsistent.

## Functional Dependency

A functional dependency concerns values of attributes in a relation.

A set of attributes $$X$$ **determine**s another set of attributes $$Y$$ if each value of $$X$$ is associated with only one value of $$Y$$.

Written $$X \rightarrow Y$$

 read as $$X$$ determines $$Y$$ (if I know $$X$$ Then I also know $$Y$$)

**Determinants $$(X, Y)\rightarrow Z$$**: the attributes on the left side of the arrow.

**Key and Non-Key attributes**: each attribute is either part of the primary key or it is not.

**Partial functional dependency $$(Y\rightarrow Z)$$**: a functional dependency of one or more non-key attributes upon part (but not all) of the primary key.

**Transitive dependency $$(Z\rightarrow D)$$**: a functional dependency between 2 or more non-key attributes.

### Armstrong's Axioms

Functional dependencies can be identified using Armstorng's Axioms

$$A=(X_1, X_2, \dots, X_n)$$

$$B=(Y_1, Y_2,\dots,Y_n)$$

1. Reflexivity: $$B\sub A \Rightarrow A\rightarrow B$$

   Example: `Student_ID, name` $$\rightarrow$$`name`

2. Augmentation: $$A\rightarrow B \Rightarrow AC \rightarrow BC$$
3. Transitivity: $$A\rightarrow B \and B\rightarrow C \Rightarrow A\rightarrow C$$

### Steps in Normalization

<img src="https://p.ipic.vip/zdo3fq.png" alt="Screenshot 2023-04-26 at 1.15.23 PM" style="zoom:50%;" />

## Normalizationvs Denormalization

It doesn't mean Denormalization is trash. Denormalization may be used to improve performance of time-critical operations.
