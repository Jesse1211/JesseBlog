---
title: Cornell CS ⅘320 Database
categories:
  - Course
  - CS-5320-Database
---

`psql -d postgres`

# Intro File-Based Approach

### DBMS - Database Management System

**Manage data, allows applications to store, update and analyze the data** and **avoids Code, Data Overlap, Inconsistencies**

- Layer 1: Query Processor (Operators, Cost Estimation, Query Optimization)
  - Parser - **Check syntax and convert to Optimizer readable command, and check semantic**
  - Optimizer - **优化器会生成许多能够满足查询请求的访问计划, 并选择具有最小成本的方案作为最终的访问计划（Acess Plan), 通过 Estimator, Plan Generator)**
  - Rewriter - **Convert to higher performance command**
  - Executor - **执行**
- Layer 2: Storage Manager (Storage Media)
  - Data Access - **Handles access to the data stored in various storage media.**
  - Buffer Manager - **Manages the memory buffers where data is temporarily stored during processing.**
  - Indexes, Filed, Data Layout - **Organizes data and indexes in storage for efficient retrieval.**
- Layer 3: Transaction Processing
  - Concurrency Control - **Manages concurrent access to the database to ensure consistency.**
  - Crash Recovery - **Restores the system to a consistent state in the event of a crash.**
  - Transaction Manager - **Ensures that all database transactions are processed correctly and ensures ACID properties.**
  - Recovery Manager
- Layer 4: Data
  - Schema Design - **Designs the database schema and ensures efficient organization of data.**
  - Detecting Redundancy - **Identifies and eliminates redundancy in the database schema.**
  - Schema Normalization - **Normalizes the database schema to avoid anomalies.**
  - Distributed Process (NoSQL and NewSQL) - **Handles data distribution across multiple nodes, especially in distributed and NoSQL databases**
  - Graph Data, Data Streams, Spatial Data - **Manages specialized data types like graph data, streaming data, and spatial data.**

---

# SQL Queries

## SQL (schema, CRUD data)

- Database
  - Organized collection of inter-related data used to model aspects of real-world
  - Set of Relations (tables)
- Relation
  - **Schema**: description (column header, type)
  - \***Instance**:\* set of data satisfying the schema
- Attribute
  - column or field
- Tuple
  - row or record `[5, Bob, NULL]`

### Postgres

- `createdb <dbname>` Creates a database
- `dropdb <dbname>` Deletes a database
- `\l` Lists all databases
- `\d` Lists all relations within the database
- `\d <relation name>` Lists the schema information of the relation

### SQL Definition (Structured Query Language)

- Issue commands to the DBMS

#### 4 types of commands (focus on first 3)

- DDL: 定义 - **Data Definition Language**
  - Define admissible database content (schema)
  - `DROP, CREATE, ALTER`
- DML: 操作数据 - **Data Manipulation Language**
  - analyze, combine, change and retrieve database content
  - `INSERT, UPDATE, DELETE`
- TCL: each update query is a transaction - **Transaction Control Language**
  - Groups SQL commands (transactions)
  - `SELECT语句`
- (不需要懂) DCL: 控制, 分配, access permissions - **Data Control Language**
  - Assign data access rights
  - `GRANT, REVOKE, COMMIT, ROLLBACK`

### DDL: 定义 Data Definition Language (Schema, Relations, Constraints)

Not using Single Table in database schema design

- Reduce redundancy
  - A well-designed schema breaks the data into **multiple related tables** to avoid repeating the same information across rows.
- Minimize (inconsistent or incorrect) update errors
  - By splitting data into related tables, each piece of information exists in only one place, making updates more reliable and reducing the chance of errors.

#### Schema

Define relations with their **schemata**

- columns and column **types** - `CREATE`
- `CREATE TABLE <table> (<table-def>)`
  - `<table>` is the table name
  - `<table-def>` is comma-separated column definitions
  - Column definition: `<col-name> <col-type>`
  - `CREATE TABLE Students(Sid int, Sname text, Gpa real);`

#### Constraints

- Define **constraints** restricting admissible content
  - Constraints on **single relations**
  - Constraints linking **multiple relations**
- Primary key
  - 每个 table, 只能有一个并且不能重复, 不能改变/删减
  - 如果是 Composite, 那就是组合不能重复
  - identifies a subset of columns as key columns
  - Fixing values for key columns must identify row
  - `ALTER TABLE <table name> ADD Primary Key (col1, col2);`
- Foreign key
  - 两个 table 之间的 match
  - Foreign key (A) references Z(B) -> A 的值完全来自于 Z.B
  - `ALTER TABLE <table-1> ADD Foreign Key (<fkey-columns>) REFERENCES <table-2> (<pkey-columns>);
- Range: `ALTER TABLE students ADD CHECK(gpa>=0 and gpa<=4.0);`
- Unique: `ALTER TABLE courses ADD CONSTRAINT c1 UNIQUE(cid);`
- Integrity
  - the logical and physical data stored in SQL Server are structurally sound and consistent
  - limit the admissible content of tables
  - enforced by DBMS
  - `ALTER TABLE` to add integrity constraints

### DML: 操作数据 Data Manipulation Language (Insert, Delete, Update, Analyze)

- Insert
  - Inserting one (**fully specified**) row into a table:
    - `INSERT INTO <table> VALUES (<value-list>)`
    - `INSERT INTO Students VALUES (3, 'Alice', 4.0)`
  - Inserting one (partially specified) row into a table:
    - `INSERT INTO <table> (<column-list>) VALUES (<value-list>)`
    - `INSERT INTO Students (Sid, Sname) VALUES (5, 'Bob')`
  - Loading data from a file into a table:
    - `COPY <table> FROM <path> DELIMITER <delimiter> NULL <null-string> CSV`
    - `COPY Courses FROM 'courses.csv' DELIMITER ',' CSV`
- Delete
  - Deleting rows from a table that satisfy condition:
    - `DELETE FROM <table> WHERE <boolean-condition>`
    - `DELETE FROM Courses WHERE Cname = 'CS6320'`
- Update
  - Updating specific rows and columns to new value:
    - `UPDATE <table> SET <column> = <value> WHERE <condition>`
    - `UPDATE Courses SET Cid = 7 WHERE Cname = 'CS4320'`
- Analyze
  - 看下一章

---

## SQL (analyze: SQL Queries)

describes a **new relation** to generate

- `SELECT`: describes **columns** of relation to generate
- `FROM`: describes **source** relations and how to match
- `WHERE`: defines **conditions** result rows must satisfy
- `select * from courses where cid = 1;`

#### Predicates

- inequalities: `>, >=`
- not equal: `<>`
- if in list: `Cname IN ('CS1111', 'CS2222')`
- Regex: `Cname LIKE 'CS**320&'`
  - `%`: zero or more arbitrary characters
  - `**`: one arbitrary character
- `AND`, `OR`, `NOT`: `Cname = 'CS4320' OR Cname = 'CS5320'`

#### Clauses

##### Select multiple columns

- `*` for all
  - `<table>.*` select all columns from table

##### Arithmetic

- Must related to numbers
- `SELECT 3 * (<column1> + <column2>)`

##### Assign names

- `SELECT Sname as StudentName`

##### Join

- combine rows from tables based on a related columns
- `<table1> JOIN <table2> USING (<column>)`
  - `<table1> JOIN <table2> ON (<table1>.<column> = <table2>.<column>)`
  - Equivalent: `FROM <table1>, <table2> WHERE <join-condition>`
- `NATURAL JOIN`
  - `<table1> NATURAL JOIN <table2>`
  - 保留合适的结果
- `OUTER JOIN`
  - Null 不会 Join Null, 但是会 join non-Null
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

##### Distinct

- generate the unique rows
  - `SELECT DISTINCT`

##### Aggregation over ROWs

- `COUNT, SUM, AVG, MIN, MAX`
  - `SUM, AVG, MIN, MAX`: numerical expression parameter
  - `COUNT(*)` for counting rows in result relation
  - `COUNT(<column>)` counts rows with value in column
  - `COUNT(DISTINCT <column>)` counts number of distinct values in column in result relation
  ```SQL
  SELECT Count(*)
  FROM Students
  	JOIN Enrollment ON (Students.sid = Enrollment.sid)
  	JOIN Courses ON (Enrollment.cid = Courses.cid)
  WHERE Courses.Cname = 'CS4320'
  ```

##### Aggregation over GROUPS

- `GROUP-BY` for multiple data subsets
  - `GROUP BY <column-list>` - distinguish data subsets based on their values in specified columns
- Result contains **one row per group**
  - Implies **restrictions** on SELECT clause!
  - Only expressions with **unique** value per group
  - This includes **aggregates** and **grouping** columns
  - 按照字段分组, 每组代表一个 row
  - 尽量搭配 Aggregation
  - 如果是 Select \* , 那只会显示每组第一行
- Put **WHERE** before grouping
- Put **HAVING** after grouping
  ```SQL
  SELECT Count(*), Cname
  FROM Students
  	JOIN Enrollment ON (Students.sid = Enrollment.sid)
  	JOIN Courses ON (Enrollment.cid = Courses.cid)
  WHERE Cname IN ('CS4320', 'CS5320') -- Filter out rows (before grouping)
  GROUP BY Cname
  HAVING Count(*) >= 100 -- Filter out groups (after grouping)
  ```

##### ORDER BY

- `ORDER BY <Column2 ASC, Column2 DESC>`
- Prioritize the earlier items
- Applied **after** grouping (for group-by queries)
  • Items must have **unique** value per group

##### LIMIT

- `Limit <Number>`: first numbers of rows
- `Limit <i> Offset <j>`: 从 j+1 开始显示之后的 i 个

##### NULL

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
  - True OR NULL = True
  - FALSE OR NULL = NULL
  - NULL AND NULL = NULL
  - `<expression> = TRUE`
  - `<expression> = FALSE`
  - `<expression> IS NULL` (not `= NULL`).
  - `SELECT 3 = NULL;` - UNKNOWN
  - `SELECT NULL = NULL;` - UNKNOWN
  - `SELECT NULL IS NULL;` - TRUE
  - `SELECT NULL IS NOT NULL;` - FALSE
  - `SELECT TRUE OR NULL;` - TRUE
  - `SELECT TRUE AND NULL;` - UNKNOWN

##### SET (2 queries must be union-compatible)

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

##### Query Nesting:

把 query 塞到 FROM, WHERE, SELECT ... clause

- Sub-Query $\in$ Query
  - `select * from A where salary < (select avg(salary) from employees);`
  - `select a, b, (select avg(c) from A) from A;`
  - `select * from (select a, b, c, d from A)`
- Uncorrelated - can be executed independently
  - `SELECT name FROM employees WHERE department = (SELECT department FROM departments WHERE manager = 'John');`
- Correlated - depends on outer query
  - Sub 里面的数据是从外面拿到的
  - `SELECT name FROM employees e WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = e.department);`
- UnCorrelated Sub-Query IN Conditions
  - EXISTS
    - `EXISTS(<sub-query>)` : TRUE if non-empty
  - IN
    - `<value> IN (<sub-query>)` : TRUE if contained
  - for all OR some: $\forall$ OR $\exists$
    - `<value> >= ALL(<sub-query>)`: TRUE if satisfied for all
    - `<value> >= ANY(<sub-query>)`: TRUE if satisfied for some

# 5. Data Storage

- Decomposing **tables** into pages, **pages** into slots, **slots** into fields
- Variable length content can be handled via **directories**

### hardware - storage

#### Relevance for DBMS

- **Capacity limits** force data to lower parts of hierarchy
- Data **access speed** may become bottleneck
- Design algorithms to **minimize data movements**
- **Random** data access is expensive
- Read data in larger **chunks ("pages")**
- Keep related data **close** together
- Take into account volatility for **recovery** considerations

#### Memory Hierarchy:

$$(\text{Volatile}): \text{Registers} \to \text{CPU Cache} \to \text{Main Memory} \to (\text{Non-Volatile}): \text{Flash/USB Memory} \to \text{Hard Disk} \to \text{Tape Backup}$$
$$\text{Faster Access}\to$$
$$\text{Lower Capacity} \to$$

#### Tape Storage

- Bits as **magnetic** information on tape
- **Very slow access** (10s of seconds)
- **Moderate read speed** (up to 300 MB/second)
- **Very cheap** (around $0.02 per Gigabyte)
- Used for long-term **archival** (e.g., by Google)

#### Hard Disk

Bits as **magnetic** information on platter

- Patters **spin** under read/write heads
- **Slow access** (10s of milliseconds access time)
- **Moderate read speed** (around 200 MB/second)
- **Cheap** (around $0.035 per Gigabyte)
- Used for **less frequently accessed** data

#### Solid State Drives SSD

- Bits as small **electric charges**
- **Elevated price** (around $0.25 per Gigabyte)
- **Fast access** (around 1 millisecond)
- **Elevated speed** (around 500 MB/second)
- Limited number of write cycles (**memory wear**)

#### Main Memory

- Bits as small **electric** charges
- **Expensive** (several dollars per Gigabyte)
- **Very fast access** (order of nanoseconds)
- **High bandwidth** (Gigabytes per second)
- Used to access **hot** data - **all** if economically feasible!

#### Cache

- Bits as small **electric** charges
- Typically organized as cache **hierarchy**
- **Very expensive** (hundreds of dollars per Gigabyte)
- **Near-instantaneous** access (few nanoseconds)
- **Very high bandwidth** (tens of Gigabytes per second)
- Used to store **immediately relevant** data

### format - relation representation

#### Tables as Files

- Table schema === database catalog; 每个 page 的一个 row
- Table content $\in$ **collection of pages** called file
- page
  - a few KB for each page
  - might not enough for entire table (only multiple rows)

#### Files to Pages

1. Store pages as **doubly linked list**
   - ![[Screenshot 2024-09-05 at 6.03.39 PM.png|400]]
   - each data page contains **records**, a **free space tracker**, and **pointers** (byte offsets) to the next and previous page.
   - header page - header reference in DB catalog
     - start of the file and separates the data pages into full pages and free pages.
     - When space is needed, empty pages are allocated and appended to the free pages portion of the list.
     - When free data pages become full, move from the free space portion to the front of the full pages portion of the linked list.
2. **Directory** with pointers to pages
   - ![[Screenshot 2024-09-05 at 6.03.55 PM.png|400]]
   - Directory pages reference **data pages with meta-data**
   - Faster records insertion
     - Read at most all of the directory pages

#### Pages divided to Slots

- One **record** per slot (i.e., table row)
  - **(pageID, slotID)**
- Multiple ways to **divide** pages into slots

##### 1 Fixed-length Records (FLR)

- Number of bytes per slot is determined
- **Packed** representation uses **consecutive slots**
  - Sorted in USED & UNUSED
  - Only keep track of **number** of slots used
- **Unpacked** representation allows unused slots in-between
  - Unsorted in USED & UNUSED
  - Need **bitmap** to keep track of used slots

##### 2 Variable-length records (VLR)

- E.g., records with variable-length **text** fields
- Number of bytes per slot is **not fixed**
- Each Page has **directory** about used slots: first byte, slot length
- **Flexibility** to move around records on page
- Can use that for regular **compaction**

#### Slots divide to Fields

- **Fixed** length field: field sizes defined in DB **catalog**
- **variable** length field: store field sizes on **page**
  - Option 1: use special **delimiter symbol** between fields
  - Option 2: store "**field directory**" at beginning of record

## Questions

1. Given a heap file implemented as a Page Directory, what is the I/O cost to insert a record in the worst case? The directory contains 4 header pages and 3 data pages for each header page. Assume that at least one data page has enough space to fit the record.
   - 4 (read header pages) + 1 (read data) + 1 (write data) + 1 (write last header) = 7
2. What is the smallest size, in bytes, of a record from the following schema? Assume that the record header is 5 bytes. (boolean = 1 byte, date = 8 bytes)
   ```SQL
   name VARCHAR
   student BOOLEAN
   birthday DATE
   state VARCHAR
   ```
   - 5 (record header) + 1 (boolean) + 8 (date) + 0 (state) + 0 (name) = 14
3. What is the maximum size, in bytes, of a record from the following schema? Assume that the record header is 5 bytes. (boolean = 1 byte, date = 8 bytes)
   ```SQL
   name VARCHAR(12)
   student BOOLEAN
   birthday DATE
   state VARCHAR(2)
   ```
   - 5 (record header) + 12 (VARCHAR) + 1 (boolean) + 8 (date) + 2 (VARCHAR) = 28

# 6 - 7 Tree & Hash Indexes

#### Index

- Index == **references** to data records
- 一个 table 可以做很多, 而且比 binary 快, 并且支持不同的 columns
- 如果 index 过多, 那就变慢了
- 一次 Query 用多个 Index
  - Index merge
- **index search key**: (the index value we are searching for)
  - Optimization: store search key value to **reference list**
    - one index value has A collection of pointers or references to the actual data
  - Advantage: avoids storing key values **redundantly**
    - Might store same key in difference rows
  - Disadvantage: creates **variable length** field (list)
    - length depends on # rows share same search key value
- Example from step by step:
  - Checks the index for the search key value `'Smith'`.
  - Retrieves the reference list associated with `'Smith'`, which contains pointers to the rows where `'Smith'` appears (e.g., rows 1, 2, and 5).
  - Fetches the actual data rows from the table using pointers

## 6. Tree Index

- B+ tree indexes
  - Average **fanout** (i.e., number of child nodes) is 133
  - traverse to find interesting leaves
  - Handle **equality and inequality**
    - Indexes in **sort order**
    - Consecutive keys stored close together
  - Composite keys: useful for conditions on key **prefix**
    - Keys with same prefix value stored close together
  - 可以做 Equality, Inequality, Prefix, 但是不能找中间值 (比如 LIKE '%a%')
  - 内部 Balance 机制用于保证 Time (Path length) & Space (underfull) complexity
- Cost
  - 找一个 / 多个 通过和完整 tuple 数量的比例计算需要读多少 Leaf Node - N / pageSize
  - Clustered index
    1. 需要经过 inner node 数量 +
    2. 需要查询 leaf node 数量
  - UnClustered Index
    1. 需要经过 inner node 数量 +
    2. 需要查询 leaf node 数量 +
    3. 查询 Tuple 的数量

#### Tree indexes

- ![[Screenshot 2024-09-12 at 4.14.34 PM.png|400]]
- R - Record Pointer (Reference)
  - Points to P for more entries
- P - Page as a Node
  - contains K and R
- K - Key
  - Indexed value, **ordered**
  - K(i): search key values. help define the boundaries or the ranges of data that each reference (R(i)) covers.
  - Holly, Olivia: R points to different P

##### Inner Nodes - R(0), K(1), R(1), K(2),...

- Any entries R(i) points to will be smaller than K(i+1) but larger than K(i).

##### Leaf Nodes: K(1), R(1), K(2),...

- Contains actual data references
- K(i): the key values related to the data records.
- R(i): point to **{P: data pages, specific slots}** (retrieving to the physical location of the data)

##### Example Usage (works if predicate references index key)

- equality predicates: `WHERE Sname = 'Alan'`
  1.  Start at root node: Searching for entries with key value V
  2.  Until reaching a leaf node:
      - Search for i such that **V ≥ K(i), V < K(i+1)**
      - Follow associated **reference R(i)**
  3.  At leaf node:
      - Search for i such that **K(i) = V**
      - Retrieve data from R(i) if found, otherwise return empty
- **inequality** predicates: `WHERE gpa > 3`
  - Searching for index entries with **key value from [L,U]**
  - Use equality search procedure to **find entry with value L**
  - Follow links between leaf nodes until **reaching value U**
  - Retrieve referenced data on the way

##### Linking leaf Nodes (B+)

- Leaf pages is **doubly linked list**
  - pointer to next/previous neighbor in leaf

#### Composite keys

- **multiple columns** $\in$ Index search key
- **sort order** for key comparisons
  - `(a, b, c)`: a first, then b, then c
- Can use index for (in)equalities on **prefix** of key columns
  - Restriction to Key Prefix: **Cannot skip first column(s)**

#### Postgres SQL Query

- `CREATE INDEX <index-name> on <table> (<columns>)`
  - Creates index for table using specified search key
  - Refer to index later via `<index-name>`
  - `<columns>` is comma-separated column list (key)
- `DROP INDEX <index-name>`
  - Delete index with given name

#### Clustered index

- **index stores data** instead of references to data
  - Actual table rows are organized (physically sorted) by index in physical storage.
- **at most one** clustered index per table
- **collocates** data with same key together (such as OrderData)
  - faster data retrieval for **Range, Ordered queries**
  - data pages are sorted on the same index on building B+ tree.
  - Two records with close keys will likely be in the same page.

#### UnClustered Index

- read a separate page for each of the records
- if we want to read adjacent records
  - read each of data pages they point to

#### Tree Variants B+: handling balance when update

- **Shallow**
  - order: # of entries in each node
  - maximal is 2 $*$ order
  - under-full is less than order entries
- keep index balanced during updates
  - **Balances** tree after insert/delete operations
  - Keeps the tree **compact**
  - Each node (except root) is **at least half full**!
    - I.e., number entries between **d** and 2$*$d
  - After deletion, need to **fix nodes** that are under-full
- Fix for balancing
  - Now: have **redistributed** entries from sibling leafs
  - may **merge** tree nodes together
  - Merge operations may **propagate upwards** in tree
    - E.g., imagine student "Bella" is **deleted** again
    - Tree loses one level (**inverse** to insertion operation)

## 7. Hash Index

- Hash index: evaluate hash function to find buckets (key hash values)
  - buckets: a storage location which f(entries) are same
  - Handle **equality**
    - Consecutive keys might not close
    - Similar hash value $\neq$ similar key value
  - Composite Keys: Condition must constrain **all components** instead of prefix
    - Keys with **same prefix** may be stored far apart
  - 可以做 Equality, 但是 Inequality 和 like 不行
- `CREATE INDEX <index-name> ON <table> USING <method> (<column-list>)`

#### Static hashing (bucket pages)

- HashFunction 不变, 每个 Bucket 所在的 LinkedList 越来越长
- Bad for dynamic data
- Hash **bucket** pages contain references to data (OR data directly)
- Hash buckets are associated with **hash value ranges**
- Can use hash index to find entries with key V
  - Calculate hash value h for V as h(V)
  - **Look up bucket page associated with h**
- Cost: 1 + num of overflow pages

##### Update

- Deletions
  - remove associated entries
- Insertions
  - Can add "overflow" pages (like ISAM index!)
  - Initial bucket page stores pointer to first overflow page
  - Overflow pages form **linked list** if more than one
- **Rehash** if number of overflow pages increases
  - Create a new, larger hash table from the old table using different hash function.
- ![[Screenshot 2024-09-23 at 5.46.04 PM.png|400]]

##### Pros & Cons

- Get data with one read
- Will waste space if too many deletions (empty pages)
  - does not adjust number of buckets dynamically
- Can use rehashing but creates significant overheads
  - Create a new table
  - Recomputing
  - Moving

#### Extendible hashing (directory $\to$ buckets $\to$ pages)

- ![[Screenshot 2024-10-16 at 5.34.04 PM.png|400]]
- More flexible than using page IDs directly
  - **Directory**: store addresses of the buckets in pointers
  - **Buckets**: store hashed keys
- Expands with few high-overhead operations
- Redistribute overflowing buckets to multiple pages
  - **Bucket split & directory expansion**
  - Increase directory size if too many splits
  - Split Bucket to 2 parts

##### Insertions

1. Calculate hash value for key of new entry
2. Consult directory to identify current bucket
3. insert
   - If bucket has space: Good
   - If overflowing:
     - Add new bucket page, rehash existing and new entry
       - For rehashing: consider one more bit of hash value
       - Expand directory if it does not consider enough bits

##### Terminology

- Global depth: num of **hash bits in directory**
  - hash function to categorize the keys
  - Max # of bits needed to tell which bucket an entry belongs to.
- Local depth: **num hash bits in bucket**
  - The # bits used to determine if an entry belongs to this bucket.
  - to decide if overflow occurs (need to split)
    - Before insert, local depth = global depth.
    - Insert causes local depth to become > global depth; directory is doubled by copying it over and **fixing** pointer to split image page.
- 当 bucket 满了以后, trigger doubling Global Depth 扩大 Directory, 然后回过头扩大 bucket 的 local depth 并且 split.

##### Deletions

- Merge bucket pages if they become empty
- Half directory size if number of buckets shrinks
- Often no compaction in practice
  - Assumption: inserts are more common than deletes

##### Pros & Cons

- No expensive rehashing
  - Only rehash one bucket at a time
- Need additional directory access
- Need to double directory occasionally
  - This may take up some time

#### Linear hashing (Avoid overflow pages : buckets $\to$ pages)

- Expands more "smoothly"
- Idea: avoid using directory by fixing next bucket to split
  - Not always split overflowing bucket
    - I.e., have temporary overflow pages
  - Buckets to split are selected in round robin fashion
    - Overflowing bucket will be split eventually

##### Insertion

- Calculate hash value for new entry to insert
- Add entry on page / overflow page
- **Split next bucket if trigger condition is satisfied**
  - May eliminate previously generated overflow pages
  - Some flexibility in choice of trigger condition

##### Splitting

- Splitting proceeds in rounds: **round-robin order**
  - All buckets present at round start split → round ends
  - "Next Split" pointer is reset to first page at round end
  - Splitting regardless of whether the bucket is full, empty, or overflowing
- Split the bucket pointed to by "Next Split"
  - Add one new page, redistribute split bucket entries
    - Consider one more bit when redistributing

##### Pros & Cons

- Avoids a directory - no expensive directory doubling
- May temporarily admit overflow pages
- May split empty pages - inefficient space utilization

#### Optimizations

- Can apply same optimizations as for tree indexes
- Have many entries for same search key value?
  - Store key value, followed by list of references
- Want to get rid of one level of indirection?
  - Can store data directly instead of references
  - Leads to "clustered index", only one per table!

# 8 Processing Overview (Buffer Manager)

![[Screenshot 2024-09-23 at 8.00.40 PM.png|400]]

- Decides when to move data between disk and RAM
- Reduce data movements using heuristics
- Buffer manager manages "buffer pool"
  - Buffer pool: main memory reserved for DBMS
  - **frames**: page-sized slots
  - Stores meta-data about each slot

#### Frame Properties

To efficiently track frames, the buffer manager allocates additional space in memory for a metadata table.

- Page ID:
  - Page ID stored in this frame
- Pin count: # of processes using the page
  - Evict page if pin count reaches zero
- Dirty bit: memory data != storage data (updated data)
  - Write page to disk before evicting it

#### Don't rely on OS

- OS uses virtual memory caches pages
- **We want a separate buffer manager**
  - DBMS knows its access patterns ahead of time
    - Smarter replacements
  - DBMS must control page writes for safety guarantees

#### Processing Page Requests (buffer manager)

- Cache Hit (requested page cached)
  - Increase pin count & return page address
- Cache Miss (requested page not cached)
  - Choose frame for storage data (replacement policy)
  - If frame contains dirty page: write to disk
    - Increase pin count and return page address
  - Read requested page from disk and store in frame, Set pint count to 0

##### LRU

- Replace page required farthest in the future
  - Reduce expensive cache misses
- However: difficult to predict that in general
- Heuristic: remove least recently used page (LRU)
  - remove use page which did not use for long time

##### Sequential Flooding (when LRU is no good)

- buffer pool’s contents are corrupted due to a sequential scan.
- Since sequential scans read every page, the timestamps of pages read may not reflect which pages we actually want.
  - the most recently used page is actually the most unneeded page.
- DBMS often have particular access patterns
  - e.g. Scanning pages in round robin mode (circular or repeated sequence)
- Least recently used page is used again soonest
  - LRU policy gets sub-optimal (inefficient)

# Operators

## Page Nested Loop Join

`Outer + Outer * Inner.` 每个 tuple 都要 load 对面所有的信息

- for (Load ALL pages from **first table**):
  - for (Load ALL pages from **second table**):
    - For all tuples in memory: **check** and **add** to result
- Memory (1 + 1 + 1 = 3)
  - Store current **page from outer table**: 1
  - Store **current page from inner table**: 1
  - Need **one buffer page** to store output (before disk write): 1
    - For the intermediate result

## Block Nested Loop Join

`Outer + Outer / blockSize * Inner`

- Memory (B + 1 + 1 = B + 2)
  - Space to store **blocks from outer relation**: B
  - Space to store **one page from inner** relation: 1
  - Need one **page to store output** (before writing to disk): 1

## Index Nested Loop Join

- `Outer + Inner * (Match + Load 1 index)` outer tuple match inner 的 x 个 tuple per page
- Memory (1 + 1 + 1 = 3)
  - Store current **page from outer table**: 1
  - Store **current page from inner table**: 1
  - **output buffer** (before disk write): 1

## Equality Joins - Hash Join

`cost = 3 * (pageNum1 + pageNum2)` 不包含 Write

1. Phase 1 - cost = (Page table1 ** 2) + (Page table2 ** 2)
   - **Partition data** by hash values in join columns (partition by even and odd value)
   - 把自己 partition 的同时 sort 每个 Bucket
   - **Read** data + **write** partitioned data
   - cost = 2 \* (pageNum1 + pageNum2)
2. Phase 2 - cost = Pages table1 + Pages table2
   - Join **each partition** pair (same hash value)
   - Read data (看情况 Write)
   - cost = 1 \* (pageNum1 + pageNum2)

- Memory
  - Phase 1:
    - 1 + # buckets
  - Phase 2:
    - 2 + # Pages in smaller table / # buckets
  - **Bucket num = $\sqrt(\text{smaller table Page num})$**

## Equality Joins - Sort-Merge Join (SMJ) (Sub-optimal in memory)

- Phase 1
  - 2 ** M ** ($1 + Ceil(log**{B-1}(M/B))$) for sorting table 1
  - 2 ** N ** $(1 + Ceil(log**{B-1}(N/B))$) for sorting table 2
  - Multiple **sorting passes**, we read and write data once in each
    - Cost per pass is **2 \* (number of pages)**
  - steps to make with B buffer pages:
    - First step: runs of length B
    - Second step: runs of length (B-1) \* B
    - Third step: runs of length (B-1) ** (B-1) ** B ...
- Phase 2
  - **M+N** (we don't count cost for writing output!)
  - After sotting, all duplicate entries on same page
    - Duplicate entry: same value in join column
  - Each input page is only read once
  - Cost is proportional to number of input pages
- Memory
  - Phase 1: use all buffer pages
    - More buffer == less merging passes
  - Phase 2: exploit 3 buffer pages
    - 3 for 1st input & 2nd input & output

## Refined Sorted-Merge Join (Refined SMJ)

`2 * (M+N) (Phase 1) + 1 * (M+N) (Phase 2)`

- Memory
  - First phase:
    - **(N+M)/B** sorted runs
  - Merge in one step:
    - **B ≥ 1+(N+M)/B**
  - Rule of thumb:
    - if N>M: need **B ≥ 2 \* √(N)**

## R-SMJ vs Hash Join

![[Screenshot 2024-09-25 at 10.24.37 AM.png||300]]

# Transaction Manager

- Assign query to transactions
- 规则
  - 开始: `BEGIN`
  - process transactions
  - 结束: `COMMIT, ABORT`
- Lock
  - 多个 trans 使用一个 object 可能造成冲突
  - tran 可以获取 Read Lock / Write Lock
  - **Read (aka shared) locks**: read access
  - **Write (aka exclusive) locks**: read+write access

## ACID

### Atomicity

Execute All or none

- **undo logs**
  - Log all details of a transaction
  - Undo transactions if any of them aborts
  - Maintain these records in memory and on disk

### Consistency

保证数据完整, 如果 abort 那就要做好信息恢复

- 永远满足数据的约束 (如外键约束, 唯一性约束)
- **abort** transactions threat consistency

### Isolation

数据隔离

- Looks like sequentially but interleaving
- Interleave: 暂停当前 Tran, 执行其他 Tran 来保证效率
- Postgres
  - `<isolation-spec> ::= isolation level <i-level>`
  - Default: `READ COMMITTED`
  - `i-level`: Isolation Level
    - SERIALIZABLE, REPEATABLE READ, READ COMMITTED, READ UNCOMMITTED
    - **Read uncommitted**: 读取不需要等待其他事务提交
    - **Read committed**: 只能读取已提交的数据变更
    - **Repeatable Read**: 生成一个快照，事务在整个执行过程看到的都是快照, 但是快照无法避免插入
    - **Serializable**: 所有读取操作都是串行执行的
  - ![[Screenshot 2024-10-12 at 3.24.32 PM.png|300]]

### Durability

信息不丢失, 所有加入的信息都会永久保存

- redo logs 或 write-ahead logging (WAL) 保证 crash 后仍然不丢失信息

### Interleave Anomaly

- Write-Read (W-R)
  - Dirty Read: T1 写了&没有 commit 的信息被读到
- Read-Write (R-W)
  - Unrepeatable Reads: T1 第一次读取之后, 信息被 T2 更改&commit, 第二次读的时候信息改变
- Write-Write (W-W)
  - Lost Updates: T1 写入&commit, T2 写入&commit 导致 T1 白写了
- Phantom Read (幻读)
  - 由于其他事务的提交，执行同一查询时可能会返回之前未返回的数据记录
  - 和 R-W 不同, 没有信息被改, 但是插入了新的信息
  - Example: 之前只有一行, 现在有两行
  - 解决:
    - **Predicate locking**
      - Locks current and future entries equally which is satisfying certain predicate
      - 锁所有相关的 row, 像是 `name like 'F%'
      - 保证之后带 F 的 entry 无法 insert
      - 无法处理没规则的内容
    - Index for equality predicates
      - Lock index page that would change at insertion
      - Cannot insert if index page is locked

## Schedule

给 trans 排序, 构成 schedule & 不会构成 Anomalies

### Serial schedule, 第一安全

按顺序进行, no interleave, 较慢

### 2PL schedule, 第二安全

- Ensure conflict serializable schedules
- schedules generated by Two-Phase Locking (2PL) 看下面锁的部分

### Conflict serializable schedule, 第三安全

通过 swap non-conflicting operation 转换到 serial schedule

- conflict operation:
  - involve the **same data item**.
  - At least one of the operations is a **write**.
- 如果 conflict graph 没有 cycle: 是 conflict serializable

#### Conflict Graph (acyclic graph)

- node 代表 transaction
- edge from i to j:
  - 逐步检查冲突并构建
  - i 和 j 是 conflict operation, i 比 j 早
- example
  - `R1(A), W1(A), R2(A), W2(A), R2(B), W2(B), R1(B)`
    - W1(A), R2(A) 导致 edge from T1 to T2
    - R1(A), .., .., W2(A) 导致 edge from T1 to T2
    - W2(B), R1(B) 导致 edge from T2 to T1.
- 转换成 serial schedule
  - 从 node without incoming edge 开始
  - Add all operations of that transaction and commit
  - Continue with node where all predecessors treated

### View serializable schedule, 第四安全

如果先执行 T1, 那之后都要先执行 T1. 在此同时, 确定每个 T 的 W/R 顺序不变

- 可能会有 W-W conflicts
- Same initial reads
  - If reads **initial value** in S1, 要读 S2 的 initial value;
  - 两个 Schedule 第一题读的信息要一样
- Same dependent reads
  - If reads a **value written** by transaction Y in S1, 要读被 TranY 写入的 S2 部分
  - 两个 schedule 读的数值都来自于同一个 Trans 写入的数值
- Same winning writes
  - If writes the **final value** written by transaction Y in S1, it also does so in S2
  - 两个 schedule 最后写入给同一个 object
- R1(A) W2(A) R1(A) C1 C2
  - R1(A) R1(A) C1 W2(A) C2
    - 第二次 R1(A)读的 dependent 不同
  - W2(A) C2 R1(A) R1(A) C1
    - 第一次 R1(A)读的 dependent 不同

### Final state serializable schedule, 第五安全

和 serial 比较 final state 是否相同, 不关心执行顺序或中间状态是否一致

- 无法保证 unrepeatable reads
- R1(A), W2(A), R1(A) 和 R1(A), R1(A), W2(A)一样
- 但是第二个 R1(A)的结果不同

## Abort

处理 abort 对 schedule 和 serializability 会产生影响, 所以有不同的处理方式

- example
  - W1(A) W2(A) W3(A) not strict (ACA & recoverable)
    - ACA: W2(A)不是读, 是写
  - W1(A) C1 R2(A) W2(B) C2 strict

### Recoverable Schedules

- 当前 Trans 必须要等 all transactions it read from 全部 commit**才能 commit**
  - 更好 rollback
  - 如果 abort, 导致当前 tran 也要 abort
- 如果 T1 读过 T2 写的 object, 那必须要等 T2 先 commit
- 防止: `W1(A) R2(A) W2(B) C2 A1`

### ACA Schedules (Avoid Cascading Aborts)

- 只能读 commit 后的 object
  - 防止 wide spread rollback aborts & 避免 dirty reads

### Strict Schedules

- 只能 read / write committed object
- highest level of isolation and make recovery simpler and more reliable.

## Lock

### Fine-grained - 用于锁较小 unit

- increase degree of parallelism
- increases locking overheads

1. request 所有需要的锁 at start
2. 等待拿到需要的锁
3. release at end

- Coarse-Grained Lock 用于锁较大 unit

#### Multiple-granularity lock - 既可以锁大也可以锁小

- 先锁 table, 再锁 row, 然后**由下往上解锁**
- **IS (Intention Shared)**: shared lock
  - 先 IS ancestor, 然后 S 小的
- **IX (Intention Exclusive)**: exclusive lock
  - 先 IX ancestor, 然后 X 小的
  - 同一个 Object 可以放多个 IX
    - 因为两个 trans 可能找的不是同一行
  - 不能放 X 到同一个 row

### 2PL

不能太早 release 也不能太晚 request lock

- 确保 conflict-serializable schedules
  1.  request 所有需要锁, S lock before read, X lock before write
  2.  release 所有的锁, 再也不能 require

#### Variant

- **acquire Locks late**
  - Tran 开始读/写之前 request 当前 operator 需要的 lock
  - increasing parallelism
  - 导致**deadlocks**:
    - Transaction 1 acquires **lock on A**, now **waiting for B**
    - Transaction 2 acquires **lock on B**, now **waiting for A**
    - W1(A) R2(A) C1 C2
      - A 的 lock 被 T1 拿到了, 等 C1 之后才能执行 R2(A)
- **released Locks early**
  - Release lock after last operation on associated object
  - 导致 cascading aborts: `W1(A) [Lock on A from 1 → 2] R2(A) A1`
- **Conservative 2PL:** 在开始运行之前就拿到所有要用的 lock
  - 避免 deadlocks
- **Strict 2PL:** release all locks at transaction end
  - 避免 cascading aborts
- **Conservative Strict 2PL**
- **Plain 2PL** : not Conservative or strict
  - **permissive** - proceed in parallel
  - 会造成 deadlocks or cascading aborts

#### Proof 没做, 看 slides

#### Index Lock

Genetic 2PL 不好: 会锁掉 root, 因为要 lock 整个 path

- 从 root 到 leaf:
  1.  锁 cur
  2.  look up children
  3.  锁 child, 解锁 cur
- 更新 leaf node 之后:
  - 向上 propagate 直到 safe node (没有 full 但是大于一半 - insert, update, delete 不会影响到)
- 悲观: request write locks: reduces performance
- 乐观: request read locks for all nodes except leaf
  - Bets on no propagation, may have to restart if we lose

### Deadlocks (for non-conservative 2PL)

解决方式: 发生后解决 OR 避免 deadlock 的发生

#### Detection

- Option1: 超时就是 deadlock
- Option2: Waits-for graph
  - node: transaction
  - T1 -> T2: if T1 waits for lock held by T2
  - Cycle = deadlock

#### Resolve

- abort 一条 tran
- 优化: abort 最新加入的 tran

#### Avoid / Prevention

priority: 越老越高 priority

- Higher priority tran 不会 abort
- 迟早会变成最高 priority
  - 不会 restart timestamp

##### Wound-wait

Only **lower** priority can wait for lower priority

- 更高的 Tran, T1 要 T2 的 lock
  - T2 abort
- 更低的 Tran, T1 要 T2 的 lock
  - T1 wait
- Proof by contradiction in waits-for graph

##### Wait-die (和 wound-wait 的 wait & abort 逻辑完全相反)

Only **higher** priority can wait for lower priority

- 更高的 Tran, T1 要 T2 的 lock
  - T2 wait
- 更低的 Tran, T1 要 T2 的 lock
  - T1 abort
- Pro: Transactions that acquired all locks won't abort (higher)
- Con: 更低的 Tran 可能无限 abort
- Proof by contradiction in waits-for graph

## Concurrency control CC

### Bookkeeping

用于 trans, 乐观锁用它来解决 conflict, 悲观锁用它来避免 conflict

- Read Set 保存被读的 object, 检测是否被其他 tran 修改
- Write Set 保存被写的 object, 检测写入冲突

### 乐观锁 optimistic CC

- 在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据.
- 很乐观, 不会有 conflict

1. 读: 当前 tran 在一个 private copy 上执行
2. validate: 按时间排序, 在 read & write set 中检查是否有 conflict
   - 没有 conflict:
     1. T1 在 T2 开始读前就已经写入 db
     2. T1 在 T2 写入之前就已经写入 db
     3. T1 先读, 然后 T2 再读
3. 写入到 database

### 悲观锁 pessimistic CC

- 悲观, 会有 conflict
- 可能导致 deadlock & overheads, 因为在不停的 acquire & release
- 给每个要用的 object 一个锁, 如果已经被锁那就等 release

### Timestamp (TS) CC

- TS(T): T 启动的 timestamp
- RTS(A), WTS(A): object A 最后读取时间/写入时间
- 读 A
  - Tran 之后被改过: TS(T) < WTS(A): abort + restart
- 写 A
  - Tran 之后被读过: TS(T) < RTS(A): abort + restart
- Thomas Write Rule
  - 如果写 A 并且 TS(T) < WTS(A)
    - R1(A) W2(A) C2 W1(A) C1
    - 基于旧的 TS, 这次写入毫无意义, 直接忽略 - W2(A)
    - 破坏 conflict serializable & 保证 view-serializable
    - Simplifies to R1(A) C2 W1(A) C1

### Multi-version concurrency control MVCC

- **以 Tran 启动的时刻为准，只看在 Tran 启动之前的数据版本, 否则一直找**
  - 确保 serializability
  - 每个 tran 和 object 都有对应的 timestamp
  - write 会创建新的 timestamp 给 object
  - 当前 tran 只读比自己 TS 要早的版本
    - tran 的 TS = i, 那么只能读 object TS = j: j > i, j 比 i 老
    - 如果找不到, abort
- Example
  - R1(A) W1(A) R2(A) W2(B) R1(B) W1(C)
  - Not conflict-serializable
  - R1(B) 放到 W2(B) 前面 - 确保 R1(B) 读旧版本

### Snapshot isolation SI

- 每个 tran 都执行于 snapshot - last committed value for each object
- SI 不理会 uncommitted values, 但是 MVCC 会
- Write - commit 之前检查写入的 object 有没有发生变化
  - 如果变化 - abort + restart tran
  - 这部分 MVCC 没有
- Example
  - T1: Insert into B select count( \* ) from a;
  - T2: Insert into A select count( \* ) from b;
  - 两个的 snapshot 都过期了 - 导致 write skew
    - 写乱了, 读乱了 - each transaction reads overlapping data and writes to disjoint data based on the reads

# Recovery

- 通过 log, 在 crash 时保证 ACID
  - undo log
    - 取消已经执行的指令
    - 保证 Atomic
  - redo log
    - 重新执行那些还没有写入 disk 的指令 (commit but not written to disk yet)
    - 保证 consistency
- policy
  - force
    - Tran 结束时, 写入 disk
    - 慢, 不需要 redo log
  - no force
    - 数据被 evicted from buffer 时, 写入 disk
    - 需要 redo log
  - steal
    - tran 结束之前就写入 disk, 很难 undo
    - 需要 undo log
  - no steal
    - tran commit 之前不能 evict memory 中更改的信息
    - 内存需求高, 不需要 undo log
- log types
  - Update:
    - 更新 T-Table 对应的 lastLSN
    - New value for redo, old value for undo
    - update 会更新的内容
      - PageID: 被修改的 page
      - Length: 更新的长度
      - Offset: 更新的起始位置
      - Before-Image: 修改前的原始内容 (用于 undo)
      - After-Image: 修改后的新内容 (用于 redo)
  - Commit & Abort:
    - 更新 T-Table 对应的 status
    - Commit: 写到 disk 的 log 中 (write-ahead log)
    - Abort: 触发 undo
  - End:
    - 从 T-Table 删除
  - **Compensation**: 已经 undo (undid) 了之前的 operation
    - 记录已 undo 的操作
    - recovery 可能导致 crash, 避免无限 undo 相同的事情

## ARIES - recovery algorithm

- write-ahead log
  - 修改数据前, 先把 log 写入 disk
  1.  tran 在 commit 之前的每个 log 都写下来 - 用于 redo
  2.  buffer 里的每个 log 都写下来 - 用于 undo

### Data structure

log 通过 doubly linked list 链接

- LSN: log sequence number - log id
- entry: Log 内容
- PrevLSN: 当前 tran 的上一个 entry

1. 写入 log tail: 保存 Tran 的所有 activity
2. memory buffer 的每一个 page 都有 Page lastLSN - 更改 page 的最新 LSN
   - 写入 memory: 更新 LastLSN
   - 从 memory 写入 disk: 把 LastLSN 也写入 disk
3. T-Table: transaction table:
   - 保存 active trans: {transId, status, lastLSN}
   - 也可以用于追踪需要 undo 的 log (从 disk log 中提取)
4. DP-Table: dirty page table:
   - 用于发现 memory 和 disk version 的不同: {pageId, recLSN}
   - recLSN: 第一个写入当前 page 的 LSN - 作为 redo 的起点
   - page 更新:
     - 如果 page 不在 table, 加入 DP-Table & 当前 LSN
     - 如果 page 在 table, 不变
   - page 写入 disk:
     - 从 DP-Table 中删除
5. Flushed LSN (准备进 disk 层)
   - 一个数字 - 最新写入 disk 的 LSN
   - commit 时: `T Commits`
     - flushedLSN ≥ lastLSN (这里的 flushedLSN 指的是更新后的)
     - 确保所有的 log 都已经写入 disk
   - 写入磁盘时: `P is written to hard disk`
     - flushedLSN ≥ page's pageLSN
     - 确保所有会影响当前 page 的 log 都写入 disk
6. Log (disk 层)
   - crash 之后, 通过内容进行 recover

### Run time behavior

- buffer replacement policy

### Recovery algorithm - 3 phase

- checkpoint
  - 把 transaction & dirty page 写到 disk, 这样不需要从头开始 recovery
- 三个 phase 都是从 Log 文件读取
  - 如果任意阶段 crash, 从头开始

1. **Analysis (分析阶段)**:
   - 从 checkpoint 开始, 恢复 T-Table & DP-Table
   - update
     - 如果 page 不在 DP-Table, 插入 page, recLSN = 当前 LSN
     - 如果 Tran 不在 T-Table, 插入 page, 设定为 running, lastLSN = 当前 LSN
   - commit / abort
     - 插入 T-Table, 设为 committed / aborted
   - end
     - 从 T-Table 中删除当前 Tran
   - 结果:
     - T-Table 包含未完成 Tran & lastLSN
     - DP-Table 包含脏页 & recLSN
2. **Redo (重做阶段)**:
   - 根据日志和 **DP-Table** 的 **recLSN**，重做所有已提交事务的更改
   - **只能 Redo updates / CLR**
   - 从最小的 recLSN 开始
     - Redo updates
       - 修改页面 + 更新 pageLSN
       - 不生成新 log
     - Redo CLR (compensation log records)
       - 重做之前 undo 的效果
   - 按顺序检查, 如果满足, 跳过 redo
     - Page updated, not in dirty page table
       - 当前 transaction 已经写入 disk
     - Page updated, in DP table, recLSN > LSN
       - 现在 redo 这个 page 有点早
     - PageLSN is on hard disk, pageLSN ≥ LSN
       - 已经写入 disk
3. **Undo (撤销阶段)**:
   - 使用 **T-Table** 和 **CLR**，撤销所有 aborted Trans
     - 如果是看到`T1 commit`, 不需要 undo 之后 T1 的 Trans
   - 相反顺序 - 利用 prevLSN, 从最老的 log 开始
   - Write compensation log records while undoing
     - `CLR T Updates P`

# DB Design

1. Requirement analysis
   - 根据需求, 分析需要什么东西
2. Conceptual Design
   - 通过 ER diagram 设计 DB
3. Schema normalization
   - 优化做好的 design
4. Physical tuning
   - 实施

## Step2: Conceptual Design

### Design - ER diagrams

- **Type**
  - Binary: relationship 和两个 entity
  - Ternary: relationship 和 3+个 entity
- **entity (长方形)**
  - Owner Entity (实线)
  - Weak Entity (虚线)
    - 和 owner 用 relationship 相连 (至少一次)
    - 如果 owner 不存在, 那么当前的 entity 也不存在
    - 独一无二, 不能有重复, 类似于 Key
- **Property Attribute (椭圆)**
  - 下划线 - key attribute
  - simple values (e.g., integer)
- **Relationship (菱形)**
  - entity $\to$ relationship
  - **粗直线** Participation: at least once
  - **箭头** At-most-one: at most once
  - 连接所有 entity 的组合必须 unique
- **Roles (label on edge)**
  - 当连接多个相同的 entity, 一定要有 role
- **Subclasses (三角形)**
  - 继承 attributes & relationships
  - 只能继承一层
- **Aggregation (用虚线圈起来 relationship)**
  - Models **relationship of a relationship**
  - 和 Ternary relationship 不同
    - Ternary - Each entity can have a constraint but not on the combination

### Relations

- entity == row
- Properties == columns
- Relationships
  - Columns == attributes
    - attributes (有下滑虚线) == primary key
  - Row == **relationship** between specific entities
  - Optimized: Binary relationship type by constraints
    - Many-Many: no constraints
      - Separate tables
    - Many-One: one at-most-one constraints
      - add a column in the 'One' table
    - One-One: both at-most-one constraints
      - Use one table
- Sub-Classes
  - additional attributes
  - 3 options
  1.  Separate relations for superclass and sub-class
  2.  multiple relations linking key to attributes
  3.  Use relation for **sub-class**, set unused attributes to null
- Weak Entities
  - Introduce **new relation** for storing weak entities
  - Add **foreign key columns referencing owner** entity
  - In SQL: **cascading delete** depending on owner

## Step 3: Schema normalization

- 通过 schema normalization 来优化 initial schema
- Anomaly
  - Update 修改某些值导致信息不一致, 比如 foreign key
  - Insert 新的 row 缺少某个 column
  - Delete 意外丢失重要信息

### 1. Functional dependencies (FD)

- 如果 Y dependes on X `X -> Y`, 那就说明不需要 Y
- **通过结构层级: FD -> FD Closure -> Attribute Closure 检查 + 寻找冗余**
  - Attribute: 一个 Column.
    - 如果两个 Attribute 有 dependency 关联: Attr1 决定 Attr2
      - `Attr1 -> Attr2`
  - FD: 描述 Attribute 之间的关系
    - F = `{A -> D, AB -> E,..}`

#### 寻找 FD

- 不能只看 data, 要看 dependency + 逻辑
- 通过 domain knowledge (常识) + 推导 FD 来找最优 FD

#### Inferring FD & FD Closure

- `F1 |= F2` = F1 imply FDs F2
- **Armstrong's Axioms**
  - **Reflexivity**: if Y ⊆ X then X → Y
    - 大的指向小的
  - **Augmentation**: if X → Y then XZ → YZ for any Z
  - **Transitivity**: if X → Y and Y → Z then X → Z
- Example
  - FD: Zip Code → County, State
  - Reflexivity: County, State → County
  - Augmentation: Zip Code, Name → County, Name
  - Transitivity: State → Country then Zip Code → Country
- FD Closure
  - Closure: 利用 Armstrong's axioms 推导所有的 FD
  - `F+ = {f|F |= f}`
  - F is a **cover** for G if `F+ = G+`

#### Attribute

- Attribute Closure 属性集, 从 FD 推导出集合
  - `X+` = X 的 Attribute Closure
  - 用于检查特定 FD 是否存在
- 算法 1️:
  ```python
  # O(len(FD)^2 * len(X+))
  while (True):             # O(len(FD)), 一直重复, 直到X+不变
  	for (A -> B in FDs):  # O(len(FD)), traverse所有的FD
  		if A in X+:       # O(len(X+))
  			X+ = X+ + B
  	if X+ not change in this loop:
  		break
  ```
- 算法 2: 这个应该不重要
  - Finding All Keys of a Relation
  - Iterate set A 的 Attribute
    - if A is key: (A)+ includes all attributes:
      - Calculate attribute closure (A)+

### 2. Normal forms

- 通过 FDs 来 decompose table 删减冗余信息
- key
  - key: 在某个 table 中, 唯一决定所有属性
  - super-key: 在 key 的信息上加额外属性
  - minimal key: 当 key 可以有多个的时候, minimal key 是其中的一个
  - 寻找 key:
    - 找出四个 set: L, R, LR, N
    1. 找 L join N 的 closure, 如果能 derive 到所有, 那就说明当前是最佳 key
    2. 如果找不到, 在 LR 中选一个, 找能 derive 到所有的组合, 可以出现多个 key
- `A -> B`
  - A: determinat
  - B: dependent attribute
- BCNF & 2NF: 基于 FDs, 检测 tables 是否 valid
  - 如果是 BCNF, 那么一定是 3NF

#### Boyce-Codd Normal Form (BCNF)

```python
for A -> B in FDs:
	if B in A OR A is key: # super key
		valid
	else:
		invalid
```

- example
  - `FDs = {a -> b, bc -> d, a -> c, d -> ae`
  - `Tables = {a, b}, {a, d, e}, {c, d}
  - 1 valid: `a -> b` a 是 key;
  - 2 valid: `d -> ae` d 是 key;
  - 3 valid: `d -> ae && a -> c` 所以 `d -> ce` 所以 `d -> c`

#### Third Normal Form (3NF) 有点不是很确定

```python
for A -> B in FDs:
	if B in A
		OR A is key # super key
		OR B in minimal**key: # a part of candidate key
		valid
	else:
		invalid
```

### 3. Normalization algorithms

1. reconstruct (decomposition): 比如把一个 table 分成两个 & 不篡改信息
   - 把 R 分裂成 X 和 Y 两个 table: R => X and Y
   - if (X ⋂ Y) → (X - Y) or (X ⋂ Y) → (Y - X) is an FD
     - **match** each row from Y to one row from X
     - OR
     - **match** each row from X to one row from Y
2. recompose (join): 分裂的 table 组合, 检查有没有丢信息
   - Lossless
     - 合并 table X 和 Y 到 R 不会丢失/增加信息
       - **lossless decompositions**: (R-b)⋂Ab=A→b

#### BCNF

- 如果不满足 BCNF
  - **Decompose** R into R-b and Ab
- 最终结果取决于 decomposition order
- Example
  - `R = CSJDPQV, Key C, JP -> C, SD -> P, J -> S`
  - 因为 SD -> P: 分解`CSJDPQV`=> `SDP`, `CSJDQV`
  - 因为`CSJDQV`不满足 BCNF, 继续分解
  - 因为 J -> S: 分解`CSJDQV` => `JS`, `CJDQV`
  - 结果: `SDP`, `JS`, `CJDQV`

#### Dependency Preservation

- 分裂后, FDs 仍然适用于 X 或 Y
- (BCNF 确保这个情况, 3NF 不确保)
  - 分别强制 X 和 Y
    1. enforce all FDs that only use attributes on X
    2. enforce all FDs that only use attributes on Y

#### Towards 3NF

- BCNF + 1 extension
- If A→b **broken** then add relation Ab
- Use **minimal cover** FDs
  - 每个 FD 的右边值只能是一个 attribute
  - Closure changes when **deleting any FD**
  - Closure changes when **deleting any attribute**

# CAP

- Consistency
  - 每个 node 的信息一致, write requires lock
- Availability
  - 每个 node 都可以正常工作
- Partition Tolerance
  - 系统部分故障不会影响系统的工作状态
- 在满足 P 的情况下, 要在 A 和 C 之间做出选择
- C 不是绝对的, 可以选择强一致性、因果一致性、最终一致性...

## Eventual Consistency 最终一致性

- 保证不同的强弱关系 & 覆盖范围

1. 强一致性 **Strong** consistency: 更新后可以立刻读取新内容
   - 金融系统
2. 弱一致性 **Weak** consistency: 更新后有可能读取的是旧内容
   - 社交媒体的点赞数
   - **Eventual** consistency: specializes weak consistency
     - 在足够时间后达到一致性
     - DNS 更新
   - **Read-your-writes** consistency:
     - 能读到自己更新的信息
     - 发表帖子
     - **Session** consistency: 总能看到自己写入的最新值
   - **Monotonic read** consistency:
     - 不会出现“先看到新值，再看到旧值”的情况
     - 文章更新

### SQL

- 专注于 consistency, 导致 performance 较低
  - ACID transaction

### NoSQL

- 削弱 consistency, 专注于 availability
  - BASE transaction - Basically Available Soft State Eventually Consistent
  - performance 较高
- give up ACID for higher performance
- 购物

### Apache Cassandra

- Distributed system, every node with same role
  - fault tolerance
  - linear scale: 每加一个 node, 就会多一个 worker
- wide column store: table 中, 每个 row 有不同的 columns
- 比 SQL 简单, 没有 join
- 通过 Tunable consistency 实现最终一致性
  - node 之间定时交换信息
  - 每个 node 有 peer 数量显示
- Tunable Consistency
  - parameters:
    - **Replication** factor (i.e., how many copies): N
    - Number of replicas for **successful read**: R
    - Number of replicas for **successful write**: W
  - Strong consistency if **R + W > N**
    - If so, read at least one **updated** replica
    - Use timestamps to identify **most recent** version

## Dynamo

- 最终一致性
  - 保证‘always writeable’
- Write
  - 越快相应越好
  - 造成短期 inconsistent data
- Read
  - 基于 inconsistent data
  - 如果是旧版本: 丢弃 + 使用 **reconciliation** strategy
- 如何检测是旧版本
  - 每个 object 都有一个 monotonic vector lock 记载 update count
  - 通过检车每个 node 的 object 的 vector lock 分辨新/旧版本

## NewSQL - H-Store

- 在 memory 中运行 - ACID & Performance 较高
  - 通过 H-Store 违反 CAP 规律, 但是更多偏向于 CP
- 满足 consistency: 省去了 IO 的消耗, 可以进行 serial execution
  - 不用 disk 的写入/读取 (log)
  - 无锁 (serial)
  - Pre-processing partitioning, 事务大多在单个分区内完成，因此不需要跨节点的网络通信或磁盘 I/O。

# Graph Data

- 建立 Node & Edges, 每个都有自己的 label & Property
  - Social networks, knowledge graphics, road networks

## Relational DB

- Relational DBMS 可以存但是很慢 - self-join 太贵了

```SQL
create table Stations (
	StationId int primary key, name text
);

create table Connected (
	StationId1 int, StationId2 int,
	primiary key (StationId1, StationId2),
	foreign key (StationId1) references Stations(StationId1),
	foreign key (StationId2) references Stations(StationId2),
)
```

- Find Paths from P to N (Self Join)

```sql
SELECT * from Connected C1
	join Connected C2 on (C1.stationid2 = C2.stationid1)
	join Connected C3 on (C2.stationid2 = C3.stationid1) 
...
join Connected Cn ... 
WHERE C1.name = 'Port Authority'  
and Cn.name = 'NYU'
```

## Graph Data - Cypher in Neo4j

- 可以运行复杂的 patterns 来搜索 graph
  - 支持 Aggregation, filtering, sub-queries etc.
- Data layout 数据存储方式 & 结构
  - 针对 Graph 优化, 支持图遍历和查询操作
  - node:
    - node 同学小张
    - label 是分类:同学
    - property 是数据: name
    - edge reference: list of incoming and outgoing edges
  - edge:
    - 同学之间的 edge
    - label 是分类: 同学
    - property 是数据: since
    - node reference: 两个节点
- Transaction - ACID
  - read-committed isolation by default
  - 用锁来实现 isolation
  - 用保存 log 到 disk 来实现 durability

### Query

- Create Node - `CREATE (:label {properties})`
  - `CREATE ()` Create node without labels or properties
  - `CREATE (:Student)` Create node labeled as student, no properties
  - `CREATE (:Student {name : 'Marc'})` Create node labeled as student, name set to 'Marc'
- Find Node
  - `MATCH (m:Student {name : 'Marc'})`
  - Finds nodes labeled as "Student", name property as "Marc"
  - Assign result to variable `m`
- Create relationship
  - Relationship as label `Enrolled` between `a` & `b`
  ```Cypher
  MATCH (a:Student {name: 'Marc'}),
  	(b:Course {name: 'CS4320'})
  	CREATE (a)-[:Enrolled {semester: 'FS20'}]->(b) 
  ```
  - Inserts edge from a to b with label "Enrolled"
  - Set Edge property "semester": "FS20"
  - `:Relationship*` - chain
    - 加星号代表所有直接 & 间接关系
  - `:Relationship*0..2` - chain
    - 以当前 match 为中心, 扩散 0~2 层内的所有 match
- Update Node
  - `MATCH (m:Student {name: 'Marc'}) SET m:Alumnus`
    - Change label of Marc from Student to Alumnus
  - `MATCH (m:Student {name: 'Marc'}) SET m.name = 'Marcus'`
    - Change value of name property to "Marcus"
- Update Relationships
  ```Cypher
  MATCH (a:Student {name: 'Marc'})
  	-[e:Enrolled {semester: 'FS20'}]-
  	(b:Course {name: 'CS4320'}) 
  	SET e.semester = 'FS21'
  ```
  - Find relationship `e`, then set its property
- Delete Node
  - `MATCH (a:Student {name: 'Marc'})  DELETE a`
  - Deletes students with name "Marc" from the database
  - **Delete relationships before deleting nodes**
- Aggregation
  ```Cypher
  MATCH ( :Student {name: 'Marc'} )
  	-[:friendsWith]-> (:Student) 
  RETURN count(*)
  ```
  - Count number of friends of Marc
- Examples
  ```Cypher
  MATCH (s1:Student) -[:friendsWith]->(s2:Student),
  	(s1)-[:Enrolled]->(c:Course),
  	(s2)-[:Enrolled]->(c) 
  WHERE s1.name IN ['Marc', 'Maria']
  	AND NOT c.name = 'CS4320'
  return s2
  ```

### Query Processing

- 由 standard operators & graph-specific operators (e.g., shortest path) 组成
- 通过比较 cost 来选择最优 query plan
- index - retrieve specific nodes/edges

## Distributed Graph Processing

- 一个机器跑 graph 可能会爆
  - 管理 cluster 来运行

### PageRank

- 把网络格局转换成 graph, pageRank 代表能访问到该网站的概率
  - pageRank 越高, more preferable in search result
- Node: 网站
- Edge: 链接

#### Random Surfer model

```python
f(任意网站)

def f(当前网站 x):
	for (outgoing link : x):
		f(x)
	f(任意网站)
```

1. 选取任意网站
2. 选取该网站任意 outgoing link
   - 选取概率为 𝛂, 继续执行 2
   - 如果没有 outgoing link, 回到 1

#### Calculating PageRank - iterative algorithm

```python
def PageRank(G, alpha):
    N = 节点总数
	# 初始化每个节点的PageRank值为1/N
    PageRank = {node: 1 / N for node in G.nodes}
    # 存储下一次迭代的PageRank值
    new**PageRank = {node: 0 for node in G.nodes}

	# 遍历每个节点，分配PageRank
	for node in G.nodes:
		outgoing**links = G.outgoing**links(node)

		# 将PageRank分配给出边指向的节点
		for target in outgoing**links:
			new**PageRank[target] += PageRank[node] / len(outgoing**links)
			PageRank[target] += new**PageRank[target] # 准备下个iteration

    return new**PageRank
```

- 每个 node 起始值: to 1/NrNodes
- In each node
  - redistribute PageRank over links
  - Each node partitions PageRank among outgoing links
    - node 的 pageRank 是`x`, 那么 outgoing link 会传递 `x / #outgoinglink`
  - PageRank in next iteration = sum over incoming links
    - sum 收到的所有 link

### Pregel

- Worker nodes process their partition (each has >= 1 partitions) in parallel

#### Computation Model

- Supersteps: 把 computation 分解成多个 iteration
- each iteration
  1.  invoke Compute for each node
      - Compute function:
        - customized by user
        - Input: messages sent to this vertex in prior iteration
  2.  message other nodes to deliver in next iteration
- ends if all nodes vote to halt

#### Fault Tolerance for large cluster

- Workers persist input and state at iteration start
- Coordinator 通过 ping 检查 worker 宕机情况
  - Recovery may start several supersteps earlier
  - Re-partition graph to replace failed workers
- "Confined recovery" restricted to failed partitions
  - Requires persisting outgoing messages as well

### PageRank in Pregel

```python
Compute(ReceivedPR : int[]):
	NewPR = sum(ReceivedPR)
	For o in OutgoingLinks:
		Send(o.target, NewPR/|OutgoingLinks|)
```

- Basic version sends lots of page rank values
- aggregate messages via custom "Combiners"
- Here: can combine page rank for same target as sum

# Data stream 数据流

- query 一直在跑并且高频率地 generate data
- 保持高效
  - 确保 memory 低利用率, 因为数据量非常大
  - 特定 data structure for fast insert (queue, hash...)
  - 多来源: data 可能有多个来源, 所以更多和 ETL 一起使用

## Steam System

- query 和 data 有两种状态: dynamic or static
- Database management: static data & dynamic query
  - 信息源不会变, 通过改变 query 得到不同的信息
- Stream Data management: dynamic data & static query
  - 信息源会变, 在 query 不变的情况下也会有不同的信息

### Relation & Stram

- R(t): 在时间 t 的情况下, table / relation R 的信息不同
- S: Stream, 包含 timestamped tuples
- Tuple
  - Stream 只能 generate (add) tuple, 数据流是没有边界的, 所以要用 window 限制
  - Relation 可以 add/delete tuple

#### R -> R

SQL

#### S -> R

- 把 stream 的某个时间段 (sliding window) 转换成 tableR(t)
- **Tuple-based** sliding window: **S [Rows N]**
  - R(t) - 把最新的 N 个 tuples 存到 table
  - `select * from StockStream [Rows N]`
- **Time-based** sliding window: **S [Range T]**
  - R(t) - 把时间段内的 tuple 存到 table
  - `select C.id from Customer [Range 2 Minutes] C` 从现在到 2 分钟前
- **Partitioned** sliding window: **S [Partition by A1, A2, ... Rows N]**
  - 每个 combination (A1, A2)都代表一个 window, 分别存入 table

#### R -> S

- **Istream(R)**: 根据 table 内插入数据的时间
  - `select Istream() from ..`
- **Dstream(R)**: 根据 table 内删除数据的时间
- **Rstream(R)**: 根据 table 内每个时间段的信息

### Query Processing

- Process 步骤
  1.  Input query
  2.  continuous query plan:
      - 静态查询计划: 预先定义所有操作，适合关系型数据库。
      - 动态查询计划: 根据数据流变化动态调整，适合数据流处理。
  3.  standard operators: Join...
      - 每个 operator generate 出的 tuple 带有 timestamp
- Adaptive Query Planning
  1.  **Re-optimizer** keeps requesting Statistics to **Profiler**
      - 基于 stats 优化 query plan
      - profiler 用于收集 stats
  2.  Re-optimizer update query and send to **Executor**
      - executor 用于执行 query plan
- **Synopses**: the state to be maintained by different operators (e.g. hashTables)
  - Join: hash table for Input 1
  - 多个 operator 可能产生相同 synopses

#### Join

- data structure: 2 inputs with 2 hashTables

```sql
SELECT *
FROM S1 [Rows 1,000], S2 [Range 2 Minutes]
WHERE S1.A = S2.A and S1.A > 10
```

- 触发条件: S1 改变 - add / delete data

1. 拿到 **join key**: A in S1
2. use S1.A to Probe (search) hash table of Input 2 with key
   - 拿着 join key 找 S2.A
3. 如果有 S2.A, 检查 join condition
   - If true, add tuple to output
   - 如果是从 S1 中删除, the joined tuples will remove the tuple
4. Update (insert / remove tuple) synopsis

### Minimizing Space Requirements

- 使用**Exploit constraints**删减冗余信息
  - **Referential integrity** k-constraint
    - 保证在匹配的元组到达之间最多有 k 个元组的延迟。
    - 最普通的情况 - key-foreign key joins
  - **Ordered-arrival** k-constraint
    - 只保存先到的 input 的 hash, 并且保证 sorted hash (at least k tuples by key / time-stamp)
    - 如果 A1 先到, 那只需要保存 A1 HashTable, 每个收到的 A2 都去 Sorted A1 Hash 找
  - **Clustered-arrival** k-constraint
    - Elements with same key can be at most k tuples apart
    - 如果 clustered id 改变, 意味着上一次的 clustered id 不会再出现, 可以删除 prev clustered id
- 通过 synopsis sharing 减小 memory 利用率
  - Global synopses with operator-specific views, 特定 operator 查看特定部分
  - one synopses for different plans, 合并 operator 的 synopses into one
- 通过 optimized scheduling 优化 intermediate results
  - 决定 operator 运行顺序来优化 queue size (空间利用率)
  - **FIFO**: fully process tuple batches in the order of arrival
    - 先到先跑
  - **Greedy**: invoke operator discarding most tuples
    - try to get rid of extra tuples ASAP
    - 优先调度可以丢弃最多 tuple 的 operator
  - **Mix**: combine operators into chains
    - 链内: FIFO (within one operator)
    - 链与链: greedy

### Approximation

- 更快, 但是会牺牲精度
- **Load shedding**: drop tuples to save overheads
  - approximate aggregates based on samples
  - balance impact over all aggregates
- **Reducing synopses** sizes: save memory
  - 因为 hashTable 变小, match 也会变小, reduces output size
  - exception: 有时候会导致 output 变大

## ksqlDB

- 两部分组成 composition:

### Apache Kafka Cluster

- 数据流处理引擎, in java
  - Kafka Streams API 提供各种 operator - filter / grouping...
  - 提供底层的分布式数据流存储和处理平台，负责消息传递、存储和分区管理
- 用于管理如何存储数据流 - Insert & Delete

#### 关键词

- topic
  - 用于管理消息 (消息的逻辑分类, 按顺序存储消息, Append-Only 不可变)
  - **key-value**: a log of ordered records
    - `id : {价格: ...}`
  - Regular topic:
    - 依据空间或时间限制定期删除旧记录
  - Compacted topic: new tuples override old keys
    - 通过增加 tuple 实现 override 来实现 update
- **Producers**: 向特定 topic APPEND records
- **Consumers**: subscribe to specific topics
- Partitions: topic 的物理划分
  - 支持水平拓展
  - 一个有序的, 不可变的日志文件序列
  - 每个 partition 分配一个 leader
    - leader 是 partition 的唯一接收点 - receive update from outside
    - 负责把信息共享到其他 replica
  - 每个 server 的备份, **replicated** across servers
    - 实现负载均衡 + 并行处理 + Fault tolerance

#### Insert

- 三种方式 + cost
  - Append-Only: 1 / Entries per page
  - Index: 1
    - Low insertion if random data access - not acceptable for streams
  - LSM: Size ratio \* Nr.levels / Entries per page
    - Size ratio: number of merges per level
  - use RocksDB engine
    - optimized for writes, good read performance
    - Key idea: sequential (instead of random) access
    - 利用 **WAL(Write-Ahead Log)** 来保护内存中尚未写入磁盘的数据, 防止丢失
    - 当 Memory 的 Buffer 满了, 写入 Disk
    - read 太慢了
- LSM (Log Structured Merge Tree : LSM (with Leveling Merge Policy)
  - **multiple levels (memory & disk)** for sorted/indexed data
    1. 首先放入 upper level
    2. 当 upper level capacity is full, 全部写入 lower
    - Constant **size ratio** between consecutive levels
  - Data from one level is **merged** into next at overflow
    - Merge operations need only **sequential** writes

#### Read

- 三种方式 + cost
  - Append-Only: O(all entries)
  - Index: 1 \* lookup cost (B+ Tree)
  - LSM: Nr.Levels \* Lookup cost
- LSM (Log Structured Merge Tree : LSM (with Leveling Merge Policy))
  - check each level is **fast** because data is sorted/indexed
  - **Bloom filters** reduce the number of levels to consider
    - 像是一个 hash table, 如果没有, 那说明当前 level 不存在
    - 高效的概率性数据结构, 可以快速判断某个值是否可能存在, 但是会有**假阳性**
    - Capture non-empty hash buckets
    - Summarize keys present at each level

### ksqlDB Server (Engine & Interface)

- 使用 Kafka Stream API
  - 在 Kafka 的基础上构建，提供一个 SQL 风格的接口来处理流式数据，降低开发复杂度，让用户专注于业务逻辑
- Pull: execute once on current state
  - Data source: Table
  - NO-windowed aggregation: lookup by key
  - return one result, 只跑一次
  - `SELECT * FROM pageviewsByRegionTable WHERE region = 'Ithaca'`
- Push: results continuously updated
  - Data source: Table & Stream
  - Life time: keeps returning updates
  - 一直跑, 直到收到 terminate
  - `SELECT * FROM clickEventStream WHERE region = 'Ithaca' EMIT Changes`
    - `emit changes`: 如果有 update insertion, output 会显示 (不会显示在 query run 之前的信息)

#### Collection

- collections - 定义和组织 Kafka 中的数据, 描述如何表示和管理数据流或状态
  - Stream - append-only 每条记录(topic)都会保留
    - represent historical information
    - 比如发生过的所有事件
  - Table - key-value collection - 用于表示当前状态
    - new entry overrides if had same key
    - 比如库存, 用户状态
  - 可以互相转化 Stream 和 table
- 创建

  - `kafka**topic = 'tickerTopic'`
  - create stream / table if the topic is valid
  - Need to specify primary key for table

  ```sql
  CREATE STREAM priceHistory(symbol varchar, price int)
  	WITH (kafka**topic = 'tickerTopic', value**format = 'JSON')

  CREATE TABLE curStockPrice(
  	symbol varchar PRIMARY KEY, price int)
  	WITH (kafka**topic = 'tickerTopic', value**format = 'JSON')
  ```

- 提取

  ```sql
  CREATE STREAM appleTicker AS
  	SELECT *
  	FROM priceHistory
  	WHERE symbol = 'AAPL'

  CREATE STREAM advertisementStream AS
  	SELECT *
  	FROM clickStream C JOIN advertiserTable A
  	ON C.advertiserID = A.advertiserID
  ```

#### Insert

- `INSERT INTO temperatureStream (Location, temperature) VALUES ('Ithaca', 32)`

# Spatial Data

- SQL 需要 spatial operator 实现
- Data Types
  - Point data
    - location: (x, y, ..)
  - Region data
    - boundary (line or surface)
    - have anchor location (e.g., centroid 中心点)
- Query Types
  - Spatial range queries
    - E.g., show sth in Ithaca
  - Nearest neighbor queries
    - E.g., show the nearest gas station
  - Spatial joins
    - E.g., show sth with sth within 100 m

## Index

### B+ tree

- input: (D1, D2, D3...)
  - ![[Screenshot 2024-11-30 at 7.56.43 PM.png|200]]
  - Index order: D1 最优先, 之后的用于 break tie
  - range queries 效果不稳定
- 1 B+ tree per dimension 建立一个 tree
  1.  Tree: 当前 dimension 信息 指向 对应的 record id
  2.  merge RIDs
  - 很难 maintain trees, merge 很贵

### Space-filling curves: Z-Ordering

- 用于保存 points
- **将多维坐标用二进制表示后按位交错**
  - 把 N-D 转换成 1D
  - **按交错后的结果进行排序**
  - `(a1 a2 a3.. an, b1 b2 b3.. bn)` for 2D
  - `a1 b1 a2 b2 ... an bn` for Z-Ordering
- Example
  - (1,0): 交错为 `01`, Z-order 值是 2
  - ![[Screenshot 2024-11-27 at 10.22.55 PM.png|200]]
    - 10 号点: x = 11, y = 00, binary = `1010`
    - 坐标顺序: `0000, 0001, 0010, 0011`
    - 对应位置: `(0,0),(0,1),(1,0),(1,1)`

### Region quad tree

- 用于保存 regions, 每个 partition 都是 independent
- Divide space recursively
  - In 2D: divide each region into four quadrants
  - Quadrants are associated with child nodes in tree
  - ![[Screenshot 2024-11-30 at 7.57.07 PM.png|200]]

1. root: 整个 tree
2. 把 root 分成 4 部分 as child
3. 分裂需要的 child 成 4 部分 as child
4. 一直分裂直到当前 node 不能再分裂 & 包含要找的 region - leaf node

### Grid files

- 用于寻找 region 的信息很 skewed
  - skewed: 有些很密, 有些很稀疏
- Denser area: More fine-grained representation
  - 分裂到比较小的 cells 确保精度
- Sparse area
  - 分裂的 cell 比较大

### R+ tree

![[Screenshot 2024-11-30 at 7.57.30 PM.png|200]]

- 搜索 Search key: multi-dimensional bounding box
  - bounding box: 搜索对象的范围 (Xmin, Ymin, ... , Xmax, Ymax, ...)
  - 找到范围内的所有 object
    - Object: single point (x, y) or region
    - object: Tree 的核心数据, 每个 object 都用一个 bounding box 描述范围
    - region: inner node (index entry) 或 覆盖的范围, 由 child 决定
- 结构
  - inner node = Index entries: (bounding box, pointer to child)
  - leaf node = Data entries: (bounding box, record id)

#### Lookups

1. Compute bounding box
2. Start at root node of R tree
3. Check children containing query object
   - result 有可能在多个 object: check multiple children

#### Insertions

1. Compute bounding box
2. Start at root node and proceed to leafs
3. Select child needing **minimal extension** for object
   - 找最高效的 insertion place - most overlap
4. Insert object at **leaf node**
   - May enlarge bounding boxes on path to leaf
   - May rebalance the tree

# SQL

## 1

```SQL
CREATE TABLE Students (studentID int PRIMARY KEY, studentName text);
CREATE TABLE Courses (courseID int PRIMARY KEY, courseName text);
CREATE TABLE Enrollments (courseID int, studentID int);

ALTER TABLE Enrollments ADD PRIMARY KEY (courseID, studentID);
ALTER TABLE Enrollments ADD FOREIGN KEY (studentID) REFERENCES Students(studentID);
ALTER TABLE Enrollments ADD FOREIGN KEY (courseID) REFERENCES Courses(courseID);
```

- Write an SQL query calculating the average number of enrollments per student (you can assume that the database stores at least one student). The query result contains one column and one row, containing the average. (5 points)

```SQL
select
(
	select count(*)
	from Enrollments
) / (
	select count(*)
	from Students
);
```

- Write an SQL query retrieving students with the courses they are enrolled for. The query result contains two columns: the student name and the course name. Each row represents one course enrollment. (5 points)

```SQL
select S.studentName, C.courseName
from Students as S
	join Enrollments as E on S.studentID = E.studentID
	join Courses as C on C.courseID = E.courseID
```

- Write an SQL query retrieving all students who are enrolled in the maximal number of courses (i.e., if multiple students have the maximal number of course enrollments, the query should return all of them). The query result has one single column for the student name. (5 points)

```SQL
select S.studentName
from Students as S
where not exists (
	select *
	from Students as S1
	where (
		select count(*)
		from Enrollments as E
		where E.studentID = S1.studentID
	) > (
		select count(*)
		from Enrollments as E
		where E.studentID = S.studentID
	)
)
```

- Retrieve students who are enrolled in all courses in the database. The query result contains two columns, one for the student ID and one for the student name. (5 points)

```SQL
select S.studentID, S.studentName
from Students as S
where not exists (
	select *
	from Courses as C
	where not exists (
		select *
		from Enrollments as E
		where C.courseID = E.courseID AND S.studentID = C.studentID
	)
);
```

## 2

```SQL
CREATE TABLE Rooms (roomID int PRIMARY KEY, floor int);
CREATE TABLE Guests (guestID int PRIMARY KEY, guestName text);
CREATE TABLE Reservations (guestID int, roomID int,startDate date, endDate date);

ALTER TABLE Reservations ADD PRIMARY KEY (guestID, roomID, startDate, endDate);
ALTER TABLE Reservations ADD FOREIGN KEY (roomID)REFERENCES Rooms(roomID);
ALTER TABLE Reservations ADD FOREIGN KEY (guestID)REFERENCES Guests(guestID);
```

1. Write an SQL query retrieving the room at which a guest named 'John Smith' stayed on January 1, 2023. The query result contains one single column with the room ID. You can assume that there is a suitable reservation in the database. (5 points)

```SQL
select R.roomID
from Rooms as R
	join Reservations as R1 on R.roomID = R1.roomID
	join Guests as G on G.guestID = R1.guestID
where G.guestName = 'John Smith'
	and R1.startDate <= 'January 1, 2023'
	and R1.endDate > 'January 1, 2023'
```

2. Write an SQL query calculating the number of rooms that are available (i.e., without associated reservation) as of today (December 9, 2023). The query result contains one column and one row, representing the number of rooms. (5 points)

```SQL
select count(*)
from Rooms as R
where not exists (
	select *
	from Reservations as R1
	where R1.roomID == R.roomID
		and R1.startDate <= 'December 9, 2023'
		and R1.endDate > 'December 9, 2023'
)
```

3. Write a query retrieving the names of guests who made at least two reservations. The query result contains one column with the guest names. (5 points)

```SQL
select G.guestName
from Guests as G
	join Reservations as R on G.guestID = R.guestID
group by R.guestID
having count(*) >= 2;
```

4. Write a query retrieving the ID of the guest who made most reservations. The query result contains one column with the guest ID. (5 points)

```SQL
select G.guestID
from Guests as G
where not exists (
	select *
	from Guests as G1
	where (
		select count(*)
		from Reservations as R
		where R.guestID = G1.guestID
		) > (
		select count(*)
		from Reservations as R
		where R.guestID = G.guestID
	)
)
```

# Prelim

1. Consider table T with 100 rows, consuming 10 pages, and a table U with a hash index on the join column. Calculate the cost of an index-nested loops join, using T as outer and U as inner table. We assume that the cost for each index access is composed of one page read for the index hash bucket page and one read for each matching entry. Assume that each row in the outer relation has 10 matching entries in the inner relation. Show your calculations to obtain partial credit. (5 points)
   - We count a cost of 10 to read the outer operand (T). For each row in T, we have a cost of 1 + 10, i.e. the total cost for index accesses is 100 \* 11 = 1,100. In total, we have a cost of 1,110.
2. We sort a data set with 10,000 pages, using the external sorting algorithm seen in class. We allocate 100 buffer pages in main memory for the sort. Calculate the cost for the sort operation. Show your calculations to obtain partial credit.
   - We sort data chunks of size 100 pages in the first pass over the data. The cost is 2 \* 10,000 = 20,000 page I/Os. After the first run, 100 sorted runs remain. In the second pass, we merge 100-1 = 99 sorted runs at once. As 100 runs are left, this means we need two more passes over the data. As the third pass generates the final result, we do not count the writing cost. Hence, the total cost is 50,000 page I/Os.
3. We use the external sort algorithm, seen in class, to sort a table with 90 pages. We use 10 buffer pages for the sort operator. Calculate the cost for sorting (assuming that the sort result is the final result). Show your calculations to obtain partial credit.
   - In the first pass over the data, we obtain sorted chunks of size 10. Using 10 buffer pages, we can merge 9 runs in each further pass. In the second pass, we can therefore merge all remaining runs. As we read and write data in each pass, the cost per pass is 2 \* 90 = 180. We have two passes. However, the cost of writing the final result is not counted. Hence, the total cost is 2 \* 180-90 = 270.
