---
title: Index Locking In MySQL8 with InnoDB
date: 2023-09-07 23:26:30
description: "Index Locking In MySQL8 with InnoDB"
categories: MySQL
tags:
- MySQL
- Database
---

# **MySQL data structures for Index**

[8.3.9 Comparison of B-Tree and Hash Indexes](https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html)

[B-Tree vs Hash Table](https://stackoverflow.com/questions/7306316/b-tree-vs-hash-table)

- Hash Table: You can only access elements by their primary key `O(1)`. When you select ranges (*everything in between `x` and `y`*) can result in a full table scan `O(n)` . There may be a point where your index exceeds a tolerable size compared to your hash sizes and your entire index needs to be re-built.
- Tree algorithms support ranges selecting in *`log(n)`* , elements accessing and inserting by their primary key in *`log(n)`* .
<!--more-->

***Related topic:***

- Page Splitting: Create a new table with`NOT NULL PRIMARY KEY AUTO_INCREMENT` to avoid vacancy in page(higher memory usage) and to ensure orderly insertion(higher performance).
- Primary key type: The smaller the length of the primary key, result in smaller occupied space by the ordinary index(leaf nodes in Indexes other than the primary key → primary key).

# **MySQL Clustered and Secondary Indexes**

> 15.6.2.1 Clustered and Secondary Indexes
> 
> 
> *Each `InnoDB` table has a special index called the clustered index that stores row data. Typically, the clustered index is synonymous with the primary key.*
> 
- Select by primary key → find in clustered index B+ tree → get the required data(row).
- Select by Indexes other than the primary key → find in secondary index B+ tree→ get the primary key for data → find in clustered index B+ tree → get the required data(row).

***Related topic:***

[Using Covering Indexes to Improve Query Performance](https://www.red-gate.com/simple-talk/databases/sql-server/learn/using-covering-indexes-to-improve-query-performance/)

- Covering index: The covering index contains the data to be searched through include, so that the SQL query can get the required data without reaching the basic table.

[MySQL Prefix Index](https://www.mysqltutorial.org/mysql-index/mysql-prefix-index/)

- Prefix index: MySQL allows you to create an index for the leading part of the column values of the string columns using the following syntax: `column_name(length)` . Create a prefix index can save space, but may increase the number of query scans and make the Covering index useless.

# **MySQL multiversion concurrency control (MVCC)**

[15.7.2.4 Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)

- SELECT … FOR SHARE: Sets a shared mode lock on any rows that are read. Other sessions can read the rows, but cannot modify them until your transaction commits.
- SELECT … FOR UPDATE: For index records the search encounters, locks the rows and any associated index entries, the same as if you issued an `UPDATE` statement for those rows.

[13.3.1 START TRANSACTION, COMMIT, and ROLLBACK Statements](https://dev.mysql.com/doc/refman/8.0/en/commit.html)

[15.7.2.3 Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)

[15.3 InnoDB Multi-Versioning](https://dev.mysql.com/doc/refman/8.0/en/innodb-multi-versioning.html)

- The snapshot of the database state applies to `[SELECT](https://dev.mysql.com/doc/refman/8.0/en/select.html)` statements within a transaction, not necessarily to [DML](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dml) statements.

```
# suppose command below are in each transaction, REPEATABLE READ# Snapshot/Consistent read, no lock (MVCC, redo log), each consistent read within a transaction sets and reads its own fresh snapshot.
select * from table where id = ?;
-----------------------------------------------------------------# See the “freshest” state of the database, A SELECT blocks until the transaction containing the freshest rows ends.# set S lock
select * from table where id = ? for share;# set X lock
select * from table where id = ? for update;show engine innodb status;
```

# **MySQL Lock**

## **Lock types for MySQL server:**

***1.Table Lock (***[13.3.6 LOCK TABLES and UNLOCK TABLES Statements](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html))

```
LOCK TABLES
    tbl_name [[AS] alias] lock_type
    [, tbl_name [[AS] alias] lock_type] ...lock_type: {
    READ [LOCAL]
  | [LOW_PRIORITY] WRITE
}UNLOCK TABLES
```

A session can release its table locks explicitly with `UNLOCK TABLES`. If the connection for a client session terminates, whether normally or abnormally, the server implicitly releases all table locks held by the session (transactional and nontransactional). Since InnoDB supports row-level locking, we rarely use LOCK TABLES command.

***2. MDL(Meta data lock) (***[8.11.4 Metadata Locking](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html))

MDL is added automatically when accesses a table. MySQL uses metadata locking to manage concurrent access to database objects and to ensure data consistency. The server achieves this by acquiring metadata locks on tables used within a transaction and deferring release of those locks until the transaction ends. MDL read lock is added when you manipulate data in a table(DML — Data Manipulation Language). MDL read lock is added when you perform a Data Definition Language(DDL) operation or `LOCK TABLES` on a table.

- Meta data Read lock: Read locks are not mutually exclusive, so you can have multiple threads adding, deleting, modifying, and querying a table at the same time.
- Meta data Write lock: Read-write locks and write locks are mutually exclusive to ensure the security of operations that change the table structure. Therefore, if two threads want to add fields to a table at the same time, one of them will not start executing until the other is finished.

***Related topic:***

- Long transaction:
1. DML(***MDL read***) block DDL(***MDL write***): If session A operate a DML on table Z, the fellowing DDL operation on table Z will be blocked until transaction in session A completes. To solve this problem, check `innodb_trx` table in `information_schema` before DDL operation. Another solution is to set wait time in the DDL statement to avoid blocking fellow operation when fail in MDL acquirement. (`ALTER TABLE tbl_name WAIT N add column …`). The Performance Schema `[metadata_locks](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-metadata-locks-table.html)` table exposes metadata lock information, which can be useful for seeing which sessions hold locks, are blocked waiting for locks, and so forth.
2. DDL(***MDL write***) block DML(***MDL read***)
3. DDL(***MDL write***) block DDL(***MDL write***)

## **Lock type for MySQL engine(`InnoDB`):**

[15.7.1 InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-gap-locks)

***Shared and Exclusive Locks***

`InnoDB` implements standard row-level locking where there are two types of locks, shared (`S`) locks and exclusive (`X`) locks.

***Intention Locks***

Intention locks are table-level locks that indicate which type of lock (shared or exclusive) a transaction requires later for a row in a table. There are two types of intention locks:

- An intention shared lock (`IS`) indicates that a transaction intends to set a *shared* lock on individual rows in a table.
- An intention exclusive lock (`IX`) indicates that a transaction intends to set an exclusive lock on individual rows in a table.

The main purpose of intention locks is to directly determine whether there is a lock conflict based on whether the intention lock exists.

- Before a transaction can acquire a shared lock on a row in a table, it must first acquire an `IS` lock or stronger on the table.
- Before a transaction can acquire an exclusive lock on a row in a table, it must first acquire an `IX` lock on the table.

***1.Record Locks***

Record locks always lock index records, even if a table is defined with no indexes. For such cases, `InnoDB` creates a hidden clustered index and uses this index for record locking.

***2.Gap Locks***

A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record. For example, `SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;` prevents other transactions from inserting a value of `15` into column `t.c1`, whether or not there was already any such value in the column, because the gaps between all existing values in the range are locked.

- Gap locking is not needed for statements that lock rows using a unique index to search for a unique row. (This does not include the case that the search condition includes only some columns of a multiple-column unique index; in that case, gap locking does occur.).
- Gap locks can co-exist. A gap lock taken by one transaction does not prevent another transaction from taking a gap lock on the same gap. There is no difference between shared and exclusive gap locks. They do not conflict with each other, and they perform the same function.
- Gap locking can be disabled explicitly. This occurs if you change the transaction isolation level to `READ COMMITTED`(MySQL default transaction isolation level:`REPEATABLE READ` ) .Gap locks exist when the isolation level is `REPEATABLE READ` and `SERIALIZABLE.`

***3. Next-Key Locks***

[Why Next-Key lock is called this way?](https://stackoverflow.com/questions/56828411/why-next-key-lock-is-called-this-way)

A next-key lock is a combination of a record lock on the index record and a gap lock on the ***gap* *before the index record***.

By default, `InnoDB` operates in `REPEATABLE READ` transaction isolation level. In this case, `InnoDB` uses next-key locks for searches and index scans, which prevents phantom rows (see [Section 15.7.4, “Phantom Rows”](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html)).

```
CREATE TABLE `t` (
  `pId` int(10) NOT NULL,
  `name` varchar(10) DEFAULT NULL,
  `num` int(10) DEFAULT NULL,
  PRIMARY KEY (`pId`),
  KEY `num` (`num`)
) ENGINE=InnoDB;covering index: num
primary index: pId------------------------------------
pId(int) | name(varchar) | num(int)
---------|---------------|----------
 2       |      Atom     |   55
---------|---------------|----------
 5       |      Bom      |   22
---------|---------------|----------
 9       |     Cindy     |   33
------------------------------------select * from table where name = `Bom` for update;
 --> scan all clustered index(pid) in table
 --> addRecord Lockson clustered index 2, 5, 9
 --> addGap Lockson clustered index (-∞,2)(2,5)(5,9)(9,+∞)select * from table where pid = 2 for update;
 --> scan pid = 2 on clustered index(pid) in table
 --> addRecord Lockson clustered index 2select * from table where pid = 7 for update;
 --> scan pid = 7 on clustered index(pid) in table, not exist
 --> addGap Lockson clustered index (5, 9)select * from table where num = 33 for share;
 --> scan secondary index(num) in table
 --> addRecord Lockson secondary index 33
 --> addGap Lockson secondary index (22,33)(33,55)select * from table where num = 33 for update;
 --> scan secondary index(num) in table
 --> addRecord Lockson secondary index 33
 --> addGap Lockson secondary index (22,33)(33,55)
 --> scan clustered index(pid) by secondary index 33 in table
 --> addRecord Lockson clustered index 9
```

***Related topic:***

- Long transaction.
- Dead lock:
1. set `innodb_lock_wait_timeout` to avoid long blocking.
2. `innodb_deadlock_detect = on` If a deadlock is found, roll back a transaction in the deadlock chain to let other transactions continue to execute.