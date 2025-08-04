# SQL Operations

In SQL, operations are categorized based on their function. The main categories are:

## DQL - Data Query Language
Query data from table, your typical select

## DDL - Data Definition Language
Define, modify and remove the structure of database object like tables, indexes and schemas. They manage the *structure* of the table, not the data itself.
- DDL commands are used to define, modify, and remove the structure of database objects like tables, schemas, and indexes.
- **`CREATE`**: Used to build new database objects. Examples include `CREATE TABLE`, `CREATE DATABASE`, and `CREATE INDEX`.

- **`ALTER`**: Modifies the structure of an existing database object. Examples include adding a column or changing a data type.
- **`DROP`**: Deletes an entire database object. 
- **`TRUNCATE`**:
- **`COMMENT`**:
- **`RENAME`**:

## DML - Data Manipulation Language
Commands used for adding, deleting and modifying data within the tables
DML commands are used for managing and manipulating data within existing table structures.
- **`INSERT`**: Adds new rows of data into a table.
- **`UPDATE`**: Modifies existing data within a table.
- **`DELETE`**: Removes rows from a table.
- **`MERGE`**: Performs an "upsert" operation (inserting or updating) based on whether a row exists.

## DCL - Data Control Language
These are used to manage user access rights and permissions in the database.
- **`GRANT`**: Gives a specific user permissions to perform certain tasks on database objects.
- **`REVOKE`**: Removes permissions from a user.

## TCL - Transactions 
Transactions are a group of operations treated as an all-or-nothing unit.
- **`BEGIN`** or **`START TRANSACTION`**: Marks the beginning of a transaction.
- **`COMMIT`**: Saves all the work done in the current transaction.
- **`ROLLBACK`**: Discards all changes made during the transaction.
- **`SAVEPOINT`**: Sets a point within a transaction that you can later roll back to, without discarding the entire transaction.

<img width="725" height="404" alt="image" src="https://github.com/user-attachments/assets/060cbf56-2675-4c8d-a2b8-08218d4e5508" />


## Production Guidelines:

Usually DCL and DDL are reserved for DBA roles
Any onther than DQL, should be done under your organizational deployment guideliness/rules.

From a resiliency production point of view, you should run at least two database instances in separate hosts: prod and a prod-replication instance. These different instancess should always be hosted at separate infrastructure - different *physical* servers or different availabiliyty zones.

Not only you will address the issue of ensuring availability and disaster recovery to your prod operation but will also address an overlooked but, necessary operational need: providing your operations team with DQL access to the data of the applications they support. 

Offloading DQL queries to the replication instance will provide the required visibility without impacting the performance of the main database. Concealing DQL access under a process to retrieve the ability of running a Select statement, is a bottle neck, slows business requriments, incident response and indicates lack of trust. 

A persistent refusal to grant Select access points to a deep organizational issue: a dysfunctional IT culture or, to put it nicely; a misalignment between the hired people and the trust a company is depositing in such resources. 

If a table contains PII or sensitive data that you need to conceal or keep apart due to GDPR or HIPAA regulations, the solution is not blocking access but implementing modern security controls like **data masking, column level security and/or granular roles** to strike a balance between providing access to your ops teams and being compliant with security policies.

