# SQL for Prod Support and SRE

This is the place for you if:

- You know basic SQL and need a practical refresher
- You have been lost in the land of management or scrum mastering, but you now need to get your hands -ugh, like a peasant- dirty, with code again.
- You are learning SQL and theory has tired you up

We will start with the simple stuff and ramp up, focus is prod-support and SRE operations.

## Conventions

- Everithing here is PostgreSQL syntax. Concepts are Universal, syntax remains Postgres.
- **Sybase** People who has worked with me knows I have my fair share of war stories due to Sybase outages and these are not fond memories but, a surprinsingly number of financial instutions still run over on it; so yes, we will have a sybase section. Commands like `sp_who`, `sp_lock` are easy and lifesavers during a crisis, these will make you look like an SQL wizard
- I will think of more conventions as the time goes by :D

# SQL Operations

In SQL, operations are categorized based on their function. These are usually broken in 5 categories, you will hear these acronyms thrown around in interviews and documents so you better get familiar with them.

## DQL - Data Query Language

This is your bread and butter and It will be 99% of what you will be running with in support ops.  
- SELECT retrieves data from one or more tables: `SELECT * FROM table;`.
- Some charts or organizations omit DQL and group the SELECT statement into DML, notice you dont really manipulate anything with SELECT alone.

ALl the Operations besides DQL are **DBA territory**. You should not be running these directly in a production system outside a formal approved change process. Some organizations have different names for a software implementaion, some of the ones I know: 

- Deployment
- Rollout
- Change implementation
- Release
- Push
- RFC - Request For Change

---------------------------

## DDL - Data Definition Language
Define, modify and remove the structure of database object like tables, indexes and schemas. Emphasis on manage the *structure* of the table, not the data itself.
- **`CREATE`**: Builds a new database object like `TABLE`, `DATABASE`, `SCHEMA` and `INDEX`.
- **`ALTER`**: Modifies the structure of an existing database object, like adding a column or changing a data type.
- **`DROP`**: Deletes an entire database object. This is final. **Be careful**.
- **`TRUNCATE`**: **Removes all rows from a table, but retains the table structure**. Unlike DELETE, TRUNCATE typically cannot be rolled back in some databases and does not fire individual row delete triggers. 
- **`COMMENT`**: Adds descriptive text to database objects such as tables, columns, or indexes. These comments help document the schema and improve maintainability. Consider them breadcumbs for the next person, leave the comment you would've liked to find.
- **`RENAME`**:Changes the name of a database object, such as a table or column, without altering its structure or content. 

## DML - Data Manipulation Language
Commands used for adding, deleting and modifying data inside the tables

- **`INSERT`**: Adds new rows of data into a table.
- **`UPDATE`**: Modifies existing data within a table.
- **`DELETE`**: Removes rows from a table.
- **`MERGE`**: Performs an "upsert" operation (inserting or updating) based on whether a row exists.
- **`CALL`**: Invokes a procedure.
- **`LOCK`**: Restricts access to a table by other transactions.
- **`EXPLAIN`**: Reveals the execution plan the DB uses to run a query, returns hos tables are scanned, joined and sorted. 

## DCL - Data Control Language
Manage user access rights and permissions in the database. Who gets to do what.

- **`GRANT`**: Gives a specific user permissions to perform certain tasks on database objects.
- **`REVOKE`**: Removes permissions from a user.

## TCL - Transactions 
Transactions are a group of operations treated as an all-or-nothing unit.

- **`BEGIN`** or **`START TRANSACTION`**: Marks the beginning of a transaction.
- **`COMMIT`**: Saves all the work done in the current transaction.
- **`ROLLBACK`**: Reverts all changes made during the transaction.
- **`SAVEPOINT`**: Sets a point within a transaction that you can later roll back to, without nuking the entire transaction.

<img width="725" height="404" alt="image" src="https://github.com/user-attachments/assets/060cbf56-2675-4c8d-a2b8-08218d4e5508" />

# Dive in - our first steps

I recommend you use [sqlfiddle](https://sqlfiddle.com/) or install [pgadmin](https://www.pgadmin.org/download/). Pgadmin is a great resource and most of the practical examples here do not require a dedicated db.

**Copy the statements from [this page](https://github.com/AlexAntartico/sql-crash-course/blob/main/employees_simple_table.db) and execute**. Consider the following:

- Table has duplicate IDs
  - Kirito Rogers, Deju Kent and Asuna Starks department is Sales
  - Zero Two's manager id is 101abc
  - Erza Prince's emp_id is 107mno
  - Managers have their same emp_id in manager_id field
  - Android 18 Smith is a manager

Don't use `SELECT * FROM table;` on 
- prod DBs
- DBs you are not familiar with (accessing for 1st time)

`SELECT *` is great for learning purposes but you dont want to do a full scan in a prod table during business hours. I cannot imagine a valid scenario where you whould want to have hundreds of thousands or millions of records in screen .We will approach with three simple steps:

1. Understand Table Structure
2. Fetch Basic Table Statistics
3. Sample the data

## Understand Table Structure

**The information_schema.columns view**

Run this query

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'employees';
```
You will see these records:

<img width="443" height="227" alt="image" src="https://github.com/user-attachments/assets/a7fbd23b-25e9-4e02-8bb3-e8794f2d621a" />

This specific query retrieves three pieces of information:
* `column_name`: The name of the column in the table
* `data_type`: The data type of each column, like `var`, `varchar`, `int`
* `is_nullable`: Whether each column contains `NULL` values

`information_schema.columns` is a system view. A **view** is a pre-defined SQL query, views not store data itself but dynamically displays data from one or more tables. This specific view contains metadata about the columns of all tables in the database, notice we are filtering a few columns where table name is `employees`.

**pg_indexes view**

run:
```sql
SELECT *
FROM pg_indexes
WHERE tablename = 'employees';
```

`pg_indexes` is another system view in PostgreSQL that contains information about the indexes, primary keys or foreign keys on tables in the database. We have to specify the table, else we will fetch everything. Output will look like this:

<img width="544" height="200" alt="image" src="https://github.com/user-attachments/assets/a2af996c-1fe4-4871-82be-18c8cf8c86ac" />

Do you notice something weird on the output of this query? The lack of indexes in this table is not something standard in any regular prod environment?

**Why are these two queries efficient?** These are metadata queries on system catalogues, we are not scanning the full table.

## Production Guidelines: Please Don't Be a Bottleneck

Let's get this straight: In a prod environment you **should** be having DQL access to the DB's of the applications you support. Any other command belonging to DML, DDL etc are usually reserved to either DBA teams or you will need to go through your organization's formal deployment process. Do not cowboy the system, nIt does not store data itself but dynamically displays data from one or more tables.o exceptions.

From a resiliency point of view, the absolute bare minimum is to run two database instances on separate hosts. These are usually named prod and a prod-replication (or prod-replica/prod-repl). These instancess MUST live on entirely different infrastructure, be it different physical hosts  or even better, different availability zones. If your primary replication is in the same rack or worse: on the same physical host (I have seen this happened (╯°□°)╯), you have not built resilient infrastructure, you created a single point of failure and a post-mortem document that will be very difficult and akward to explain after that outage that will render your db unusable.

The two-instance set-up is not only to switch to the healthy server during an outage. It also helps to solve an operational need: providing your support teams with read-only access to the data of the applications they manage. By offloading all those SELECT queries to replication, you provide your teams the visibility they need to do their jobs without consuming resources from the production DB. 

Not only you will address the issue of ensuring availability and disaster recovery to your prod operation but will also address an overlooked but, necessary operational need: providing your operations team with DQL access to the data of the applications they support. 

Making your operations team to file a ticket and wait for approval just to run a SELECT query is an auto sabotage that will cripple the sla to reply to business requests and slow down incident response. I can honestly just think of two reasons to do this, a dysfunctional IT culture or a  lack of trust in the people running the systems.

![a2bydk](https://github.com/user-attachments/assets/17935f96-071e-4e3b-81cc-4f39716a1efe)

"But what can I do if a table contains PII or sensitive data" I hear you ask. If a table contains data falling under GDPR or HIPAA regulations, the answer will never be to lock the door and throw away the key, but implementing modern security controls like **data masking, column level security and/or granular roles** to strike a balance between providing access to your ops teams and being compliant with security policies. Is not 2005 anymore, we have better tools, better practices. Lets keep discipline in operations and implement them.

