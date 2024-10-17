---
title: 9. Transaction Manager
categories:
  - Course-work
  - CS-5320-Database
---

- 访问(并更新)数据库的一个程序执行**单元**
- **Transaction**
  - A sequence of updates (or queries) that is "connected"
- **Two operations may conflict if**
  - They are by different transactions
  - They are on the same object, and one of them is a write
- DBMS commands for assigning queries to transactions
- DBMS makes guarantees about transactions
  - execute **all transactions or none**
- Example
  - Wire $50 from Alice to Bob
    1. Deduct $50 from Alice's account
    2. Add $50 to Bob's account
  - Semantically related - execute both or none
- Postgres operations
  - Begin a transaction with command **BEGIN;**
  - End a transaction with command **COMMIT;** or **ABORT;**
  - Everything in between belongs to **transaction**
- Locks
  - Multiple transactions can read objects without conflicts
  - **Read (aka shared) locks**: read access
  - **Write (aka exclusive) locks**: read+write access
  - Lock manager may grant **multiple read locks** on same object

## ACID guarantees

Transactions guarantee the **ACID properties** to avoid the problems

### A: Atomicity (either execute all or nothing)

- Execute All or none
  - Transaction ends in **commits** or **aborts**
- Uncertainties:
  - crash by bugs or power failure
  - Transaction step violates integrity constraints
  - Explicit transaction **abort** (e.g., PG: ROLLBACK;)
  - Internally, DBMS executes steps **separately**
  - May have to do **cleanup** in case of partial execution
- Ensure Atomicity
  - **Logging:**
    - Log all details of a transaction
    - Undo transactions if any of them aborts
    - Maintain these records in memory and on disk
  - shadow paging

### C: Consistency (enforce all integrity constraints)

- 数据库从一致性状态到另一个一致性状态
- Data is consistent with all constraints
  - **Primary key** constraints
  - Referential integrity (**foreign key** constraints)
  - More **complex constraints** can be defined
- DBMS will **abort** transactions threatening consistency
- **Careful**, differs from distributed systems consistency

### I: Isolation (avoid interleaving transactions badly)

- transaction 之间数据隔离
- Execution of each transaction is isolated from that of others.
- **simulating** sequential execution to users
- Different users may execute transactions **concurrently**
  - E.g., bank has many clients transferring money
- Executing transactions sequentially is **inefficient**
  - Clients wait until one transaction is done
- **interleave** steps from multiple transactions

#### Interleave Steps

- Motivation 1: **long running** transactions
  - Long wait if behind long transaction
  - Better: **alternate** between (long & short) transaction steps
  - Long transaction barely slower, short transaction quick
- Motivation 2: **idle time** e.g. by disk access
  - Assume transaction 1 **needs data** from disk for next step
  - Could load data while executing step from **other** transaction
- Notations
  - C: commits
  - A: aborts
  - RW: read & write
  - number: transaction
  - letter: object

### D: Durability (ensure that updates are not lost)

- Committed updates are **persist**
- DBMS notifies users that transaction has **committed**
- This must hold even if DBMS or server **crashes**
- Must store enough info on disk to **restore** at startup
- Only notify user of commit **after** that has happened

## Isolation Anomaly (if violate ACID)

- may destroy illusion of sequential execution
- **Write-Read Conflict (W-R)**
  - **Dirty Reads**: reads data written by not committed transaction
  - User 1 updates a toy’s price but this gets aborted.
  - User 2 reads the update before it was rolled back.
- **Read-Write Conflict (R-W)**
  - **Unrepeatable Reads**: Transaction gets a different value, when reading it multiple times
  - A user reads two different values for the same record because another user updated the record in between the two reads.
- **Read-Read Conflict (R-R)**
- **Write-Write Conflict (W-W)**
  - **Lost Updates**: Overwrites uncommitted data from another uncommitted transaction
  - Two users try to update the same record at the same time so one of the updates gets lost.
- **Phantom Read (幻读)**
  - 在同一个事务中，由于其他事务的提交，执行同一查询时可能会返回之前未返回的数据记录。这通常发生在以下情况：事务在开始时读取了一组数据，但在执行过程中，其他事务插入了新的数据记录，导致原始事务再次执行查询时返回了新的数据记录。
- Isolation Level
  - **Read uncommitted**: 读取数据时，不需要等待其他事务提交。
  - **Read committed**: 事务只能读取那些其他事务已经提交的数据变更。
  - **Repeatable Read**: 事务在开始时生成一个快照，事务执行期间只基于这个快照读取数据。这意味着，事务在整个执行过程中看到的都是同一个数据状态，无论其他事务在此期间进行了多少次修改和提交。
  - **Serializable**: 所有读取操作都是串行执行的

### Isolation in Postgres

Setting default isolation level for future transactions:

- **Default is READ COMMITTED**
- Set session characteristics as `<isolation-spec>`
- Setting isolation level for the current transaction:
  - Set transaction `<isolation-spec>`
- `<isolation-spec> ::= isolation level <i-level>`
- `<i-level>` is one of SERIALIZABLE, REPEATABLE READ, READ COMMITTED, READ UNCOMMITTED

## Concurrency Control (Analyze transaction schedules)

Select cheapest schedule among good ones & minimize selection **overheads**

- **Schedule**: ordered steps from multiple transactions
- A **good schedule** preserves the illusion of isolation
  - E.g., none of aforementioned anomalies
- Properties
  - **Serializability**
    - Final state serializable (Anomalies)
    - Conflict serializable (Disallows some good schedules)
    - View serializable (slow concurrency control)
    - 2PL (produces conflict serializable schedules)
  - **Aborts**
    - Recoverable
    - Avoids cascading aborts
    - Strict

### Comparing Schedules (equivalence)

Define good schedules by **comparison** with Serial schedule on certain criterion

- **Serial** schedule
  - one transaction after the other, no interleave
- **Final state** equivalence

  - **Compare** two schedules based on final database state
  - Equivalent schedules if DB content **equal** after execution
  - Must hold for arbitrary **initial** database content
  - equivalent example
    - W1(A) W2(A) W1(B) W2(B) C1 C2
    - W1(A) W1(B) C1 W2(A) W2(B) C2

- **View** equivalence (stronger than final state equivalence)

  - Same read instruction & final state
  - Same initial reads
    - If transaction X reads the **initial value** for some object in S1, it also does so in S2
  - Same dependent reads
    - If transaction X reads a **value written** by transaction Y in S1, it also does so in S2
  - Same winning writes
    - If transaction X writes the **final value** written by transaction Y in S1, it also does so in S2

- **Conflict** equivalence - When two schedules order their conflicting operations in the same way (stronger than View state equivalence)
  - The operations are from different transactions
  - Both operations operate on the same resource & At least one operation is a write
  - **Can get from S1 to S2 by swapping non-conflicting operations**
    - 每一对冲突的操作在两个调度中的顺序都相同

### Comparing Schedules (Serializability - criterions):

- **Final State Serializability:**
  - A schedule S is **final state serializable** if
    - There is a **serial** schedule ...
    - ... that is final state **equivalent** to S.
  - May have **unrepeatable reads** with final state serializability
    - Can be bad even if it does not influence db state
    - E.g., R1(A) W2(A) R1(A) is final state serializable
- **View Serializability:**
  - NOT Ensure Conflict Serializability
    - Possible W-W conflicts does not affect view serializability but affects conflict serializability
  - R1(A) W2(A) R1(A) C1 C2
    - R1(A) R1(A) C1 W2(A) C2 - not view equivalent as 2nd read now returns initial value
    - W2(A) C2 R1(A) R1(A) C1 - not view equivalent as 1st read does not return initial value
    - **Not equivalent** to any of two possible serial schedules
  - Verifying view serializability is **NP-hard**
- **Conflict Serializability**
  - Ensure View Serializability
    - Output of a schedule depends only on conflicting actions.
    - Swapping non-conflicting actions does not change the output
  - Can test efficiently if schedule is conflict serializable by
    - Draw conflict graph
    - Test if conflict graph has cycle
    - Conflict serializable if no cycle

### Conflict Graph (acyclic graph)

- A schedule is conflict serializable if and only if its dependency graph is acyclic.
- One node per transaction in schedule
- Edge from Ti to Tj if
  - conflicting operations O1 and O2
  - O1 appears earlier in schedule than O2
  - Draw edge from O1 transaction to O2 transaction
- Semantics of edge from i to j
  - Any conflict-equivalent schedule must order i before j
- Example
  - `T1: R(A), W(A), ---------------------, R(B)`
  - `T2: ----------, R(A), W(A), R(B), W(B),---`
  - T1 reads A and then T2 writes to A: there will be an edge from T1 to T2.
  - T1 writes to A and then T2 reads from A. we don’t have to add T1 to T2 again.
  - T2 writes to B and then T1 reads B: there will be an edge from T2 to T1.
- Convert to equivalent serial schedule
  - Start from the node w/o incoming edges
  - Add all operations of that transaction and commit
  - Continue with node where all predecessors treated

### Handling Aborts

- Exclude aborted transactions for checking serializability
  - DBMS acts as if aborted transactions never happened
- Orthogonal classification of schedules based on aborts

#### Schedules

##### Recoverable Schedules

- Transaction commits only **after** all transactions it read from have committed as well
  - transactions can be rolled back safely.
- If Ti reads data which was written by Tj, then Tj must commit before Ti reads
- Counter-Example:
  - W1(A) R2(A) W2(B) C2 A1
  - No trace of aborted transactions should remain
  - But write to B may have been influenced by read from A

##### ACA Schedules

- No transaction reads uncommitted data
  - reducing the risk of widespread rollbacks.
- Avoid rolling back 1 transaction causes rolling back others
- Schedule recoverable by delaying commits
- May have chain of aborting transactions
  - Transaction read from aborted transaction - tainted!
- Recoverable but does not avoid cascading aborts:
  - W1(A) R2(A) W2(B) C1 C2

##### Strict Schedules

- No transaction reads or writes uncommitted data
  - highest level of isolation and make recovery simpler and more reliable.
- E.g., W1(A) W2(A) W3(A) not strict (ACA & recoverable)
- E.g., W1(A) C1 R2(A) W2(B) C2 strict

## Concurrency Control Protocols - Lock

Use Locks to protect database objects

- Types
  - **Fine-grained** locks are better
    - increase degree of parallelism
    - increases locking overheads
  - **Shared and Exclusive Locks:** Can perform multiple reads in parallel
  - **Release Early:** Allows parallelism
  - **Acquire Late:** Allows parallelism

### Lock based Concurrency ControlLocks (CC)

- Locks can have different types, be one any object

- one lock per **database**
- Request lock at **transactions start**
- Release lock at **transaction end**
- **Only one** transaction can hold at the same time

### Refined Lock Granularity

- Transactions on **different objects** in parallel
- Enable by locking **specific DB objects** (instead of DB)
- Locking **protocol** summary:
  - Transaction requests locks on all its objects **at start**
  - **Waits** until all locks have been granted
  - Transaction executes and releases locks **at end**

### Release Locks Early

- **Releasing locks earlier** may increase parallelism
  - Release lock **after last operation** on associated object
- lead to **cascading aborts**
  - W1(A) **[Lock on A from 1 → 2]** R2(A) A1

### Acquire Locks Late

- Acquire locks **directly before** read or write operation
  - (So far: acquired all locks at transaction start)
- May improve performance by **increasing parallelism**
- May however lead to **deadlocks**:
  - Transaction 1 acquires **lock on A**, now **waiting for B**
  - Transaction 2 acquires **lock on B**, now **waiting for A**

### Two-Phase Locking (Ensure conflict serializable schedules)

- no transaction can release locks prematurely and then request new locks, which could lead to inconsistencies or deadlocks.
- Distinguishes different **lock types**
  - Locks may be **acquired late** (depends on 2PL variant)
  - Locks may be **released early** (depends on 2PL variant)
- **restrictions** when locks are acquired/released
  - Guarantees **conflict-serializable** schedules
- Phase
  - Phase 1: ONLY **acquire** locks
    - must acquire a S (shared) lock before reading, and an X (exclusive) lock before writing.
  - Phase 2: ONLY **release** locks
    - cannot acquire new locks after releasing any locks

#### Variants

- **Conservative 2PL:** acquire all locks at transaction start
  - prevents deadlocks
- **Strict 2PL:** release all locks at transaction end
  - prevents cascading aborts
- **Conservative Strict 2PL**
- **Plain 2PL**: no restrictions on acquiring & releasing lock periods
  - Being non-conservative or non-strict is **more permissive**
  - Allows more transactions to proceed **in parallel**
  - Not preventing deadlocks or cascading aborts
- Optimal variant **depends on workload**
  - E.g., how likely are deadlocks and cascading aborts?
- Analyzing 2PL Schedules
  - To ensure **conflict-serializable** schedules

#### Proof (generates conflict-serializable schedules)

- Release First Lemma:
  - if conflict graph has path T1 to T2 then put a lock L1 AND T1 releases before T2 is finished acquiring its locks
  - T1 has started releasing locks while T2 has not
- **Induction start**: holds for paths of length 1
- **Induction step**: from paths of length I to i+1

- Assume 2PL schedule leads to a cycle in conflict graph
  - T1->…..->T2->T1
- For T1->…->T2, T1 is releasing locks while T2 is still acquiring a lock
  - time(T1 starts releasing) < time(T2 finishes acquiring)
- T2->T1 means that T2 is releasing locks while T1 is acquiring locks
  - time(T2 started releasing) < time(T1 finishes acquiring)
- By contradiction, 2PL can not produce **all** conflict serializable schedules
  - W1(A) R2(A) C2 R3(B) C3 W1(B) C1
  - Conflict graph has three nodes, two edges → no cycle
  - Could this have been produced by 2PL?

### Deadlocks (non-conservative 2PL)

- Deadlock: transactions waiting in a "circle"
  - May be acceptable if deadlocks are rare
- Handling deadlocks
  - Detect and resolve deadlocks
  - Prevent deadlocks from happening

#### Deadlock Detection

- Option1: assume deadlock after timeout
- Option2: Waits-for graph
  - One node for each transaction
  - Edge from T1 to T2 if T1 waits for lock held by T2
  - Edges are added as lock requests come in
  - Cycle in waits-for graph indicates a deadlock

#### Deadlock Resolve

- Only possibility: abort one deadlocked transaction
  - Aborted == restarted
- Optimize selection of aborted transaction
  - E.g., abort youngest transaction for least overhead

#### Deadlock Avoid / Prevention

- Proactively abort transactions that may cause deadlocks
- **Priority** based on timestamps (older transaction - higher priority)
- Pro of **Wait-Die**: Transactions that acquired all locks won't abort
- Con of **Wait-Die**: Young transaction may re-abort for same reason
- Avoiding **Starvation**
  - Higher priority transaction is never restarted for both
  - When restarting transaction, assign original timestamp
  - So transaction will be eventually prioritized
  - Avoids starvation (i.e., no transaction never processed)

##### Wound-wait protocol:

- T2 abort if T1 has higher priority, else T2 wait
- Proof
  - **lower** priority transaction wait for **higher** priority
  - Assume cycle in waits-for graph, transaction T1 in cycle
    - T1 → T2: T1 must have **lower** priority than T2
    - T1 → T2 → T3: T1 must have **lower** priority than T3
    - T1 → ... → T1: T1 must have **lower** priority than T1
    - Leads to a contradiction so no cycle is possible!

##### Wait-die protocol:

- T1 waits if T1 has higher priority than T2, else T1 die
- Proof
  - Only **higher** priority transaction can wait for **lower** priority
  - Assume cycle in waits-for graph, transaction T1 in cycle
    - T1 → T2: T1 must have **higher** priority than T2
    - T1 → T2 → T3: T1 must have **higher** priority than T3
    - T1 → ... → T1: T1 must have **higher** priority than T1
    - Leads to a contradiction so no cycle is possible!

### Handling phantoms

- T1 selects students with name starting with F
- T2 inserts new student "Frank"
- T1 selects students starting with F again
  - Suddenly, new student in the query result
- Problem: 2PL only locked students present **at first query**

#### Avoid phantoms

- Predicate locking: lock tuples satisfying certain predicate
  - "name starts with F"
  - Locks current and future entries equally
  - Complex to realize for arbitrary predicates
- Use index for equality predicates
  - Lock index page that would change at insertion
  - Cannot insert if index page is locked

### Efficient index locking

- Observation: traverse tree into one direction only

1. 锁当前 node:
   - Locking one node sufficient to block other transactions
   - I.e., keeping later transactions out of current sub-tree
2. 找要的 children: Locking for index **lookups** (crabbing):
   - Identify next node (child node or root at start)
3. 锁 children, 解锁当前 node:
   - Lock next (read lock), then unlock parent - repeat

#### Locking for index updates

- Index updates change index leaf nodes, may propagate up
- However, updates may not propagate upwards of "safe" nodes
  - **Safe node** is less than full (insertions)/more than half full (deletions)
- When traversing tree, release **prior locks** at each safe node
- May pessimistically request write locks but reduces performance
- Can optimistically request read locks for all nodes except leaf
  - Bets on no propagation, may have to restart if we lose

### Multi-granularity locking

- Best granularity may depend on query
  - E.g., whether we access most or few table rows
- Multiple-granularity locking mixes lock granularities
  - Locks for entire table & single rows
- Challenge: granting locks of diverse granularity consistently
- Cannot treat locks at different granularities separately
  - May grant conflicting locks otherwise
- Need locks on containing objects before locking object

#### Intention locks

- **IS (Intention Shared)**: want shared lock on contained object
  - lock on ancestors before requesting Shared lock
- **IX (Intention Exclusive)**: want exclusive lock on contained object
  - lock on ancestors before Exclusive lock
  - two transactions can place an IX lock on the same resource
  - not directly conflict at that point because they could place the X lock on two different children! Database manager ensures NO placing X locks on the same node later on while allowing two IX locks on the same resource.
- Must release locks in bottom-up order to avoid inconsistent locks

## Optimistic Concurrency Control

- 在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据
  - Conflicts are rare, no need to avoid proactively
- **Pessimistic assumption**: conflicts are likely
  - prevents conflicts proactively
  - Locking itself leads to overheads
    - lock management: Acquiring, releasing...
    - possible deadlocks: deadlock detection and resolution...
- (Bookkeeping) Keep read & write set for each transaction
  - **Read set**: objects that the transaction read
  - **Write set**: objects that the transaction wrote
- Overheads
  - Record read and write sets
  - Restarts / rollback if validation fails
  - Slow: Critical section during validation/writes (atomic)
  - Good if probability of conflicts is low

### Execution Phases

- Read
  - Read relevant data from database
  - Execute transaction on **private copy**
- Validate
  - Check conflicts with other transactions
  - Assign transactions to unique timestamps at validation
    - Try to serialize transactions in timestamp order
  - Two transactions cannot have conflicted if
    - T1 completes before T2
    - T1 completes before T2 starts writing, W1(A) **disjunct** with R2(B)
      - T1 writes to A, THEN T2 reads and writes B and C
      - disjunct: non-overlapped sets
    - T1 completes reads before T2 completes reads, Writes(T1) disjunct with Reads(T2) and Writes(T2)
      - T1 writes to object C, while T2 writes to D
- Write
  - Publish local changes if no conflicts

### Simplification (combine validation & write)

- Only one transaction can be in validation+write phase
  - No over-lapping writes
- Only consider conflict cases 1 and 2
  - Write phases cannot overlap

## Timestamp concurrency control

- Associate transactions with timestamps
  - Read timestamp: time of last read
  - Write timestamp: time of last write
- Serialize transactions in timestamp order
- Overheads
  - Restarting overheads for aborted transactions
  - Need to keep track of object timestamps
    - space consumption increases
    - updating timestamps - write for each operation

### Rules

- TS(T): timestamp of transaction T
- RTS(A), WTS(A): read & write timestamp of object A
- want to read database object A
  - Abort & restart if TS(T) < WTS(A) (开始 Transaction 之后有被改过)
- want to write database object A
  - Abort & restart if TS(T) < RTS(A) (有人在开始 Transaction 之后读过)
- TS(T) < WTS(A)
  - 如果是 Write, A 最新的值被改到老值, 需要 abort
  - 如果是 read, A 被 overwrite, 需要 abort
  - 两个都违反了 Serialization Order
- Thomas Write Rule
  - ignores outdated writes
  - Write A but TS(T) < WTS(A)
    - R1(A) W2(A) C2 W1(A) C1
    - Not conflict serializable but view-serializable
    - Simplifies to R1(A) C2 W1(A) C1

## Multi-version concurrency control

- Idea: keep multiple versions of database objects
- **以我启动的时刻为准，如果一个数据版本是在我启动之前生成的，就认；如果是我启动以后才生成的，我就不认，必须找到它的上一个版本为止**
- Example
  - R1(A) W1(A) R2(A) W2(B) R1(B) W1(C)
  - Not conflict-serializable
  - fix by moving R1(B) before W2(B)
  - Making R1(B) read old version of B has same effect

### MVCC protocol

- Each transaction receives timestamp when entering
  - Serialize transactions in this order
- Each write creates a new version of an object
  - Perform write check and abort if not valid
  - Version has timestamp of writing transaction
- Read mapped to last version **before transaction timestamp**
  - Transaction with timestamp i reads version with largest timestamp k such that k < i

### Write Check

- Consistent with transaction timestamps
- Can transaction with timestamp I write object A?
  - Assume transaction with timestamp > I
  - Cannot read earlier version of A than I
  - Must abort if this has already happened
    - Track read timestamps for versions!

### Abort-Related Behavior

- Aforementioned protocol guarantees serializability
- Need additional mechanisms for abort properties
- E.g., delay commits for recoverability

## Snapshot isolation SI

- Each transaction operates on database snapshot
  - Uses last committed value for each object
- Maintains multiple object versions internally
  - Different from MVCC: **no uncommitted values**

### Handling Writes

- Check before commit for overlapping writes
  - if target objects changed
    - abort & restart transaction
- Example With SI
  - tables A and B with one integer column each
    - T1: Insert into B select count( \* ) from a;
    - T2: Insert into A select count( \* ) from b;
  - Both has out-dated snapshot
  - Causes Write Skew
    - each transaction reads overlapping data and writes to disjoint data based on the reads
- Serializability vs. SQL Definition
  - SQL-92 standard defines isolation via anomalies
  - The write skew anomaly is missing, drawing criticism
  - Careful, may get SI when choosing serializable isolation
