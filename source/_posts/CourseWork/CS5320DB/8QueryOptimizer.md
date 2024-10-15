---
title: 8. Query Optimizer
categories:
  - Course-work
  - CS-5320-Database
---

- SQL Query => {Plan Enumeration, Query Optimizer, Cost Prediction} => Query Plan

## Cost Prediction

### 1. Data Statistics in Postgres (PG)

- Number of distinct values in column
- Ratio of SQL null values in column
- Most frequent values with associated frequency
- Histograms approximating column data distribution

##### How to use statistics

- Force DBMS to create statistics via **VACUUM ANALYZE**
  - collects and updates distribution statistics of data in tables
- Generated statistics are exploited by query optimizer
  - Statistics causes good/poor query performance
- Can access column statistics via `pg**stats` view
  - E.g., `tablename`, `attname`, `n**distinct`, `null**frac`, ...
  - Find what query optimizer relies on and check for updates

### 2. Selectivity Model (Estimating Selectivity - pass % of rows)

- Selectivity: estimate how much data will be filtered out (probability the rows satisfy predicate)
  - More selective - filters more rows
  - A selectivity of **1**: all rows pass the condition (the predicate is true for all rows).
  - A selectivity of **0.1** : 10% of the rows pass the condition, filtering out 90% of the data.
- Estimate by constraints (uniqueness, primary keys) and data statistics
  - E.g., `Selectivity(Column=Value) = 1/NrValues`
    - If you have a column with `NrValues` distinct values, the selectivity of a query where `Column = Value` is estimated as `1/NrValues`.
    - **Assumes Uniform Data** - each value is equally likely
  - E.g., `NrRows(Column=Value) ≤ 1 if Key Column`
    - If the column is a key column (i.e., it has a unique constraint like a primary key), the selectivity of a query `Column = Value` is 1 or less, since a key column can have at most one matching row.
  - Selectivity(A⋀B) = Selectivity(A) \* Selectivity(B)
    - **Assumes Independent Predicates** (not affect each other)

### 3. Cardinality Model (produce how many rows)

- Cardinality: the number of rows produced by a query
- Helps to predict the cost of different query execution strategies

##### Estimate Cardinality

- **cardinality of single tables**
  - A table with 100 rows - cardinality of 100
- **cardinality product** for tables in FROM clause (more than one table)
  - Result rows number if predicates = true
- Now multiply with selectivity estimates for all predicates
  - Cardinality = Cardinality table A ** Cardinality table B ** ...
  - Estimate how many rows pass predicate filter

##### Estimate Page Size

Find page size for cost functions

1. Find Average byte size per Record
2. Find records number per page
   - Data in pages (fixed-size blocks), a page has records.
   - Pages cannot store fractional records → round down
3. Find total number of Pages:
   - row number / record num per page

### 4. Cost Model (estimate the cost: I/O + operators)

- Count & sum cost of page reads and writes by operators in the plan
- Each operator
  - input: how often is it read from disk?
  - output: is it written to disk? Is it final result?
  - Does the operator read/write intermediate results?

# Plan Enumeration

- **Generate** every possible plan
- **Estimate** cost for each plan
- **Select** plan with minimal cost

## Query Plan Space

- Decide **order** of operations and **implementation**
- Apply heuristic restrictions

### H1: Early Predicates and Projections

- Processing more data => more expensive
- Reduce data size ASAP
- option: Add predicates (discarding rows)
- option: Add projections (discard columns)

### H2: Avoid Joins without Predicates

- Join result size: product of input cardinality \* selectivity
- Selectivity = **1** if joining tables without predicates
- Sub-optimal for very large join results
- (Heuristic may discard optimal order in special cases)

### H3: Left Deep Plans

- Allows pipelining: joins pass on result parts in-memory
- Allows to use indices on join columns (of base tables)

## Asymptotic Analysis

- **num of join orders** = num of possible ways (orders) to join the tables = `O(nrTables!)`.
  - For 3 tables: 3! = 6
  - For 5 tables: 5!=120
- Number of join orders <= total number of query execution plans.
  - Join order many implementations (e.g., nested loops, hash joins)
- Enumerating plans is considered **impractical** from the factorial growth
  - 5 tables has 120 join orders and MANY plans

## Principle of Optimality

- Definition
  - Find the most efficient & cheapest plan to join tables
  - an optimal plan for the full join must include optimal plans for joining smaller subsets of the tables along the way.
  - If any subset is joined sub-optimally, the overall plan can be improved by replacing it with the optimal one.
- This plan joins table subsets "on the way"
  1.  looks for the optimal way to join smaller **subsets of tables**.
  2.  Once found the best way to join the subsets, it combines all results to build the full join plan.
- Assume we use sub-optimal plan for joining table subset
- Replacing by a better plan can only improve overall cost

## Efficient Optimization

- Find optimal plans for (smaller) sub-queries first
  - Sub-query: joins subsets of tables
  - if joins 5 tables, first find the best way to join subsets of 2 or 3 tables.
- Compose optimal plans from optimal sub-query plans

