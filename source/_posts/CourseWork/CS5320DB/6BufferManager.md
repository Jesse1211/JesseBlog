---
title: 6. Processing Overview (Buffer Manager)
categories:
  - Course-work
  - CS-5320-Database
---

- Decides when to move data between disk and RAM
- Reduce data movements using heuristics
- Buffer manager manages "buffer pool"
  - Buffer pool: main memory reserved for DBMS
  - **frames**: page-sized slots
  - Stores meta-data about each slot

## Frame Properties

- Page ID:
  - Find page, then frame
- Pin count: # of processes using the page
  - Evict page if pin count reaches zero
- Dirty bit: memory data != storage data (updated data)
  - Write page to disk before evicting it

## Don't rely on OS

- OS uses virtual memory caches pages
- _We want a separate buffer manager_
  - DBMS knows its access patterns ahead of time
    - Smarter replacements
  - DBMS must control page writes for safety guarantees

## Processing Page Requests (buffer manager)

- Cache Hit (requested page cached)
  - Increase pin count & return page address
- Cache Miss (requested page not cached)
  - Choose frame for storage data (replacement policy)
  - If frame contains dirty page: write to disk
    - Increase pin count and return page address
  - Read requested page from disk and store in frame, Set pint count to 0

### LRU

- Replace page required farthest in the future
  - Reduce expensive cache misses
- However: difficult to predict that in general
- Heuristic: remove least recently used page (LRU)
  - remove use page which did not use for long time

### Sequential Flooding (when LRU is no good)

- buffer pool’s contents are corrupted due to a sequential scan.
- Since sequential scans read every page, the timestamps of pages read may not reflect which pages we actually want.
  - the most recently used page is actually the most unneeded page.
- DBMS often have particular access patterns
  - e.g. Scanning pages in round robin mode (circular or repeated sequence)
- Least recently used page is used again soonest
  - LRU policy gets sub-optimal (inefficient)
