---
title: 7. Operators
categories:
  - Course-work
  - CS-5320-Database
---

# Terms

### Query Plans

- Describe query processing as **tree** about how to generate required data
  - Leaf node: database table
  - Inner node: operation
  - Tree edges: data flow
- **Logical query plan**: specifies types of operations
- **Physical query plan**: specifies implementation

### Operators

- Use fixed set of standard operators
- Consumes relation(s) and produces one relation
- **Filter operator (𝛔)**: select rows conditionally
- **Projection operator (𝛑)**: select columns conditionally
- **Join operator (⨝)**: find matching tuple pairs

### Relational Algebra

- Describe query plans as "mathematical expression"
- Evaluated from inner to outer expressions

### Passing Results Between Operators

- Version 1: each operator writes output to **disk**
  - Unnecessary read/write overheads
- Version 2: keep all intermediate results in **main memory**
  - Feasibility, depends on size
- Version 3: Pipelined operator passes result **in-memory** to next operation
  - Physical plan specifies how results are passed on
  - Label "On the fly" for unary operators means pipelined input
    - output is immediately consumed by the next without any delay or intermediate storage.

### Pipelined Nested Loop Joins

- Full join output: too large to fit in memory
- Partitioning the Join Output: Produce small join result parts consecutively
- Directly invoke next operator for result part in memory
- Chain nested loop joins by producing output with part of left input
- example
  - Table A has 100 rows, Table B has 200 rows
  - `A join B`
  - for each 10 rows in A, join 200 rows in B
  - pass the joined 2000 rows to next operator (such as count())
  - go for next 10 rows in A...

### Cost

- (Calculate intermediate result sizes if not given)
- Cost of each operator
  - Take into account how data is passed on
- Do not count output cost of final operator
- Sum up cost of all operators in plan

### Cost Example

`SELECT * FROM Enrollment WHERE CID='CS4320'`

- studentNum = 60,000
- enrollPerStuent = 10
- sizePerEnrollEntry = 10 Bytes
- bytesPerPage = 1,000 Bytes
- entryPerPage = sizePerEnrollEntry / bytesPerPage = 1000 / 10 = 100
- enrollPage =
  - = enrollPerStuent \* studentNum / entryPerPage
  - = 10 \* 60,000 / 1,00 = 6,000
- (non indexed) Total Scan Cost = enrollPage = 6,000
- (Clustered indexed) Total Scan Cost
  - = innerNodeVisite + leafNodeNum
  - = (treeHeight - 1) + enrollPage / entryPerPage
  - = 2 + 6,000 / 100
  - = 2 + 60
  - = 62
- (Unclustered indexed) Total Scan Cost
  - = innerNodeVisite + leafNodeNum + enrollPage
  - = 62 + 6000 = 6062

# Operators

## Filter Operator $\sigma$

- Fetch table rows for **predicate**
- Default: **scan** all pages, check each entry
- Faster (not always):
  - Sorted: **binary search** for specific predicates
  - Indexed: right table, right key
- Efficiency depends on **query optimizer**

## Join Operator ⨝

- one of the **most expensive** operations
- Some are **more generic** and apply to any join predicate
- Some are **faster** in specific situations
- Some need **less memory** than others
- Notations
  - **LoadPage**(P): Load page P
  - **Pages**(T): Pages from table T
  - **Tuples**(P): Tuples from page P
  - **PageBlocks**(T, b): Blocks of b pages from T
  - **LoadPages**(B): Load pages from block B
  - **Index** (Predicate): Entries satisfying predicate
  - **Tuple** (P, i): i-th tuple on page P

### Page Nested Loop Join (first table = **outer table**)

```
⨝E.Sid = S.Sid

For ep in Pages(P1):
	LoadPage(ep) // Load each page in P1
	For sp in Pages(P2):
		LoadPage(sp) // Load each page in P2
		For et in Tuples(sp), st in Tuples(sp):
			// Compare every line
			if (et.Sid = st.Sid):
				Output(et⨝st)
```

- Read inner table for each outer **page**
- Cost = pageNum1 \* load cost +
  - pageNum1 ** pageNum2 ** load cost +
  - pageNum1 ** pageNum2 ** evaluation cost
- for (Load ALL pages from **first table**):
  - for (Load ALL pages from **second table**):
    - For all tuples in memory: **check** and **add** to result

#### Memory (1 + 1 + 1 = 3)

- Store current **page from outer table**: 1
- Store **current page from inner table**: 1
- Need **one buffer page** to store output (before disk write): 1
  - For the intermediate result

### Block Nested Loop Join (improved)

```
⨝E.Sid = S.Sid

For ep in PageBlocks(P1, b):
	LoadPage(ep) // each page in P1's block
	For sp in Pages(P2):
		LoadPage(sp) // load each page in P2
		For et in Tuples(sp), st in Tuples(sp):
			if (et.Sid = st.Sid):
				Output(et⨝st)
```

- Cost = pageNum1 \* load cost +
  - **blockNum1** ** pageNum2 ** load cost
- Read inner table for each outer **block**
  - Block == **multiple** pages

#### Memory (B + 1 + 1 = B + 2)

- Space to store **blocks from outer relation**: B
- Space to store **one page from inner** relation: 1
- Need one **page to store output** (before writing to disk): 1

### Index Nested Loop Join

```
For ep in Pages(P1):
	LoadPage(ep) // Load each page in P1
	For et in Tuples(ep):
		For <sp, i> in Index(et.Sid=st.Sid): // No need to load all due to index
			LoadPage(sp)
			Output(et ⨝ Tuple(sp, i))
```

- Cost = pageNum1 \* load cost +
  - indexEntryNum2 \* load cost
- Idea: have **index on join column** and **equality predicate**
- Iterate over pages from non-indexed (outer) table
- For each outer tuple, **use index** to find matching tuples

#### Memory (1 + 1 + 1 = 3)

• Store current **page from outer table**: 1
• Store **current page from inner table**: 1

- **output buffer** (before disk write): 1

### Equality Joins - Hash Join (Alternatives for Equality Joins)

- Example
  - ![[Screenshot 2024-09-25 at 1.43.47 PM.png|300]]
- Phase 1
  - **Partition data** by hash values in join columns (partition by even and odd value)
  - **Read** data + **write** partitioned data
    - cost = 2 \* (pageNum1 + pageNum2)
- Phase 2
  - Join **each partition** pair (same hash value)
    - Read data (NO Write)
    - cost = 1 \* (pageNum1 + pageNum2)
- cost = 3 \* (pageNum1 + pageNum2)

#### Memory

- Phase 1:
  - 1 + # buckets
- Phase 2:
  - 2 + # Pages / # buckets
- **Bucket num = $\sqrt(\text{smaller table Page num})$ **
- If Lack Memory
  - Buffer pages size limits number of **output buckets**
  - Less buckets -> more data in each bucket
    - Prevents **loading one bucket** entirely in Phase 2
  - Perform **multiple passes** over data in phase 1
    - Partition buckets into **sub-buckets**
    - Iterate until data per bucket **fits** into main memory

### Equality Joins - Sort-Merge Join (SMJ) (Sub-optimal in memory)

- Phase 1 (Sort)
  - **Sort** the join column (single entries) in joined tables
    - **inefficient** to random data access
    - Access **pages** of entries instead to fit into memory
- Phase 2 (Merge)
  - **Load first page** of both sorted tables into memory
  - **Find matching tuples** and add to join result output
  - **Load next page** for table with smallest last entry
  - **Keep doing** until no pages left for one table

#### Algorithm

- B: buffer pages available
- run: A sorted sequence of data = a chunk of data in memory

1. load & sort & write to disk (runs = pageNum / bufferSize)
   1. Load: chunks of B pages into the buffer
   2. sort: each chunk
   3. write: sorted data to hard disk
2. merge B-1 sorted runs into 1 in one step: produce larger runs & less number of runs
   1. Load first page of each sorted run into B-1 pages
   2. Copy minimum entry in input buffers to output buffer
      - If output buffer full, write to disk and clear
   3. Erase minimum entry from input buffer
      - If input buffer becomes empty, load next page
3. one sorted run left

#### Cost

- Two input tables with M and N pages, B buffer pages
- Phase 1
  - 2 ** M ** ($1 + Ceil(log**{B-1}(N/B))$) for sorting table 1
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

#### Memory

- Phase 1: use all buffer pages
  - More buffer == less merging passes
- Phase 2: exploit 3 buffer pages
  - 3 for 1st input & 2nd input & output

### Refined Sorted-Merge Join (Refined SMJ)

- Idea: save steps by merging more than 2 sorted tables (No sorting tables completely in phase 1)
- Assume B buffer pages, tables with N and M pages
- Phase 1:
  - Only sort data chunks that fit into memory
  - load chunks of B pages & sort & write back
- Phase 2: merge ALL sorted tables
  - Join all sorted chunks together (one step)
  - merge B-1 sorted chunks together
- Cost:
  - **2 \* (M+N)** (Phase 1) **+ 1 \* (M+N)** (Phase 2)

#### Memory

- First phase:
  - **(N+M)/B** sorted runs
- Merge in one step:
  - **B ≥ 1+(N+M)/B**
- Rule of thumb:
  - if N>M: need **B ≥ 2 \* √(N)**

### R-SMJ vs Hash Join

![[Screenshot 2024-09-25 at 10.24.37 AM.png||300]]

## Projection $\pi$

- **SELECT without DISTINCT**
  - Calculate SELECT items, drop other columns
- If DISTINCT
  - Filter out duplicates by hash function / sorting / index

### Projection & Duplicate Elimination via Hashing

- Phase 1: partition data into hash buckets
  - Scan input, calculate projection, partition by hash function
  - Data partitions: write to hard disk (data size exceeds memory)
  - cost: **read input (1 ** num of pages) + write partitioned pages (1 ** num of pages)**
- Phase 2: eliminate duplicates for each partition
  - Read one partition into memory and eliminate duplicates
  - Can use second hash function to detect duplicates
  - cost: **read partition pages (1 ** num of pages) + write duplicated pages (1 ** num of pages)**
- Constraints on memory similar as for hash join
  - Count hash buckets for Phase 1, bucket size for Phase 2
  - **Bucket size may not fit in memory**
- Max Cost = **3 \* Number of Pages**

### Projection & Duplicate Elimination via Sorting

- Max Cost: **external sorting cost**
- Idea: sorting rows to find duplicates
- Variant of external sort algorithm
  1.  Apply projection during first pass over data
  2.  Eliminate in-memory duplicates during all steps
  3.  Now duplicate-free and sorted
  4.  Reduce number of passes with more main memory

### Projection & Duplicate Elimination via Index

- Max Cost: **reading index data**
- Retrieve relevant data by index
  - Assume index key includes projection columns (可以通过 index 找到 projection 里面的 attribute)
  - Saves cost considering index smaller than data
- Even better: tree index with projections as **key prefix**
  - Duplicates retrieved consecutively, easy to eliminate

## Grouping ($\Gamma$) & Aggregation ($\Sigma$ ...)

### Aggregation without Groups

- SQL offers Min, Max, Sum, Count, Avg
- Scan input data and update in-memory aggregate
  - Use constant amount of memory
  - Cost of reading input data once
- Count distinct requires duplicate elimination (as above - Projection with DISTINCT)

### Aggregation with Groups

group keys: the attribute inside GROUP BY

- hashing
  - Maintain hash table of group keys with aggregates
- sorting
  - Sort on group keys, aggregate groups consecutively
- indexes
  - Index key must contain group-by keys

## Set Operations (∩,∪,-)

- **INTERSECT** by JOIN
  - `SELECT R.id, R.value FROM R JOIN S ON R.id = S.id AND R.value = S.value;`
  - Join equality condition on all columns
- **UNION** eliminates duplicates (unlike UNION ALL)
  - `SELECT id, value FROM R UNION SELECT id, value FROM S;`
  - **Hash & eliminate duplicates** in each bucket
    - Hash the rows from both tables into buckets based on the columns.
    - Eliminate duplicates in each bucket during insertion (if a row is already in the bucket, do not insert it again).
  - **Sort & eliminate duplicates during merging**
    - Sort the combined data from both `R` and `S` on the columns (`id`, `value`).
    - Eliminate duplicates by scanning the sorted result and skipping over consecutive duplicates.
- **R EXCEPT S**
  - `SELECT id, value FROM R EXCEPT SELECT id, value FROM S;`
  - **Partition via hash**, then treat each bucket separately
    - Check if hashed R row exists in other bucket of S
  - **Sort** and check whether R tuple in S during merge steps
