---
title: 2. DDL & DML (SQL Query)
categories:
  - Course-work
  - CS-5320-Database
---

# SQL Queries Definition

- Database
  - Organized collection of inter-related data used to model aspects of real-world
  - Set of Relations (tables)
- Relation
  - **Schema**: description (column header, type)
  - **Instance**: set of data satisfying the schema
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
  - Groups SQL commands (transactions - Read / Write)
  - `SELECT`
- DCL: 控制, 分配, access permissions - **Data Control Language**
  - Assign data access rights
  - `GRANT, REVOKE, COMMIT, ROLLBACK`

---

## DDL: 定义 Data Definition Language (Schema, Relations, Constraints)

Not using Single Table in database schema design

- Reduce redundancy
  - A well-designed schema breaks the data into **multiple related tables** to avoid repeating the same information across rows.
- Minimize (inconsistent or incorrect) update errors
  - By splitting data into related tables, each piece of information exists in only one place, making updates more reliable and reducing the chance of errors.

### Schema

Define relations with their **schemata**

- columns and column **types** - `CREATE`
- `CREATE TABLE <table> (<table-def>)`
  - `<table>` is the table name
  - `<table-def>` is comma-separated column definitions
  - Column definition: `<col-name> <col-type>`
- Example
  - `CREATE TABLE Students(Sid int, Sname text, Gpa real);`

### Constraints

- Define **constraints** restricting admissible content
  - Constraints on **single relations**
  - Constraints linking **multiple relations**
- Primary key
  - unique
  - refers to a single table
  - identifies a subset of columns as key columns
  - Fixing values for key columns must identify row
  - `ALTER TABLE <table name> ADD Primary Key (col1, col2);`
    - `ALTER TABLE Students ADD PRIMARY KEY(Sid, Cid);`
- Foreign key
  - A foreign key constraint links **two tables**
  - table 1: has **foreign key columns**
  - table 2: has **primary key** and mapped (`=`) to table1's foreign key
  - Maps **each row** in table 1 to a row from table 2
  - `ALTER TABLE <table-1> ADD Foreign Key (<fkey-columns>) REFERENCES <table-2> (<pkey-columns>);
    - Sid in Enrollment ===== student in Students table
    - `ALTER TABLE Enrollment ADD FOREIGN KEY(Sid) REFERENCES Students(Sid);`
- Range: `ALTER TABLE students ADD CHECK(gpa>=0 and gpa<=4.0);`
- Unique: `ALTER TABLE courses ADD CONSTRAINT c1 UNIQUE(cid);`
- Integrity
  - Ensuring that the logical and physical data stored in SQL Server are structurally sound and consistent
  - limit the admissible content of tables
  - enforced by DBMS
  - `ALTER TABLE` to add integrity constraints

---

## DML: 操作数据 Data Manipulation Language (Insert, Delete, Update, Analyze)

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
    - `UPDATE **<table> SET <column> = <value> WHERE <condition>`
    - `UPDATE Courses SET Cid = 7 WHERE Cname = 'CS4320'`
- Analyze
  - describes a **new relation** to generate
    - `SELECT`, `FROM`, `WHERE`,...
