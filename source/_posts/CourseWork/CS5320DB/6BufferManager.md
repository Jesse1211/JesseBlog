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
  - **frames**: Divided into page-sized slots
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
  - Read requested page from disk and store in frame
  - Increase pin count and return page address

### LRU

- Replace page required farthest in the future
  - Reduce expensive cache misses
- However: difficult to predict that in general
- Heuristic: remove least recently used page (LRU)
  - remove use page which did not use for long time

### Sequential Flooding (when LRU is no good)

- DBMS often have particular access patterns
  - e.g. Scanning pages in round robin mode (circular or repeated sequence)
- Least recently used page is used again soonest
  - LRU policy gets sub-optimal (inefficient)
- ==Otherwise a reasonable strategy!==
