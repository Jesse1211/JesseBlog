---
title: 5. Indexes Tress & Hash
categories:
  - Course-work
  - CS-5320-Database
---

## Definition

- Binary search
- B+ tree indexes
  - Average **fanout** (i.e., number of child nodes) is 133
  - traverse to find interesting leaves
  - Handle **equality and inequality**
    - Indexes in **sort order**
    - Consecutive keys stored close together
  - Composite keys: useful for conditions on key **prefix**
    - Keys with same prefix value stored close together
- Hash index: evaluate hash function to find buckets (key hash values)
  - buckets: a storage location which f(entries) are same
  - Handle **equality**
    - Consecutive keys might not close
    - Similar hash value $\neq$ similar key value
  - Composite Keys: Condition must constrain **all components** instead of prefix
    - Keys with **same prefix** may be stored far apart
- `CREATE INDEX <index-name> ON <table> USING <method> (<column-list>)`

### Index

- Index == **references** to data records
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

# Tree indexes

- R - Record Pointer (Reference)
  - Points to P for more entries
- P - Page as a Node
  - contains K and R
- K - Key
  - Indexed value, **ordered**
  - K(i): search key values. help define the boundaries or the ranges of data that each reference (R(i)) covers.
  - Holly, Olivia: R points to different P

#### Inner Nodes - R(0), K(1), R(1), K(2),...

- Any entries R(i) points to will be smaller than K(i+1) but larger than K(i).

#### Leaf Nodes: K(1), R(1), K(2),...

- Contains actual data references
- K(i): the key values related to the data records.
- R(i): point to **{P: data pages, specific slots}** (retrieving to the physical location of the data)

#### Example Usage (works if predicate references index key)

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

#### Linking leaf Nodes (B+)

- Leaf pages is **doubly linked list**
  - pointer to next/previous neighbor in leaf

### Composite keys

- **multiple columns** $\in$ Index search key
- **sort order** for key comparisons
  - `(a, b, c)`: a first, then b, then c
- Can use index for (in)equalities on **prefix** of key columns
  - Restriction to Key Prefix: **Cannot skip first column(s)**

### Postgres SQL Query

- `CREATE INDEX <index-name> on <table> (<columns>)`
  - Creates index for table using specified search key
  - Refer to index later via `<index-name>`
  - `<columns>` is comma-separated column list (key)
- `DROP INDEX <index-name>`
  - Delete index with given name

### Clustered index

**index stores data** instead of references to data - Actual table rows are organized (physically sorted) by index in physical storage.

- **at most one** clustered index per table
- More efficient as it saves chasing **one reference**
- **collocates** data with same key together (such as OrderData)
  - faster data retrieval for **Range, Ordered queries**
  - data pages are sorted on the same index on building B+ tree.
  - NOT sorted exactly, **keys** are roughly in the same order as data.
  - Two records with close keys will likely be in the same page.
- Cost: ~ 1 I/O per record

### UnClustered Index

- read a separate page for each of the records
- if we want to read adjacent records
  - read each of data pages they point to
- Cost: ~ 1 I/O per page of records

### Tree Variants B+: handling balance when update

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

# Hash Index

### Static hashing (bucket pages)

- Bad for dynamic data
- Hash **bucket** pages contain references to data (OR data directly)
- Hash buckets are associated with **hash value ranges**
- Can use hash index to find entries with key V
  - Calculate hash value h for V as h(V)
  - **Look up bucket page associated with h**

#### Update

- Deletions
  - remove associated entries
- Insertions
  - Can add "overflow" pages (like ISAM index!)
  - Initial bucket page stores pointer to first overflow page
  - Overflow pages form **linked list** if more than one
- **Rehash** if number of overflow pages increases
  - Create a new, larger hash table from the old table using different hash function.

#### Pros & Cons

- Get data with one read
- May need multiple reads (**> O(1)**) in case of overflow pages
- Will waste space if too many deletions (empty pages)
  - does not adjust number of buckets dynamically
- Can use rehashing but creates significant overheads
  - Create a new table
  - Recomputing
  - Moving

### Extendible hashing (directory $\to$ buckets $\to$ pages)

- More flexible than using page IDs directly
  - Directory: store addresses of the buckets in pointers
  - Buckets: store hashed keys
- Expands with few high-overhead operations
- Redistribute overflowing buckets to multiple pages
  - **Bucket split & directory expansion**
  - Increase directory size if too many splits
  - Split Bucket to 2 parts

#### Insertions

1. Calculate hash value for key of new entry
2. Consult directory to identify current bucket
3. insert
   - If bucket has space: Good
   - If overflowing:
     - Add new bucket page, rehash existing and new entry
       - For rehashing: consider one more bit of hash value
       - Expand directory if it does not consider enough bits

#### Terminology

- Global depth: num of **hash bits in directory**
  - hash function to categorize the keys
  - Max # of bits needed to tell which bucket an entry belongs to.
- Local depth: **num hash bits in bucket**
  - The # bits used to determine if an entry belongs to this bucket.
  - to decide if overflow occurs (need to split)
    - Before insert, local depth = global depth.
    - Insert causes local depth to become > global depth; directory is doubled by copying it over and **fixing** pointer to split image page.
- 当 bucket 满了以后, trigger doubling Global Depth 扩大 Directory, 然后回过头扩大 bucket 的 local depth 并且 split.

#### Deletions

- Merge bucket pages if they become empty
- Half directory size if number of buckets shrinks
- Often no compaction in practice
  - Assumption: inserts are more common than deletes

#### Pros & Cons

- Avoids overflow pages
- No expensive rehashing
  - Only rehash one bucket at a time
- Need additional directory access
- Need to double directory occasionally
  - This may take up some time

### Linear hashing (Avoid overflow pages)

- Expands more "smoothly"
- Idea: avoid using directory by fixing next bucket to split
  - Not always split overflowing bucket
    - I.e., have temporary overflow pages
  - Buckets to split are selected in round robin fashion
    - Overflowing bucket will be split eventually

#### Insertion

- Calculate hash value for new entry to insert
- Add entry on page / overflow page
- **Split next bucket if trigger condition is satisfied**
  - May eliminate previously generated overflow pages
  - Some flexibility in choice of trigger condition

#### Splitting

- Splitting proceeds in rounds: **round-robin order**
  - All buckets present at round start split → round ends
  - "Next Split" pointer is reset to first page at round end
  - Splitting regardless of whether the bucket is full, empty, or overflowing
- Split the bucket pointed to by "Next Split"
  - Add one new page, redistribute split bucket entries
    - Consider one more bit when redistributing

#### Pros & Cons

- Avoids a directory - no expensive directory doubling
- May temporarily admit overflow pages
- May split empty pages - inefficient space utilization

### Optimizations

- Can apply same optimizations as for tree indexes
- Have many entries for same search key value?
  - Store key value, followed by list of references
- Want to get rid of one level of indirection?
  - Can store data directly instead of references
  - Leads to "clustered index", only one per table!
