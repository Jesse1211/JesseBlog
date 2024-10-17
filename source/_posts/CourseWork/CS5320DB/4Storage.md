---
title: 4. Data Storage
categories:
  - Course-work
  - CS-5320-Database
---

- Decomposing **tables** into pages, **pages** into slots, **slots** into fields
- Variable length content can be handled via **directories**

## Hardware - Storage

### Relevance for DBMS

- **Capacity limits** force data to lower parts of hierarchy
- Data **access speed** may become bottleneck
- Design algorithms to **minimize data movements**
- **Random** data access is expensive
- Read data in larger **chunks ("pages")**
- Keep related data **close** together
- Take into account volatility for **recovery** considerations

### Memory Hierarchy:

**(Volatile):** Registers → CPU Cache → Main Memory → **(Non-Volatile):** Flash/USB Memory → Hard Disk → Tape Backup  
**Faster Access →**  
**Lower Capacity →**

### Tape Storage

- Bits as **magnetic** information on tape
- **Very slow access** (10s of seconds)
- **Moderate read speed** (up to 300 MB/second)
- **Very cheap** (around $0.02 per Gigabyte)
- Used for long-term **archival** (e.g., by Google)

### Hard Disk

Bits as **magnetic** information on platter

- Patters **spin** under read/write heads
- **Slow access** (10s of milliseconds access time)
- **Moderate read speed** (around 200 MB/second)
- **Cheap** (around $0.035 per Gigabyte)
- Used for **less frequently accessed** data

### Solid State Drives SSD

- Bits as small **electric charges**
- **Elevated price** (around $0.25 per Gigabyte)
- **Fast access** (around 1 millisecond)
- **Elevated speed** (around 500 MB/second)
- Limited number of write cycles (**memory wear**)

### Main Memory

- Bits as small **electric** charges
- **Expensive** (several dollars per Gigabyte)
- **Very fast access** (order of nanoseconds)
- **High bandwidth** (Gigabytes per second)
- Used to access **hot** data - **all** if economically feasible!

### Cache

- Bits as small **electric** charges
- Typically organized as cache **hierarchy**
- **Very expensive** (hundreds of dollars per Gigabyte)
- **Near-instantaneous** access (few nanoseconds)
- **Very high bandwidth** (tens of Gigabytes per second)
- Used to store **immediately relevant** data

## Format - relation representation

### Tables as Files

- Table schema === database catalog; 每个 page 的一个 row
- Table content $\in$ **collection of pages** called file
- page
  - a few KB for each page
  - might not enough for entire table (only multiple rows)

### Files to Pages

1. Store pages as **doubly linked list**
   - each data page contains **records**, a **free space tracker**, and **pointers** (byte offsets) to the next and previous page.
   - header page - header reference in DB catalog
     - start of the file and separates the data pages into full pages and free pages.
     - When space is needed, empty pages are allocated and appended to the free pages portion of the list.
     - When free data pages become full, move from the free space portion to the front of the full pages portion of the linked list.
2. **Directory** with pointers to pages
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

# Questions

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
