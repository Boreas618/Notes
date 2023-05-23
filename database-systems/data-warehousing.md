# Data Warehousing

Informational Database vs. Transactional Database

**Warehouse: An informational Database**
* A single repository of organisational data
* Integrates data from multiple sources
* Makes data available to managers/users
* Support analysis and decision-making

Involve a large data store. 

Data warehouse supports analytical queries. 
* **Numerical aggregations**: How many? Average? Total cost?
* **Understanding dimensions**: Sales by state by customer type

**Characteristics of a Data warehouse**

* **Subject oriented** Data warehouses are organized around particular subjects (sales, customers, products)
* **Validated, Intergrated data** Data from different systems converted to a common format: allowing comparison and consolidation of data from different sources. Data from various sources validated before storing it in a data warehouse

* **Time variant** Historical data
* **Non-volatile** Users have read access only

-----

**A DW Architecture**

<img src="https://p.ipic.vip/oyo470.png" alt="Screenshot 2023-05-23 at 4.55.13 PM" style="zoom:50%;" />

# Dimensional Modeling

Dimensional Analysis: To support business analysis view:

Revenue (**Fact**) per product (**Dimension**) per customer (**Dimension**) per location (**Dimension**)

A dimensional model consists of:

* Fact table
* Serveral dimensional tables
* Hierarchies in the dimensions

Essentially a simple and restricted type of ER model

A fact table contains the actual business measures(additive, aggregates), called facts. The fact table also contains foreign keys pointing to dimensions. 

<img src="https://p.ipic.vip/qgydze.png" alt="Screenshot 2023-05-23 at 5.05.25 PM" style="zoom:50%;" />

The actual data for the fact table may look like this:

<img src="https://p.ipic.vip/wsxjht.png" alt="Screenshot 2023-05-23 at 5.07.05 PM" style="zoom:50%;" />

* Measures: Dollar sales and Unit Sales

* Granularity, or level of detail

  Finest level of detail for a fact table is determined by the finest level of each dimension.

  For example, suppose I want to know the hourly rainfall, daily rainfall and monthly rainfall, hourly rainfall is the finest level of detail.

<img src="https://p.ipic.vip/cith2c.png" alt="Screenshot 2023-05-23 at 5.17.19 PM" style="zoom:50%;" />

Fact table is an intersection table. 

## Steps

* Choose a Business Process
* Choose the measured facts
* Choose the granularity of the fact table
* Choose the dimensions
* Complete the dimension tables

## Normalized or Denomarlized

Normalization:

* Eliminates redundancy
* Storage effiency
* Referential integrity

Denormalization

* Fewer tables (fewer joins)
* Fast querying
* Design is tuned for end-user analysis

