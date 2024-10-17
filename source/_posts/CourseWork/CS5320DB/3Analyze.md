---
title: 3. DML Contd. (SQL Query)
categories:
  - Course-work
  - CS-5320-Database
---

# Analyze SQL Queries

describes a **new relation** to generate

- `SELECT`: describes **columns** of relation to generate
- `FROM`: describes **source** relations and how to match
- `WHERE`: defines **conditions** result rows must satisfy
- `select * from courses where cid = 1;`

## Predicates

- inequalities: `>, >=`
- not equal: `<>`
- if in list: `Cname IN ('CS1111', 'CS2222')`
- Regex: `Cname LIKE 'CS**320&'`
  - `%`: zero or more arbitrary characters
  - `**`: one arbitrary character
- `AND`, `OR`, `NOT`: `Cname = 'CS4320' OR Cname = 'CS5320'`

## Clauses

### Select multiple columns

- `*` for all
  - `<table>.*` select all columns from table

### Arithmetic

- Must related to numbers

- `SELECT 3 * (<column1> + <column2>)`

### Assign names

- `SELECT Sname as StudentName`

### Join

- combine rows from tables based on a related columns
- `<table1> JOIN <table2> USING (<column>)`
  - `<table1> JOIN <table2> ON (<table1>.<column> = <table2>.<column>)`
  - Equivalent: `FROM <table1>, <table2> WHERE <join-condition>`
- `NATURAL JOIN`
  - `<table1> NATURAL JOIN <table2>`
  - Equalities btw ALL columns with same name
- `OUTER JOIN` (不用关心有 null 的数据)
  - Eliminates rows in other table, keep regardless
  - Fills up fields in missing row with **NULL** values
  - **keep left table**: `<table-1> LEFT OUTER JOIN <table-2> ON ..`
  - **keep right table**: `<table-1> RIGHT OUTER JOIN <table-2> ON ..`
  - **keep both tables**: `<table-1> FULL OUTER JOIN <table-2> ON ..`

```SQL
  SELECT <column1, column2>
  	FROM <table1>
  	JOIN <table2> ON (<join-conditions>)
  		...
  WHERE <additional-conditions>
  ---
  SELECT S.Sname
  FROM Students S
  	JOIN Enrollment E ON (S.sid = E.sid)
  	JOIN Courses C ON (E.cid = C.cid)
  WHERE C.Cname = 'CS4320'
```

### Distinct

- generate the unique rows
  - `SELECT DISTINCT`

### Aggregation over ROWs

- `COUNT, SUM, AVG, MIN, MAX`

  - `SUM, AVG, MIN, MAX`: numerical expression parameter
  - `COUNT(*)` for counting rows in result relation
  - `COUNT(<column>)` counts rows with value in column
  - `COUNT(DISTINCT <column>)` counts number of distinct values in column in result relation

  ```sql
  SELECT Count(*)
  FROM Students
  	JOIN Enrollment ON (Students.sid = Enrollment.sid)
  	JOIN Courses ON (Enrollment.cid = Courses.cid)
  WHERE Courses.Cname = 'CS4320'
  ```

### Aggregation over GROUPS

- `GROUP-BY` for multiple data subsets
  - `GROUP BY <column-list>` - distinguish data subsets based on their values in specified columns
- Result contains **one row per group**
  - Implies **restrictions** on SELECT clause!
  - Only expressions with **unique** value per group
  - This includes **aggregates** and **grouping** columns
- Put **WHERE** before grouping
- Put **HAVING** after grouping
  ```SQL
  SELECT Count(_), Cname
  FROM Students
  JOIN Enrollment ON (Students.sid = Enrollment.sid)
  JOIN Courses ON (Enrollment.cid = Courses.cid)
  WHERE Cname IN ('CS4320', 'CS5320') -- Filter out rows (before grouping)
  GROUP BY Cname
  HAVING Count(_) >= 100 -- Filter out groups (after grouping)
  ```

### ORDER BY

- `ORDER BY <Column2 ASC, Column2 DESC>`
- Prioritize the earlier items
- Applied **after** grouping (for group-by queries)
  • Items must have **unique** value per group

### LIMIT

- `Limit <Number>`: first numbers of rows

### NULL

- Unknown values, can be used as a value for any data type
- **If you do anything (> = <) with NULL (except IS), you’ll just get NULL**
- `NULL` is falsey
  - `WHERE NULL` === `WHERE FALSE`
- Ternary
  - `TRUE`
    - If evaluate to `TRUE`
  - `FALSE`
    - If evaluate to `FALSE` regardless
  - `NULL`
    - if it depends on the `NULL` value.
  - Result can be TRUE > UNKNOWN > FALSE
- If **NULL** in **comparison operation (=)**, no rows will be returned.
  - `TRUE` dominates in `OR` operations (anything OR `TRUE` is `TRUE`).
  - `FALSE` dominates in `AND` operations (anything AND `FALSE` is `FALSE`).
  - `NULL` (Unknown) propagates when the result can't be definitively determined by the other operand.
- Example
  - `<expression> = TRUE`
  - `<expression> = FALSE`
  - `<expression> IS NULL` (not `= NULL`).
  - `SELECT 3 = NULL;` - UNKNOWN
  - `SELECT NULL = NULL;` - UNKNOWN
  - `SELECT NULL IS NULL;` - TRUE
  - `SELECT NULL IS NOT NULL;` - FALSE
  - `SELECT TRUE OR NULL;` - TRUE
  - `SELECT TRUE AND NULL;` - UNKNOWN

### SET (2 queries must be union-compatible)

- UNION
  - get tuples about duplicates
  - `<query-1> UNION <query-2>` : eliminates duplicates
  - `<query-1> UNION ALL <query-2>` : keep duplicates
- Intersect
  - result from 2 queries
  - `<query-1> INTERSECT <query-2>`
- EXCEPT
  - diff btw queries
  - `<query-1> EXCEPT <query-2>`

### Query Nesting

把 query 塞到 FROM, WHERE, SELECT ... clause

- Sub-Query $\in$ Query
- Uncorrelated - can be executed independently
  - `SELECT name FROM employees WHERE department = (SELECT department FROM departments WHERE manager = 'John');`
- Correlated - depends on outer query
  - `SELECT name FROM employees e WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);`
- UnCorrelated Sub-Query IN Conditions
  - EXISTS
    - `EXISTS(<sub-query>)` : TRUE if non-empty
  - IN
    - `<value> IN (<sub-query>)` : TRUE if contained
  - for all OR some: $\forall$ OR $\exists$
    - `<value> >= ALL(<sub-query>)`: TRUE if satisfied for all
    - `<value> >= ANY(<sub-query>)`: TRUE if satisfied for some
