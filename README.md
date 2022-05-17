# Quick SQL Cheatsheet

original fork from enochtangg's github repo

A quick reminder of all relevant SQL queries and examples on how to use them. 

This repository is constantly being updated and added to by the community. 
Pull requests are welcome. Enjoy!

# Table of Contents
1. [ SQL. ](#intro)
2. [ Data types. ](#datatypes)
3. [ Finding Data Queries. ](#find)
4. [ Data Modification Queries. ](#modify)
5. [ Aggregate Functions ](#aggregatefunctions)
6. [ Join Queries. ](#joins)
7. [ View Queries. ](#view)
8. [ Altering Table Queries.](#alter)
9. [ Creating Table Query.](#create)


<a name="intro"></a>
# 1. SQL as:
### DDL: Data Definition Language
- SELECT
- INSERT
- CREATE
- ALTER

### DML: Data Manipulation Language
- CREATE DOMAIN
- {CREATE, DROP, USE} DATABASE
```sql
    CREATE DATABASE database_name;
    DROP DATABASE database_name;
    USE database_name;
```
- {CREATE, ALTER, DROP} TABLE
- {CREATE, ALTER, DROP} VIEW
- {CREATE, DROP} INDEX
- GRANT, REVOKE
- COMMIT, ROLLBACK

## Order of execution of a Query
FROM + JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> ORDER BY -> LIMIT / OFFSET
<br/>
<br/>
<a name="datattypes"></a>


<a name="find"></a>
# 1. Finding Data Queries

### **SELECT**: used to select data from a database
* `SELECT` * `FROM` table_name;

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
* `SELECT DISTINCT` column_name;

### **WHERE**: used to filter records/rows
* `SELECT` column1, column2 `FROM` table_name `WHERE` condition;
* `SELECT` * `FROM` table_name `WHERE` condition1 `AND` condition2;
* `SELECT` * `FROM` table_name `WHERE` condition1 `OR` condition2;
* `SELECT` * `FROM` table_name `WHERE NOT` condition;
* `SELECT` * `FROM` table_name `WHERE` condition1 `AND` (condition2 `OR` condition3);
* `SELECT` * `FROM` table_name `WHERE EXISTS` (`SELECT` column_name `FROM` table_name `WHERE` condition);

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
* ❗ the columns used to sort the result-set must be in the target list

### **SELECT TOP**: used to specify the number of records to return from top of table
* `SELECT TOP` number columns_names `FROM` table_name `WHERE` condition;
* `SELECT TOP` percent columns_names `FROM` table_name `WHERE` condition;
* Not all database systems support `SELECT TOP`. The MySQL equivalent is the `LIMIT` clause
* `SELECT` column_names `FROM` table_name `LIMIT` offset, count;

### **LIKE**: operator used in a WHERE clause to search for a specific pattern in a column
* % (percent sign) is a wildcard character that represents zero, one, or multiple characters
* _ (underscore) is a wildcard character that represents a single character
* `SELECT` column_names `FROM` table_name `WHERE` column_name `LIKE` pattern;
* `LIKE` ‘a%’ (find any values that start with “a”)
* `LIKE` ‘%a’ (find any values that end with “a”)
* `LIKE` ‘%or%’ (find any values that have “or” in any position)
* `LIKE` ‘_r%’ (find any values that have “r” in the second position)
* `LIKE` ‘a_%_%’ (find any values that start with “a” and are at least 3 characters in length)
* `LIKE` ‘[a-c]%’ (find any values starting with “a”, “b”, or “c”

### **IN**: operator that allows you to specify multiple values in a WHERE clause
* essentially the IN operator is shorthand for multiple OR conditions
* `SELECT` column_names `FROM` table_name `WHERE` column_name `IN` (value1, value2, …);
* `SELECT` column_names `FROM` table_name `WHERE` column_name `IN` (`SELECT STATEMENT`);

### **BETWEEN**: operator selects values within a given range inclusive
* `SELECT` column_names `FROM` table_name `WHERE` column_name `BETWEEN` value1 `AND` value2;
* `SELECT` * `FROM` Products `WHERE` (column_name `BETWEEN` value1 `AND` value2) `AND NOT` column_name2 `IN` (value3, value4);
* `SELECT` * `FROM` Products `WHERE` column_name `BETWEEN` #01/07/1999# AND #03/12/1999#;

### **AS**: aliases are used to assign a temporary name to a table or column
* `SELECT` column_name `AS` alias_name `FROM` table_name;
* `SELECT` column_name `FROM` table_name `AS` alias_name;
* `SELECT` column_name `AS` alias_name1, column_name2 `AS` alias_name2;
* `SELECT` column_name1, column_name2 + ‘, ‘ + column_name3 `AS` alias_name;
```sql
SELECT column1, (TemperatureCelsius+273,15) AS TemperatureKelvin FROM table_name WHERE TemperatureKelvin > 298 ORDER BY TemperatureKelvin
```

### **SET OPERATOR**: general syntax
```sql
SELECT TargetList_1
FROM Table_1
WHERE condition_1
                      UNION / INTERSECT / EXCEPT
SELECT TargetList_2
FROM Table_2
WHERE condition_2
```
* ❗ `TargetList_1` must be the same (number, order, datatype) of `TargetList_2`
* ❗ SET OPERATOR selects distinct values, to include duplicate use `ALL`

### **UNION**: set operator used to combine the result-set of two or more SELECT statements
<br/>
equivalent form with JOIN + OR-in-WHERE

```sql
SELECT TargetList_1
FROM Table_1 JOIN Table_2 ON Table_1.col = Table_2.col
WHERE condition_1 OR condition_2
```

### **INTERSECT**: set operator which is used to return the records that two SELECT statements have in common
<br/>
equivalent form with JOIN + AND-in-WHERE

```sql
SELECT TargetList_1
FROM Table_1 JOIN Table_2 ON Table_1.col = Table_2.col
WHERE condition_1 AND condition_2
```

### **EXCEPT**: set operator used to return all the records in the first SELECT statement that are not found in the second SELECT statement
* Generally used the same way as **UNION** above
* `SELECT` columns_names `FROM` table1 `EXCEPT SELECT` column_name `FROM` table2;

### **ANY|ALL**: operator used to check subquery conditions used within a WHERE or HAVING clauses
* The `ANY` operator returns true if any subquery values meet the condition
* The `ALL` operator returns true if all subquery values meet the condition
* `SELECT` columns_names `FROM` table1 `WHERE` column_name operator (`ANY`|`ALL`) (`SELECT` column_name `FROM` table_name `WHERE` condition);

### **GROUP BY**: statement often used with aggregate functions (COUNT, MAX, MIN, SUM, AVG) to group the result-set by one or more columns
```sql
SELECT column_name1, COUNT(column_name2)
FROM table_name
WHERE condition
GROUP BY column_name1
ORDER BY COUNT(column_name2) DESC;
```
* ❗ GROUP BY + SELECT => target list must be made up of columns (**_all or just some_**) IN GROUP BY clause
* ❗ target list can contain AGGREGATE FUNCTIONS
* ❗ SQL-99, target list can contain columns even if they don't appear in GROUP BY clause, **but** the GROUP BY clause must contain some UNIQUE NOT NULL column


### **HAVING**: in the WHERE clause you can specify condition to be validated row by row, but if you need a condition to be validated on a group of rows, place it in the HAVING clause.
```sql
SELECT column_name2, COUNT(column_name1)
FROM table_name
GROUP BY column_name2
HAVING COUNT(column_name1) > 5;
```

### **WITH**: often used for retrieving hierarchical data or re-using temp result set several times in a query. Also referred to as "Common Table Expression"
* `WITH RECURSIVE` cte `AS` (<br/>
    &nbsp;&nbsp;`SELECT` c0.* `FROM` categories `AS` c0 `WHERE` id = 1 `# Starting point`<br/>
    &nbsp;&nbsp;`UNION ALL`<br/>
    &nbsp;&nbsp;`SELECT` c1.* `FROM` categories `AS` c1 `JOIN` cte `ON` c1.parent_category_id = cte.id<br/>
  )<br/>
  `SELECT` *<br/>
  `FROM` cte


<a name="modify"></a>
# 2. Data Modification Queries

### **INSERT INTO**: used to insert new records/rows in a table
* `INSERT INTO` table_name (column1, column2) `VALUES` (value1, value2);
* `INSERT INTO` table_name `VALUES` (value1, value2 …);

### **UPDATE**: used to modify the existing records in a table
* `UPDATE` table_name `SET` column1 = value1, column2 = value2 `WHERE` condition;
* `UPDATE` table_name `SET` column_name = value;

### **DELETE**: used to delete existing records/rows in a table
* `DELETE FROM` table_name `WHERE` condition;
* `DELETE` * `FROM` table_name;

<a name="aggregatefunctions"></a>
# 5. Aggregate Funtions
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
* ❗ just one aggreate function at a time, in the SELECT clause
* ❗ aggregate functions performed after WHERE clause

### **COUNT**: returns the # of occurrences
* `SELECT COUNT (DISTINCT` column_name`)`;

### **MIN() and MAX()**: returns the smallest/largest value of the selected column
* `SELECT MIN (`column_names`) FROM` table_name `WHERE` condition;
* `SELECT MAX (`column_names`) FROM` table_name `WHERE` condition;

### **AVG()**: returns the average value of a numeric column
* `SELECT AVG (`column_name`) FROM` table_name `WHERE` condition;

### **SUM()**: returns the total sum of a numeric column
* `SELECT SUM (`column_name`) FROM` table_name `WHERE` condition;

<a name="joins"></a>
# 4. Join Queries
### **JOIN**: general syntax
```sql
SELECT [DISTINCT] target_list [AS alias_column]
FROM table1_name [AS alias_table] [type_of_join] JOIN table2_name ON join_condition [ [type_of_join] JOIN table3_name ON join_condition ...]
[ WHERE tuple_of_tableX_name_condition ]
```
### types of join: ###

###  **INNER JOIN**: returns records that have matching value in both tables
* `SELECT` column_names `FROM` table1 `INNER JOIN` table2 `ON` table1.column_name=table2.column_name;
* `SELECT` table1.column_name1, table2.column_name2, table3.column_name3 `FROM` ((table1 `INNER JOIN` table2 `ON` relationship) `INNER JOIN` table3 `ON` relationship);

### **LEFT (OUTER) JOIN**: returns all records from the left table (table1), and the matched records from the right table (table2)
* `SELECT` column_names `FROM` table1 `LEFT JOIN` table2 `ON` table1.column_name=table2.column_name;

### **RIGHT (OUTER) JOIN**: returns all records from the right table (table2), and the matched records from the left table (table1)
* `SELECT` column_names `FROM` table1 `RIGHT JOIN` table2 `ON` table1.column_name=table2.column_name;

### **FULL (OUTER) JOIN**: returns all records when there is a match in either left or right table
* `SELECT` column_names `FROM` table1 ``FULL OUTER JOIN`` table2 `ON` table1.column_name=table2.column_name;

* ❗ with OUTER-JOIN, you should have to ***handle NULL values in WHERE clause***

### **Self JOIN**: a regular join, but the table is joined with itself
```sql
SELECT target_list
FROM Table_name T1 JOIN Table_name T2 ON T1.columnA = T2.columnA
[ WHERE condition_to_remove_duplicates ]
```
* ❗ JOIN-condition can be performed on any column (same table remember)
* ❗ in WHERE clause you can specify a particular condition to remove duplicates: same row but considered twice or same rows but in reverse ordere. To do this you can use the **<** operator between T1.columnB and T2.columnB for example
* ❗ if columnA is not a PK, and if columnB is not a PK, the condition_to_remove_duplicates could be contain the following: <br /> T1.PK < T2.PK AND T1.columnB <> T2.columnB.
<br/>**But**<br />
$PK \Rightarrow {other\;column\;values}$
<br/>so<br/>
(PK_1 == PK_2) --> (other_column_values_1 == other_column_values_2)
<br/>equals to<br/>
NOT (other_column_values_1 == other_column_values_2) --> NOT (PK_1 == PK_2)
<br/>so the condition can be simplified as follows:<br/>
~~T1.PK < T2.PK AND~~ **T1.columnB < T2.columnB**

<a name="view"></a>
# 5. View Queries

### **CREATE**: create a view
* `CREATE VIEW` view_name `AS SELECT` column1, column2 `FROM` table_name `WHERE` condition;

### **SELECT**: retrieve a view
* `SELECT` * `FROM` view_name;

### **DROP**: drop a view
* `DROP VIEW` view_name;

<a name="alter"></a>
# 6. Altering Table Queries

### **ADD**: add a column
* `ALTER TABLE` table_name `ADD` column_name column_definition;

### **MODIFY**: change data type of column
* `ALTER TABLE` table_name `MODIFY` column_name column_type;

### **DROP**: delete a column
* `ALTER TABLE` table_name `DROP COLUMN` column_name;

<a name="create"></a>
# 7. Creating Table Query

### **CREATE**: create a table
* `CREATE TABLE` table_name `(` <br />
   `column1` `datatype`, <br />
   `column2` `datatype`, <br />
   `column3` `datatype`, <br />
   `column4` `datatype`, <br />
   `);`
   
<a name="nestedqueries"></a>
# 9. Nested Queries or Subqueries

### **NESTED QUERY** (or subquery) is a query nested inside another query
* ⛩️ DIVIDE ET IMPERA approach.
* used to return a **single-value** (*single row and single column*), a **single-column** (*single column but multiple rows*) or a **multiple-columns** (*multiple columns and multiple rows*).
* **non**-correlated subquery is evaluated before the outer query, just one time. 
* **correlated** subquery is evaluated *once for each row of the outer query processed*, this because the subquery refers to a column that is not in the FROM clause, but is related to the outer query-table.
* used:
    - within `WHERE` clause to **filter** data/rows
    - within `HAVING` clause to **group** using, for example, an aggregate function
    - within `FROM` clause

#### syntax: **single-value** nested query in WHERE clause with *COMPARISON OPERATOR*
```sql
SELECT target_list_1
FROM Table_1 T1
WHERE columnX {=, <>, <, <=, >, >=} ( SELECT columnX
                                      FROM Table_2 T2
                                      WHERE condition_on_T2_PK )
```
* ❗ single-value subquery so:
  - one column, columnX, same domain
  - one value, condition_on_T2_**PK** in subquery ***selects just one row using the Primary Key***

#### syntax: **single-column** nested query in WHERE clause with *COMPARISON OPERATOR + **ANY/ALL***
```sql
SELECT target_list_1
FROM Table_1 T1
WHERE columnX {=, <>, <, <=, >, >=} ANY/ALL ( SELECT columnX
                                              FROM Table_2 T2
                                              WHERE condition_on_T2 )
```
* ❗ single-column subquery so:
  - one column, columnX, same domain
  - multiple rows, values.
* ❗ `ANY` return true if any of the subqueries values meet the condition with the comparison operator.
  - pattern to select non-max values for columnX with ANY and Table subquery
  ```sql
  ... WHERE columnX < ANY (SELECT columnX FROM Table_2)
  ```
  - alternative with aggregate function and Single-Value subquery
  ```sql
  ... WHERE columnX < (SELECT MAX(columnX) FROM Table_2)
  ```
* ❗ `ALL` return true if all of the subqueries values meet the condition with the comparison operator

   - pattern to select max value for columnX with ALL, **NULLABLE** column and Table subquery
  ```sql
  ... WHERE columnX >= ALL (SELECT columnX FROM Table_2 WHERE columnX IS NOT NULL)
  ```
    - alternative with aggregate function and Single-Value subquery, where columnX can be NULL
  ```sql
  ... WHERE columnX = (SELECT MAX(columnX) FROM Table_2)
  ```
#### syntax: **single-column** nested query in WHERE clause with **IN**
```sql
SELECT target_list_1
FROM Table_1 T1
WHERE columnX IN/NOT IN ( SELECT columnX 
                          FROM Table_2 T2
                          WHERE condition_on_T2 )
```
* ❗ the subquery must select for a column value (columnX) to produce a list of values for the outer columnX 
#### syntax: **multiple-columns** nested query in WHERE clause with **IN**
```sql
SELECT target_list_1
FROM Table_1 T1
WHERE (columnA, columnB) IN/NOT IN (  SELECT columnA, columnB
                                      FROM Table_2 T2
                                      WHERE condition_on_T2 )
```
* ❗ the subquery must select for multiple columns, can contain an aggregate function combined to a column to produce multiple rows for example
  ```sql
  ... WHERE (id, Qty) IN (SELECT id, MIN(Qty) FROM Table_2 GROUP BY id)
  ```
#### syntax: **single-value** nested query in SELECT clause
* almost a correlated subquery
* used to display in the result-set a data related to a specific row of the outer query
```sql
SELECT target_list_1, (
                        SELECT columnX /expression
                        FROM Table_2 T2
                        WHERE T1.columnA = T2.columnB ... AND condition_on_T2_PK
                      )
FROM Table_1 T1
[ WHERE condition_on_T1 ]
```
* ❗ the subquery must select for a single value, so for example columnX can be used in combination to condition_on_T2_PK or the expression can be an aggregate function for example
  ```sql
  SELECT target_list_1, (
                          SELECT columnX
                          FROM Table_2 T2
                          WHERE T1.columnA = T2.columnB AND T2.PK = <value>
                        )...
  ```
  ```sql
  SELECT target_list_1, (
                          SELECT MIN(Qty)
                          FROM Table_2 T2
                          WHERE T1.columnA = T2.columnB
                        )...
  ```
#### syntax: **multiple-columns** nested query in FROM clause
* never a correlated subquery
```sql
SELECT target_list_1
FROM Table_1 T1 [OUTER] JOIN ( SELECT columnA [, AGGREGATE_FUNC(columnB)]
                               FROM Table_2 T2
                               WHERE condition_on_T2 
                               [GROUP BY columnA]
                               )
                        ON T1.columnA = T2.columnA
[ WHERE condition_on_T1_T2 ]
```
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
