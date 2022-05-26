# Quick SQL Cheatsheet

original fork from enochtangg's github repo

A quick reminder of all relevant SQL queries and examples on how to use them.

In general is used the MySQL sytntax.

# [Table of Contents.](#tableofcontentes)

1. [SQL.](#intro)
2. [Data types & Custom Domain.](#datatypes)
3. [Database.](#database)
4. [Table.](#table)
5. [Finding Data Queries.](#find)
6. [Data Modification Queries.](#modify)
7. [Aggregate Functions.](#aggregatefunctions)
8. [Join.](#joins)
9. [View.](#view)
10. [Nested Queries or Subqueries.](#nestedqueries)

## Order of execution of a Query

    FROM + JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> ORDER BY -> LIMIT / OFFSET

<a name="intro"></a>

# 1. SQL

### DDL: Data Definition Language

to create, modify or delete the database and/or the schema of a database, its tables.

- `CREATE`
- `DROP`
- `ALTER`
- `TRUNCATE`
- `COMMENT`
- `RENAME`

### DQL: Data Query Language

- `SELECT`

### DML: Data Manipulation Language

to insert, fetch, modify or delete data stored in a database.

- `INSERT`
- `UPDATE`
- `DELETE`
- `LOCK`
- `MERGE`

### DCL: Data Control Language

to grant or revoke the credentials needed to use DML, DDL and DCL

- `GRANT`
- `REVOKE`

**[top](#tableofcontentes)**

<a name="datattypes"></a>

# 2. Data types & Custom Domain

### DATA TYPES

- string
  - one char: `CHAR`
  - strict fixed size: `CHAR(size)`
  - variable up to size `VARCHAR(size)`

- number
  - integer: `TINYINT` or `SMALLINT` or `INTEGER` ...look at the specific RDMBS to see how many bytes
  - exact decimal: `DECIMAL(precision, [scale])` or `NUMERIC(precision, [scale])`. precision: TOTAL number of digits allowed (decimal) or needed (numeric) and scale: number of digits on the right of the decimal point.
  - floating point: `FLOAT(precision)` or `REAL`

** example:
<br/>DECIMAL(6,4) [-99.9999, ..., +99.9999]
<br/>NUMERIC(3,2):  [-9.99, ..., +9.99]

- date
  - `DATE` with fields: `year,month,day`
- time
  - `TIME` with fields: `hour, minute, second`
- date + time
  - `TIMESTAMP` with fields: `year, month, day, hour, minute, second`

** example

```sql
CREATE TABLE orders (
  id INT NOT NULL AUTO_INCREMENT,
  ...,
  order_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
  PRIMARY KEY (id)
  [ constraints ]
)
```

more on [MySQL](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html)

- true/false
  - `BOOLEAN`
- binary large object (img, video, ...)
  - `BLOB`
- charatecte large object (long text)
  - `CLOB`

### CUSTOM DOMAIN

❗ it can be useful to define a domain including the related constraints, the **following syntax doensn't work on MySQL**. ❗

```sql
CREATE DOMAIN DOMAIN_NAME AS <data_type>
DEFAULT <default_value> | NULL
CHECK condition_to_specify_the_valid_values
```

condition_to_specify_the_valid_values can be as follows:

  ```sql
  CHECK (column >= value_down AND column <= value_up)
  ```

  ```sql
  CHECK (column IN (range_of_valid_values))
  ```

  ```sql
  CHECK (column LIKE 'regex_here')
  ```

** example

```sql
CREATE DOMAIN SCORE
AS SMALLINT
DEFAULT NULL
CHECK (value >= 0 AND value <= 100)
```

if you prefer alter an existing table e.g. exams(..., score,...)

- setting a default value for a column

```sql
ALTER TABLE exams ALTER COLUMN score
SET DEFAULT 0
```

- adding a contraint for a column value

```sql
ALTER TABLE exams
ADD CONSTRAINT
CHECK (score >= 1 AND score <= 20);
```

**[top](#tableofcontentes)**

<a name="database"></a>

# 3. Database

- to create a database

```sql
CREATE DATABASE db_name
```

- to delete a database

```sql
DROP DATABASE db_name
```

    ✅ snake_case
    ✅ 30 bytes long (1 byte = 1 char ASCII)

**[top](#tableofcontentes)**

<a name="table"></a>

# 4. Table

- to creata a table for each column we:
  - **MUST** specify column_name, domain or data-type
  - **CAN** include the default value, precision for domain or data-type, if it can be nullable

- general syntax

```sql
CREATE TABLE table_name (
  coloumn_name_1 Domain_1 [DEFAULT <value_1>] [column_contraint],
  ...,
  coloumn_name_x Domain_x [DEFAULT <value_x>] [column_contraint]
)
```

❗ MOST COMMON **column**_constraint:

- `NOT NULL` when the column cannot be null

```sql
column_name Domain NOT NULL
```

- `UNIQUE` to ensure that **all values** in the column are *different*

```sql
column_name Domain UNIQUE
```

**❗ `UNIQUE` can represent a UNIQUE BUNDLE of values, in this case the unique value is the combination of n columns

```sql
column_1 Domain_1,
column_2 Domain_2,
column_3 Domain_3,
...
UNIQUE (column_1, column_2, column_3)
```

- `NOT NULL UNIQUE` to **indentify** a **row** in a table, uniquely. It can be multi columns (see above). Several not-null-unique constraints on the same table are possible.

```sql
column_name Domain NOT NULL UNIQUE
```

- `PRIMARY KEY` to **indentify** a **row** in a table, uniquely.

>The PRIMARY KEY constraint uniquely identifies each record in a table.
Primary keys must contain UNIQUE values, and cannot contain NULL values.
A table can have only ONE primary key; and in the table, this primary key can consist of single or multiple columns (fields). [W3Schools](https://www.w3schools.com/sql/sql_primarykey.asp)

```sql
column_name Domain PRIMARY KEY
```

- `CHECK` (condition_to_specify_valid_values_for_column)
- `DEFAULT` to set the value of the column if not given

❗**table**_constraint:

- `FOREING KEY`

  > The FOREIGN KEY constraint is used to prevent actions that would destroy links between tables.
  A FOREIGN KEY is a field (or collection of fields) in one table, that refers to the PRIMARY KEY in another table. The table with the foreign key is called the child table, and the table with the primary key is called the referenced or parent table. [W3Schools](https://www.w3schools.com/sql/sql_foreignkey.asp)

**example

```sql
CREATE TABLE IF NOT EXISTS boos(
    book_id VARCHAR(15) PRIMARY KEY,
    title VARCHAR(50),
    isbn_no VARCHAR(15) NOT NULL UNIQUE,
    publication DATE CHECK (publication LIKE '--/--/----')
);
```

### **column** or **table** constraint can be specificed

1. during the creation of the table

```sql
CREATE TABLE matches(
    id_player INT NOT NULL,
    id_monster INT NOT NULL,
    no_enemies INT NOT NULL,
    experience INT NOT NULL,
    CONSTRAINT pk_match PRIMARY KEY(id_player, id_monster),
    CONSTRAINT fk_match_player FOREIGN KEY(id_player) REFERENCES players(id_player) ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_match_monster FOREIGN KEY(id_monster) REFERENCES monsters(id_monster) ON UPDATE CASCADE ON DELETE CASCADE
);
```

2. added later on

```sql
CREATE TABLE matches(
    id_player INT NOT NULL,
    id_monster INT NOT NULL,
    no_enemies INT NOT NULL,
    experience INT NOT NULL
);
```

```sql
ALTER TABLE
    matches ADD CONSTRAINT pk_match PRIMARY KEY(id_player, id_monster);
```

```sql
ALTER TABLE
    matches ADD CONSTRAINT fk_match_player FOREIGN KEY(id_player) REFERENCES players(id_player) ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_match_monster FOREIGN KEY(id_monster) REFERENCES monsters(id_monster) ON UPDATE CASCADE ON DELETE CASCADE;
```


### **ADD**: add a column

```sql
ALTER TABLE table_name ADD COLUMN column_name data_type_or_domain [constraints];
```

### **MODIFY**: change data type of column

```sql
ALTER TABLE table_name MODIFY COLUMN column_name new_data_type_or_domain [constraints];
```

### **DROP**: delete a column

```sql
ALTER TABLE table_name DROP COLUMN column_name;
```

**[top](#tableofcontentes)**

<a name="find"></a>

# 5. Finding Data Queries

### **SELECT**: used to select data from a database

- `SELECT` * `FROM` table_name;

### **SELECT**: general syntax

```sql
SELECT [DISTINCT] target_list [AS alias_column]
[ FROM table_name ] [AS alias_table]
[ WHERE condition ]
[ GROUP BY group_by_expression ]
[ HAVING condition ]
[ ORDER BY column_in_target_list [ ASC | DESC ] ]
```

#### **SELECT** + **ALIAS** to modify value and/or name of a column in the result-set

```sql
SELECT column1, (TemperatureCelsius+273,15) AS TemperatureKelvin FROM table_name
```

### **DISTINCT**: filters away duplicate values and returns rows of specified column

- `SELECT DISTINCT` column_name;

### **WHERE**: used to filter records/rows

- `SELECT` column1, column2 `FROM` table_name `WHERE` condition;

- `SELECT` * `FROM` table_name `WHERE` condition1 `AND` condition2;
- `SELECT` * `FROM` table_name `WHERE` condition1 `OR` condition2;
- `SELECT` * `FROM` table_name `WHERE NOT` condition;
- `SELECT` * `FROM` table_name `WHERE` condition1 `AND` (condition2 `OR` condition3);
- `SELECT` * `FROM` table_name `WHERE EXISTS` (`SELECT` column_name `FROM` table_name `WHERE` condition);

### **WHERE** + **NULL**: used to filter records/rows in case of NULL value

In case of condition on columns with a possible null value, the result is always ***false***

```sql
SELECT * FROM table_name WHERE column1 IS NULL;
SELECT * FROM table_name WHERE column1 IS NOT NULL;
```

### **ORDER BY**: used to sort the result-set in ascending or descending order

```sql
SELECT * FROM table_name WHERE condition ORDER BY column;
SELECT column1, column2 FROM table_name ORDER BY column1 DESC;
SELECT column1,column2,column3 FROM table_name ORDER BY column2 [ASC], column1 DESC;
```

- ❗ the columns used to sort the result-set must be in the target list

### **SELECT TOP**: used to specify the number of records to return from top of table

- `SELECT TOP` number columns_names `FROM` table_name `WHERE` condition;

- `SELECT TOP` percent columns_names `FROM` table_name `WHERE` condition;
- Not all database systems support `SELECT TOP`. The MySQL equivalent is the `LIMIT` clause
- `SELECT` column_names `FROM` table_name `LIMIT` offset, count;

### **LIKE**: operator used in a WHERE clause to search for a specific pattern in a column

- % (percent sign) is a wildcard character that represents zero, one, or multiple characters

- _ (underscore) is a wildcard character that represents a single character
- `SELECT` column_names `FROM` table_name `WHERE` column_name `LIKE` pattern;
- `LIKE` ‘a%’ (find any values that start with “a”)
- `LIKE` ‘%a’ (find any values that end with “a”)
- `LIKE` ‘%or%’ (find any values that have “or” in any position)
- `LIKE` ‘_r%’ (find any values that have “r” in the second position)
- `LIKE` ‘a_%_%’ (find any values that start with “a” and are at least 3 characters in length)
- `LIKE` ‘[a-c]%’ (find any values starting with “a”, “b”, or “c”

### **IN**: operator that allows you to specify multiple values in a WHERE clause

- essentially the IN operator is shorthand for multiple OR conditions

- `SELECT` column_names `FROM` table_name `WHERE` column_name `IN` (value1, value2, …);
- `SELECT` column_names `FROM` table_name `WHERE` column_name `IN` (`SELECT STATEMENT`);

### **BETWEEN**: operator selects values within a given range inclusive

- `SELECT` column_names `FROM` table_name `WHERE` column_name `BETWEEN` value1 `AND` value2;

- `SELECT` * `FROM` Products `WHERE` (column_name `BETWEEN` value1 `AND` value2) `AND NOT` column_name2 `IN` (value3, value4);
- `SELECT` * `FROM` Products `WHERE` column_name `BETWEEN` #01/07/1999# AND #03/12/1999#;

### **AS**: aliases are used to assign a temporary name to a table or column

- `SELECT` column_name `AS` alias_name `FROM` table_name;

- `SELECT` column_name `FROM` table_name `AS` alias_name;
- `SELECT` column_name `AS` alias_name1, column_name2 `AS` alias_name2;
- `SELECT` column_name1, column_name2 + ‘, ‘ + column_name3 `AS` alias_name;

```sql
SELECT column1, (TemperatureCelsius+273,15) AS TemperatureKelvin FROM table_name WHERE TemperatureKelvin > 298 ORDER BY TemperatureKelvin
```

### **SET OPERATORs**: general syntax

```sql
SELECT targetList_1
FROM table_1
WHERE condition_1
                      UNION / INTERSECT / EXCEPT
SELECT targetList_2
FROM table_2
WHERE condition_2
```

- ❗ `targetList_1` must be the same (number, order, datatype) of `targetList_2`

- ❗ SET OPERATOR selects distinct values, to include duplicate use `<SET OPERATOR> ALL`

### **UNION**: set operator used to combine the result-set of two or more SELECT statements

<br/>
equivalent form with JOIN + OR-in-WHERE

```sql
SELECT [DISTINCT] targetList_1
FROM table_1 JOIN table_2 ON table_1.col = table_2.col
WHERE condition_1 OR condition_2
```

### **INTERSECT**: set operator which is used to return the records that two SELECT statements have in common

<br/>
equivalent form with JOIN + AND-in-WHERE

```sql
SELECT [DISTINCT] targetList_1
FROM table_1 JOIN table_2 ON table_1.col = table_2.col
WHERE condition_1 AND condition_2
```

### **EXCEPT**: set operator used to return all the records in the first SELECT statement that are not found in the second SELECT statement

<br/>
equivalent form with subquery (see later) + NOT IN

```sql
SELECT [DISTINCT] targetList_1
FROM table_1 
WHERE condition_1 AND columnX NOT IN (
                                      SELECT columnX
                                      FROM table_2
                                      WHERE condition_2
                                      )
```

- ❗ columnX can be replaced with (columnA, columnB, ...)

### **ANY | ALL**: operator used to check subquery conditions used within a WHERE or HAVING clauses

- The `ANY` operator returns true if any subquery values meet the condition

- The `ALL` operator returns true if all subquery values meet the condition
- `SELECT` columns_names `FROM` table1 `WHERE` column_name operator (`ANY`|`ALL`) (`SELECT` column_name `FROM` table_name `WHERE` condition);

### **GROUP BY**: statement often used with aggregate functions (COUNT, MAX, MIN, SUM, AVG) to group the result-set by one or more columns

```sql
SELECT column_name1, COUNT(column_name2)
FROM table_name
WHERE condition
GROUP BY column_name1
ORDER BY COUNT(column_name2) DESC;
```

- ❗ GROUP BY + SELECT => target list must be made up of columns (***all or just some***) IN GROUP BY clause

- ❗ target list can contain AGGREGATE FUNCTIONS
- ❗ SQL-99, target list can contain columns even if they don't appear in GROUP BY clause, **but** the GROUP BY clause must contain some UNIQUE NOT NULL column

### **HAVING**: in the WHERE clause you can specify condition to be validated row by row, but if you need a condition to be validated on a group of rows, place it in the HAVING clause

```sql
SELECT column_name2, COUNT(column_name1)
FROM table_name
GROUP BY column_name2
HAVING COUNT(column_name1) > 5;
```

**[top](#tableofcontentes)**

<a name="modify"></a>

# 6. Data Modification Queries

### **INSERT INTO**: used to insert new records/rows in a table

- insert: with specificed pairs (column_i : value_i)

- ❗ for each column not specified the value is: `NULL`(if nullable) or the default-value

```sql
INSERT INTO table_name (column1, ..., columnx) VALUES (value1, ..., valuex)
```

- insert: in case values are specified for **ALL** columns in table

```sql
INSERT INTO VALUES (value1, ..., valuex)
```

- insert: **MULTIPLE** rows

```sql
INSERT INTO table_name (column1, ..., columnx)
  VALUES 
    (value1, ..., valuex) ,
    (value1, ..., valuex) ,
    (value1, ..., valuex) ,
    ...
    (value1, ..., valuex) ,
```

- insert: **MULTIPLE** rows **from** an other table

```sql
INSERT INTO table_name (target_list)
  SELECT ( target_list FROM table_name_2 WHERE condition_on_table_name_2 )
```

- ❗ targetl_list: same domain, order, number of columns

### **UPDATE**: used to modify the existing records in a table

- `UPDATE` table_name `SET` column1 = value1, column2 = value2 `WHERE` condition;

- `UPDATE` table_name `SET` column_name = value;

### **DELETE**: used to delete existing records/rows in a table

- `DELETE FROM` table_name `WHERE` condition;

- `DELETE` * `FROM` table_name;

**[top](#tableofcontentes)**

<a name="aggregatefunctions"></a>

# 7. Aggregate Funtions

> An aggregate function performs a calculation on a set of values to return a single value,
> got from the application of that specific function.

### **AGGREGATE FUNCTION**: general syntax

```sql
SELECT column_X, CONT | SUM | AVG | MIN | MAX (column)
FROM table_name
[WHERE tuple_condition]
[GROUP BY column_X]
[ORDER BY column]
```

- ❗ just one aggreate function at a time, in the SELECT clause

- ❗ aggregate functions performed after WHERE clause

### **COUNT**: returns the # of occurrences

- `SELECT COUNT (DISTINCT` column_name`)`;

### **MIN() and MAX()**: returns the smallest/largest value of the selected column

- `SELECT MIN (`column_names`) FROM` table_name `WHERE` condition;

- `SELECT MAX (`column_names`) FROM` table_name `WHERE` condition;

### **AVG()**: returns the average value of a numeric column

- `SELECT AVG (`column_name`) FROM` table_name `WHERE` condition;

### **SUM()**: returns the total sum of a numeric column

- `SELECT SUM (`column_name`) FROM` table_name `WHERE` condition;

**[top](#tableofcontentes)**

<a name="joins"></a>

# 8. Join

### **JOIN**: general syntax

```sql
SELECT [DISTINCT] target_list [AS alias_column]
FROM table1_name [AS alias_table] [type_of_join] JOIN table2_name ON join_condition [ [type_of_join] JOIN table3_name ON join_condition ...]
[ WHERE tuple_of_tableX_name_condition ]
```

### types of join ###

### **INNER JOIN**: returns records that have matching value in both tables

```sql
SELECT column_names FROM table1 [INNER] JOIN table2 ON table1.column_name=table2.column_name;
```

```sql
SELECT table1.column_name1, table2.column_name2, table3.column_name3 FROM ((table1 INNER JOIN table2 ON relationship) INNER JOIN table3 ON relationship)
```
❗ parenthesis order, necessary to build the sequence of joins to perform

### **LEFT (OUTER) JOIN**: returns all records from the left table (table1), and the matched records from the right table (table2)

```sql
SELECT column_names FROM table1 LEFT JOIN  table2 ON table1.column_name=table2.column_name;
```

### **RIGHT (OUTER) JOIN**: returns all records from the right table (table2), and the matched records from the left table (table1)

```sql
SELECT column_names FROM table1 RIGHT JOIN  table2 ON table1.column_name=table2.column_name;
```

### **FULL (OUTER) JOIN**: returns all records when there is a match in either left or right table

```sql
SELECT column_names FROM table1 FULL OUTER JOIN  table2 ON table1.column_name=table2.column_name;
```

- 

- ❗ with OUTER-JOIN, you should have to ***handle NULL values in WHERE clause***

### **Self JOIN**: a regular join, but the table is joined with itself

```sql
SELECT target_list
FROM table_name T1 JOIN table_name T2 ON T1.columnA = T2.columnA
[ WHERE condition_to_remove_duplicates ]
```

- ❗ JOIN-condition can be performed on any column (same table remember)

- ❗ in WHERE clause you can specify a particular condition to remove duplicates: same row but considered twice or same rows but in reverse ordere. To do this you can use the **<** operator between T1.columnB and T2.columnB for example
- ❗ if columnA is not a PK, and if columnB is not a PK, the condition_to_remove_duplicates could be contain the following: <br /> T1.PK < T2.PK AND T1.columnB <> T2.columnB.
<br/>**But**<br />
$PK \Rightarrow {other\;column\;values}$
<br/>so<br/>
(PK_1 == PK_2) --> (other_column_values_1 == other_column_values_2)
<br/>equals to<br/>
NOT (other_column_values_1 == other_column_values_2) --> NOT (PK_1 == PK_2)
<br/>so the condition can be simplified as follows:<br/>
~~T1.PK < T2.PK AND~~ **T1.columnB < T2.columnB**

**[top](#tableofcontentes)**

<a name="view"></a>

# 9. View

### **CREATE**: create a view

- `CREATE VIEW` view_name `AS SELECT` column1, column2 `FROM` table_name `WHERE` condition;

### **SELECT**: retrieve a view

- `SELECT` * `FROM` view_name;

### **DROP**: drop a view

- `DROP VIEW` view_name;

**[top](#tableofcontentes)**

<a name="nestedqueries"></a>

# 10. Nested Queries or Subqueries

### **NESTED QUERY** (or subquery) is a query nested inside another query

- ⛩️ DIVIDE ET IMPERA approach.

- used to return a **single-value** (*single row and single column*), a **single-column** (*single column but multiple rows*) or a **multiple-columns** (*multiple columns and multiple rows*).
- **non**-correlated subquery is evaluated before the outer query, just one time.
- **correlated** subquery is evaluated *once for each row of the outer query processed*, this because the subquery refers to a column that is not in the FROM clause, but is related to the outer query-table.
- used:
  - within `WHERE` clause to **filter** data/rows
  - within `HAVING` clause to **group** using, for example, an aggregate function
  - within `FROM` clause

#### syntax: **single-value** nested query in WHERE clause with *COMPARISON OPERATOR*

```sql
SELECT target_list_1
FROM table_1 T1
WHERE columnX {=, <>, <, <=, >, >=} ( SELECT columnX
                                      FROM table_2 T2
                                      WHERE condition_on_T2_PK )
```

- ❗ single-value subquery so:
  - one column, columnX, same domain
  - one value, condition_on_T2_**PK** in subquery ***selects just one row using the Primary Key***

#### syntax: **single-column** nested query in WHERE clause with *COMPARISON OPERATOR + **ANY/ALL***

```sql
SELECT target_list_1
FROM table_1 T1
WHERE columnX {=, <>, <, <=, >, >=} ANY/ALL ( SELECT columnX
                                              FROM table_2 T2
                                              WHERE condition_on_T2 )
```

- ❗ single-column subquery so:
  - one column, columnX, same domain
  - multiple rows, values.

- ❗ `ANY` return true if any of the subqueries values meet the condition with the comparison operator.
  - pattern to select non-max values for columnX with ANY and Table subquery

  ```sql
  ... WHERE columnX < ANY (SELECT columnX FROM table_2)
  ```

  - alternative with aggregate function and Single-Value subquery

  ```sql
  ... WHERE columnX < (SELECT MAX(columnX) FROM table_2)
  ```

- ❗ `ALL` return true if all of the subqueries values meet the condition with the comparison operator

  - pattern to select max value for columnX with ALL, **NULLABLE** column and Table subquery

  ```sql
  ... WHERE columnX >= ALL (SELECT columnX FROM table_2 WHERE columnX IS NOT NULL)
  ```

  - alternative with aggregate function and Single-Value subquery, where columnX can be NULL

  ```sql
  ... WHERE columnX = (SELECT MAX(columnX) FROM table_2)
  ```

#### syntax: **single-column** nested query in WHERE clause with **IN**

```sql
SELECT target_list_1
FROM table_1 T1
WHERE columnX [NOT] IN ( SELECT columnX 
                          FROM table_2 T2
                          WHERE condition_on_T2 )
```

- ❗ the subquery must select for a column value (columnX) to produce a list of values for the outer columnX

#### syntax: **multiple-columns** nested query in WHERE clause with **IN**

```sql
SELECT target_list_1
FROM table_1 T1
WHERE (columnA, columnB) [NOT] IN (  SELECT columnA, columnB
                                      FROM table_2 T2
                                      WHERE condition_on_T2 )
```

- ❗ the subquery must select for multiple columns, can contain an aggregate function combined to a column to produce multiple rows for example

  ```sql
  ... WHERE (id, Qty) IN (SELECT id, MIN(Qty) FROM table_2 GROUP BY id)
  ```

#### syntax: **single-value** nested query in SELECT clause

- almost a correlated subquery --> **T1**.columnA in WHERE clause in the nested query

- used to display in the result-set a data related to a specific row of the outer query

```sql
SELECT target_list_1, (
                        SELECT columnX / expression
                        FROM table_2 T2
                        WHERE T1.columnA = T2.columnB ... AND condition_on_T2_PK
                      )
FROM table_1 T1
[ WHERE condition_on_T1 ]
```

- ❗ the subquery must select for a single value, so for example columnX can be used in combination to condition_on_T2_PK for example

  ```sql
  SELECT target_list_1, (
                          SELECT columnX
                          FROM table_2 T2
                          WHERE T1.columnA = T2.columnB AND T2.PK = <_value_>
                        )
  ...
  ```

   ***otherwise*** the expression can be an aggregate function for example

  ```sql
  SELECT target_list_1, (
                          SELECT MIN(Qty)
                          FROM table_2 T2
                          WHERE T1.columnA = T2.columnB
                        )
  ...
  ```

#### syntax: **multiple-columns** nested query in FROM clause

- never a correlated subquery

```sql
SELECT target_list_1
FROM table_1 T1 [OUTER] JOIN ( SELECT columnA [, AGGREGATE_FUNC(columnB)]
                               FROM table_2 T2
                               WHERE condition_on_T2 
                               [GROUP BY columnA]
                               )
                        ON T1.columnA = T2.columnA
[ WHERE condition_on_T1_T2 ]
```

#### syntax: **non-empty / empty SET** as result of a correlated subquery in WHERE clause + EXISTS / NOT EXISTS

- **EXISTS**, correlated subquery returns a **non**-empty set if the condition in where clause, in the inner query, is **satisfed** by the currently evaluated row of the outer query

- **NOT EXISTS**, correlated subquery returns an **empty** set if the condition in where clause, in the inner query, is **not** satisfed by the currently evaluated row of the outer query  

```sql
SELECT target_list_1
FROM table_1 T1
WHERE [NOT] EXISTS            ( SELECT *
                                FROM table_2 T2
                                WHERE T1.columnA = T2.columnB AND / OR ...
                              )
[AND condition_on_T1 ]
```

**[top](#tableofcontentes)**