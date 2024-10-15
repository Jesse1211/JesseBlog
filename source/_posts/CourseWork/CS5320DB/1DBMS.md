---
title: 1. DBMS
categories:
  - Course-work
  - CS-5320-Database
---

### Database Management System Highlevel

<!-- ![DBMS](/image/DBMS.jpg) -->

Manage data, allows applications to store, update and analyze the data and avoids Code, Data Overlap, Inconsistencies

- Layer 1: Query Processor (Operators, Cost Estimation, Query Optimization)
  - **Parser** - Check syntax and convert to Optimizer readable command, and check semantic
  - **Optimizer** - 优化器会生成许多能够满足查询请求的访问计划, 并选择具有最小成本的方案作为最终的访问计划 (Acess Plan), 通过 Estimator, Plan Generator
  - **Rewriter** - Convert to higher performance command
  - **Executor** - Execute
- Layer 2: Storage Manager (Storage Media)
  - **Data Access** - Handles access to the data stored in various storage media.
  - **Buffer Manager** - temporarily store data during processing.
  - **Indexes, Filed, Data Layout** - Organizes data and indexes in storage for efficient retrieval.
- Layer 3: Transaction Processing
  - **Concurrency Control** - Manages concurrent access to the database to ensure consistency.
  - **Crash Recovery** - Restores the system to a consistent state in the event of a crash.
  - **Transaction Manager** - Ensures that all database transactions are processed correctly and ensures ACID properties.
  - Recovery Manager
- Layer 4: Data
  - **Schema Design** - Designs the database schema and ensures efficient organization of data.
  - **Detecting Redundancy** - Identifies and eliminates redundancy in the database schema.
  - **Schema Normalization** - Normalizes the database schema to avoid anomalies.
  - **Distributed Process (NoSQL and NewSQL)** - Handles data distribution across multiple nodes, especially in distributed and NoSQL databases
  - **Graph Data, Data Streams, Spatial Data** - Manages specialized data types like graph data, streaming data, and spatial data.
