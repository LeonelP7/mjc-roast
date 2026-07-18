# Database Study Guide for Junior Java Engineers

## Core Architectural Concepts & Trade-offs

<details>
<summary>What is the **CAP Theorem**, and what do Consistency, Availability, and Partition Tolerance mean in practice?</summary>

**Consistency (C):** means "Single Truth", every read operation receives the most recent write, or an error. If a customer updates their balance on Server 1, Server 2 must reflect that instantly. If it can't, it would rather throw an error than give old data.

**Availability (A):** means "Always Respond", every non-failing node returns a non-error response, without a guarantee that it contains the most recent write. Server 2 will always give the customer an answer, even if it's slightly outdated data because it lost connection to Server 1.

**Partition Tolerance (P):** means "Survive the Break", the system continues to operate despite an arbitrary number of messages being dropped or delayed by the network between nodes.

In the real world a partition (a network failure) it is an accident that happens. Backhoes cut cables, routers crash, and networks fail. Therefore, **Partition tolerance (P)** is mandatory for any distributed database. You can't choose to switch it off.

The real choice happens when a network partition occurs. You are forced to choose between Consistency (C) or Availability (A).

The CAP theorem states that a distributed database can only guarantee two of three properties at the same time: Consistency, Availability and Partition tolerance.
In practice, networks will eventually fail, so Partition Tolerance is mandatory. This means when a network partition occurs you must choose between:

* Consistency (CP): Where the database returns errors or blocks requests to prevent reading or writing outdated data (like in a banking app).
* Availability (AP): Where every node remains open to handle requests, but the data returned might be temporarily out of sync (like a social media feed).

</details>

<details>
<summary>How does the CAP Theorem apply differently to traditional **RDBMS** vs. **NoSQL** databases?</summary>

Traditional RDBMS systems like PostgreSLQ, MySQL and Oracle are typically designed to be CP (Consistent and Partition tolerant). They prioritize strict data integrity and cosistency across related tables, because they rely heavily on relationships between tables, complex joins and strict data types, having outdated or mismatched data across different tables would break the database rules. Meaning they would rather reject a request or return an error during a network split than serve outdated data.
Conversely, many NoSQL databases like Cassandra or DynamoDB are designed as AP (Available and Partition tolerant) systems. They are built for high-throughput, massive horizontal scalling and constant uptime. In these systems, availability is prioritized, maning every node will always accept reads and writes, even if data takes some time to sync across the cluster

</details>

<details>
<summary>What are the core differences between **ACID** (Relational) and **BASE** (NoSQL) consistency models?</summary>

BASE stands for Basically Available, Soft state, and Eventual consistentcy. Its the opposite of the strict ACID model used in relational databases.

* Basically Available means the system guarantees availability even during heavy loads or partial network failures. The database will keep responding, even if it has to return degraded performance or slightly stale data instead of crashing.

* Soft State means the data in the system can change over time without explicit user actions, because the background nodes are constantly syncing. A user might see slightly different data depending on which node they query.

* Eventual Consistency guarantees that given enough time without new updates, all replicas will eventually synchronize and reflect the exact same, latest state.

</details>

<details>
<summary>When designing a Java application, how do you decide whether to use a Relational Database over a NoSQL database?</summary>

Choosing between RDBMS and NoSQL database always depents on the specific requirements of the project.

I would choose a Relational Database (RDBMS) if the application requires strict data integrity, complex transactions, and immediate consistency - for example, an e-commerce checkout or inventory system where over-selling an item must be prevented at all costs. RDBMS excels here because of its strict ACID compliance.
On the other hand, i would choose a NoSQL database if the application demands high write speeds, horizontal scalability, and flexible data structures - such as IoT tracking system handling millions of GPS coordinates per second. In this case, massive write throughput and constant availability are critical, while slightly stale data is perfectly acceptable.

</details>

<details>
<summary>What are the main types of NoSQL databases (Document, Key-Value, Column-family, Graph), and what is an example of each (e.g., MongoDB, Redis)?</summary>

The four primary typesof NoSQL databases are:

1. Document databases: They store data in flexible JSON-like documents. A great Java-ecosystem example is MongoDB. It maps naturally to Java objects.

2. Key-Value Stores: They operate like a massive distributed hash map, optmized for ultra-fast lookups. A common examples used for caching in Java applications is Redis.

3. Graph Databases: They focus on data with complex relationships, using nodes and edges - perfect for social networks. Neo4j is a popular example.

4. Column-family (or Wide Column) Stores: They store data in columns instead of rows, which is highly efficient for analytical queries on massive datasets. An example is Apache Cassandra.

</details>

## 2. Relational Database Fundamentals & Design

<details>
<summary>What is a relational data model, and what are its core components?</summary>

A relational data model organizes data into structured, two-dimensional tables, where data points are related to one another.

In formal database theory, its core components are:
* Relations (which we commonly call Tables or Entities in Java)
* Tuples (which represent Rows, records or specific instances of an entity)
* Attributes (which represents Columns, fields or properties of that data)
The model enforces data integrity through contraints and keys, ensuring that relationships between different relations remain consistent.

</details>

<details>
<summary>Data point:</summary>

Data points are individual pieces of information, when said "data points are related to one another" we simply mean that the individual pieces of information within the same row belong together, and information in one table can link to information in another table.

Think of it like a single row in an Employee table:

    id: 101

    name: "Alice"

    salary: 75000

In this case, "101", "Alice", and "75000" are the individual data points. They are related because they all describe the exact same person (Alice). If you look at just the number 75000 by itself, it means nothing. But because it sits in the same row as "Alice," the database model inherently relates that salary to her.

Furthermore, if Alice’s row has a department_id of 3, that specific data point relates her entire row to a row in the Departments table.

</details>

<details>
<summary>What are **Primary Keys**, **Foreign Keys**, and **Candidate Keys**? Why are they essential for data integrity?</summary>

A Primary key is a column or combination of columns, chosen to uniquely identify each row in a table. It cannot contain null values.
A Candidate key is any column or set of columns that could qualify as a primary key because its values are unique and non-null cross the table. Out of all the Candidate keys, we choose one to be the primary key and the rest become Alternate keys.
A Foreign key is a column that links to aprimary key in another table. It is used to stablish and enforce relationships between tables.

These keys are crucial for Data integrity because Primary Keys prevent duplicate records, while Foreign keys enforce Referential Integrity, meaning they prevent links to non-existent data.

</details>

<details>
<summary>What is **Database Normalization**? Explain the requirements and practical goals of **1NF, 2NF, and 3NF**.</summary>

Database normalization is the process of structuring a relational database to reduce data redundancy (stop repeating the same data) and improve data integrity.

* 1NF (First Normal Form) requires data to  be atomic, maning each table cell must hold only a single indivisible value - no comma-separated lists or arrays.

For example moving the items to a separate table so every cell contains exactly one value.

* 2NF (Second Normal Form) requires the table to be in 1NF, and all non-key columns must depend on the entire primary key, eliminating partial dependencies.

The Problem: Look back at our table. If the primary key of an order lines table is a composite key of (order_id, item_id), but we also store customer_phone in that same table, the phone number only depends on the customer, not on what item was bought.

The Solution: Move customer details to a separate Customers table.

* 3NF (Third Normal Form) requires the table to be in 2NF, and eliminates transitive dependencies, meaning non-key columns cannot depent on other non-key columns. They must depend only on the primary key.

The Problem: In a Customers table, if you have customer_id, zip_code, and city. The city actually depends on the zip_code (a non-key column), not directly on the customer_id.

The Solution: Move zip_code and city to a separate Locations table. (As a famous database saying goes: All columns must depend on "the key, the whole key, and nothing but the key.")

</details>

<details>
<summary>What is **De-normalization**? In what specific real-world scenarios or performance bottlenecks would you use it?</summary>

De-normalization is the intentional process of adding redundant data back into a normalized database to optimize read performance.
We use it in read-heavy applications or analytics dashboards where executing complex SQL JOIN operations across multiple tables creates a performance bottleneck. By flattening the data structure, the database can fetch the required information in a sigle query, significantly reducing CPU load and response times at the expense of using more disk space and making write operations slightly more complex.

</details>

<details>
<summary>What are the different types of table relationships (1:1, 1:M, M:M), and how do you implement a **Many-to-Many (M2M)** relationship in SQL?</summary>

* 1:1 (One-to-One): Implemented by placing a unique foreign key in one of the tables.
For example: A User and their Passport. It's implemented by  placing a `UNIQUE` foreign key in one of the tables.
* 1:M (One-to-Many): Implemented by putting a foreign key on the "Many" side (eg., customer_id goes inside the Oders table).
For example: A Customer and their Orders. It's implemented by placing a foreign key on the Many side table (Orders).
* M:M (Many-to-Many): Implemented using an intermediate table (often called a Join Table or Junction Table).
For example: Students and Courses

</details>

<details>
<summary>How is a bidirectional 1:1 relationship implemented?</summary>

In the physical database, a 1:1 relationship is always unidirectional and is implemented by placing a UNIQUE foreign key in one of the two tables.
Bidirectionality is entirely a Java or ORM-level concept. In frameworks like Hibernate, we achieve a bidirectional 1:1 relationship by mapping both entities to each other using the `mappedBy` attribute on the non-owning side too to point to the actual physical foreign key on the owning side.

</details>

## 3. SQL Commands & Aggregations

<details>
<summary>* What are the categories of SQL statements? Give examples of:
  * **DDL** (Data Definition Language)
  * **DML** (Data Manipulation Language)
  * **DQL** (Data Query Language)
  * **TCL** (Transaction Control Language)</summary>

SQL commands are divided into four main categories based on their functionality:

1. DDL (Data Definition Language): These commands define or modify the database schema and structure. Examples include `CREATE`, `ALTER`, and `DROP`.

2. DML (Data Manipulation Language): These commands are used to manage and manipulate the data within the dables. Examples `INSERT`, `UPDATE` and `DELETE`.

3. DQL (Data Query Language): This is used strictly for retrieving data from the database, consisting of the `SELECT` statement.

4. TCL (Transaction Control Language): These commands manage database transactions to ensure data consistency. Examples include `COMMIT` and `ROLLBACK`.

</details>

<details>
<summary>What are column constraints (e.g., `NOT NULL`, `UNIQUE`, `CHECK`), and why should you use them?</summary>

Column constraints are declarative rules applied to a database columns that restrict the type, format or range of data that can be inserted or updated. Common examples include `NOT NULL`, which prevents missing values; `UNIQUE`, which ensures distinct values across rows; and `CHECK`, which validates data against a specific logical condition. You should always use constraints because they act as the ultimate, fooproof line of defense for data integrity. While validation can be done in Java code, database-level constraints ensure that your data remains clean, and safe from bugs, external scripts or manual data entry errors.

</details>

<details>
<summary>What are the standard SQL **Aggregation functions** (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`), and how do they work with `GROUP BY` and `HAVING`?</summary>

Think of a SQL query like an assembly line in a factory. The database processes your query in a specific order:

    FROM / JOIN: The database grabs the raw tables.

    WHERE: The database filters the individual data rows before doing any math. (For example, WHERE status = 'ACTIVE').

    GROUP BY: The database bunches the remaining rows into groups.

    Aggregation Math: Only now does the database calculate things like AVG(), SUM(), or COUNT() for each group.

Standard SQL aggregation functions - such as `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX` - are used to perform mathematical calculations on a set of values and return a single summary value. When combined with the `GROUP BY` clause, the database splits the data rows into distinct groups based on a specific column before running these calculations on each group individually. Because these calculations occur after individual rows are filtered, you cannot use a standard `WHERE` clause to filter the results of an aggregation. Instead, the `HAVING` clause is used immediately after `GROUP BY` to filter the final summarized groups based on the aggregated values.

</details>

## 4. Performance Optimization: Indexes & Execution

<details>
<summary>What is a **Database Index**, and how does it speed up queries?</summary>

A Database index is a separate data structure, typically organized as a B-tree, that speeeds up data retrieval operations by allowing the database engine to locate specific rows without scanning the intire table. Instead of performing a costly Full Table Scan, the database queries the index to find the exact location of the record. The trade-off  is that indexes require additional disk space, and they introduce performance overhead on write operations like `INSERT`, `UPDATE` and `DELETE` because the index must be updated every time the underlaying data changes"

</details>

* How are indexes typically represented inside a database engine (e.g., **B-Trees / B+ Trees**)?

<details>
<summary>What is the difference between a **Clustered Index** and a **Non-Clustered Index**?</summary>

The fundamental difference between the two is that a **Clustered Index** physically reorders and stores the actual data rows of tha table based on the index key, just like how a dictionary is physically sorted from A to Z, Because the table can only be physically sorted in one way, a table can only have one clustered index (typically the primary key). In contrast a **Non-Clustered index** is a separate data structure that stores a sorted list of values alongside pointers or row indentifiers that reference the actual data location (like a book's page number) telling the database exactly where to find the actual row. Because non-clustered indexes do not alter the physical layout of the underlaying table, you can create multiple non-clustered indexes on a single table to optimize different query paths.

</details>

<details>
<summary>What are the pros and cons of over-indexing a table? Does it make sense to index a column with very low cardinality (e.g., a boolean or gender column)?</summary>

Over-indexing a table creates significant write performance overhead because the database must update the underlying B-Tree structures during every `INSERT`, `UPDATE`, and `DELETE`operation, while also consuming extra disk space. Additionally, indexing columns with low cardinality - such as booleans or gender flags - is generally ineffective. Because these columns yield high data duplication, queries reeturn a large percentance of the table, prompting the database optimizer to bypass the index entirely in favor of a faster sequential Full Table Scan.

The effectiveness of an index depents entirely on the query's filter criteria. For queries targetting a highly frequent value (e.g., fetching the 99% of the active records), the database optimizer will bypass the index in favor of a Full Table Scan because fetching that many row pointers is inefficient. However, for queries targetting rare values (e.g., fetching the 1% of soft-deleted records), the index is highly effective, allowing the engine to instantly pinpoint and retrieve the sparse data without scanning the entire table.

</details>

<details>
<summary>What are **Stored Procedures** and **Triggers**? What are their use cases, and what are the architectural downsides of putting business logic inside the database?</summary>

**Stored Procedures:** A precompiled collection of SQL statements and contro-flow logic (like loops and conditional statemets) that is stored directly on the database server. It acts like a function or method within the database that must be explicitly called by an application or user to execute a specific task or business process.

**Trigger:** A specialized, event-driven script that is automatically executed ("fired") by the database engine in response to a specific modification event on a table such as `INSERT`, `UPDATE`, or `DELETE`. Triggers cannot be called manually; they act as automated event listeners to enforce data integrity or audit logs.

While stored procedures and triggers offer high performance by executing code directly on the database server and eliminating network latency, they introduce severe architectural downsides. Reliying on them forces vertical scaling (upgrading database hardware) which is expensive and limited, whereas keeping business logic in the Java application layer allows for cheap, highly available horizontal scalling across stateless nodes. Furthermore, putting logic in the database fractures the architecture, severely hindering developer productivity by crippling standard Java debugging tools, JUnit automated testing and robust Git-based version control flows.

**Use cases:**

1. Stored procedure: bulk operations

imagine you need to update the salary of 50,000 employees based on some calculation (e.g., apply a 3% raise to everyone in a department). If you did this from java you would have to:

* Pull all 50,000 rows across the network into your app
* Loop through them in Java
* Send 50,000 `UPDATE` statements back over the network

That's 50,000+ round trips (or one big batch, but still a lot of data movement). A stored procedure does the entire loop on the database server - no data has to leave de DB, no network latency per row. So:

* Bulk/batch data processing (mass updates, report generation, data migrations)
* Legacy enterprise systems where a lot of business logic was written directly in SQL/PL-SQL decades ago (this is comon at places like banks)
* Security - sensitive operations, where you want to gran an application user permission to execute a procedure but not permission to directly `SELECT`/`UPDATE` the underlaying tables - the procedure acts as a controlled gateway.

2. Trigger

A `CHECK` constraint can only look at one row, in isolation, using values within that same row. It has no memory of the past and can't compare against other rows.

A trigger can do both of those things. Examples:

* Auditing/history: When a row in an `orders` table is updated, a trigger automatically copies the old values into an `orders_history` table before the change is applied. A `CHECK` constraint physically cannot do this - it doesn't know what the previous value was, and it can't write to different table.

* Cross-row validations: "The total quantity of items reserved for a product can never exceed the stock available." That requires comparing the new row against a sum of other rows - a `CHECK` constraint can't see other rows, only a trigger (or an application-level check) can.

</details>

## 5. Transactions, Concurrency, & Java Integration

<details>
<summary>What does **ACID** stand for? Explain each property (Atomicity, Consistency, Isolation, Durability) in the context of a database transaction.</summary>

A: Atomicity, is about each transaction has to be treated as one single command for ensuring data consistency and avoiding errors that can occur in the middle of a query. Either all the transaction success or it fails and rollsback.

C: Consistency, meaning that each query has to take the database from a valid state, and leave it in a valid state also.

I: Isolated: states that concurret transations should be independent and isolated from the rest, a transaction cannot interfer with other's intermediate, uncommited state.

D: Durable, that once made changes on a table or inserting data, they remain in the database even in the event of a crash or power loss.

</details>

<details>
<summary>What are the standard **Transaction Isolation Levels** (Read Uncommitted, Read Committed, Repeatable Read, Serializable), and what concurrency phenomena (Dirty Read, Non-repeatable Read, Phantom Read) do they prevent?</summary>

**Concurrency phenomena:**

Setup: Imagine two transactions running at the same time on a balance column that currently holds `100`.

1. **Dirty Read - reading data that isn't even final yet**
* Transaction A  updates the balance to `50`, but hasn't commited yet (still "in progress").
* Transaction B reads the banlance right now and sees `50`
* Then Transaction A hits an error and rolls back - the balance goes back to `100`.
* Transaction B walked away believing the balance was `50`. That's wrong. It read "dirty" (uncommited, possibly-about-to-be-undone) data.

2. **Non-Repeatable Read - the same query gives two different answers, within the same transaction**
* Transaction B reads the balance: sees `100`.
* Transaction A updates it to `50` and commits (this time for real).
* Transaction B reads the balance again, in the same transaction: now sees `50`.
* B asked the same question twice and got two different answers. The value of an existing row changed underneath it.

3. **Phantom Read - the same query returns a different set of rows**
* Transaction B runs: `SELECT * FROM accounts WHERE balance > 40` → gets 5 rows.
* Transaction A inserts a new row with balance 60, and commits.
* Transaction B runs the exact same query again: now gets 6 rows.
* Nothing about the original 5 rows changed - a new row appeared like a ghost ("phantom") that matches the filter.

**Isolation levels:**

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
| :--- | :--- | :--- | :--- |
| **Read Uncommitted** | ❌ allowed | ❌ allowed | ❌ allowed |
| **Read Committed** | ✅ prevented | ❌ allowed | ❌ allowed |
| **Repeatable Read** | ✅ prevented | ✅ prevented | ❌ allowed |
| **Serializable** | ✅ prevented | ✅ prevented | ✅ prevented |

* **Reads Uncommited** - no protection at all. You can literally see other transactions' half-finished work, Rarely used in practice.
* **Reads Commited** - the default in PostgreSQL, and Oracle. You can only ever read data that's has been commited, but a value can still change between two reads in your own transaction.
* **Repeteable Read** - the default in MySQL/InnoDB. Once you've read a row, it's guaranteed to look the same for the rest of your transaction. New rows can still sneak in though.
* **Serializable** - the strictest. Transactions behave as if run one at a time. In sequence. No phantoms, no surprises - but heavy locking, so throughput drops.

</details>

<details>
<summary>Strategies for isolation (Repeteable Read):</summary>

On the Java read-write lock comparison:
It's a reasonable mental starting point, but here's the catch — not all databases actually implement isolation using locks. There are two different strategies databases use to achieve the same guarantees:

1. Lock-based - this is closer to your Java analogy. A transaction reading a row can block other from writting to it (or vice versa), literally like `ReadWriteLock`.
2. MVCC (Multi-Version Concurrency Control) - this is what PostgresSQL, and MySQL/InnoDB actually use, and it's bit different. Instead of blocking anyone, the database keeps multiple versions of a row. When your transaction starts, it gets a "snapshot" of the data as it looked at that moment. Other transactions are completely free to keep reading and writing the row - they're just working with their own snapshot/version. You never get blocked and neither do they.

</details>

<details>
<summary>What is the difference between **Optimistic Locking** (e.g., `@Version` in JPA/Hibernate) and **Pessimistic Locking**?</summary>

**Pessimistic Locking** - "I assume someone else will try to touch this row, so i'll lock it immediately"

When a Transaction A reads the row it wants to update, it takes out an actual database lock on that row right then and there (e.g., `SELECT ... FOR UPDATE` in SQL). Any other transaction that tries to read-for-update or write that same row has to wait until A commits or rolls back and releases the lock.

* Guarantees no lost update, no race at all - the second transaction physically cannot procced until the first is done.
* Cost: other transactions sit there blocked, waiting. If A is slow (or crashes without releasing the lock cleanly), everyone queued behind it suffers. Bad for high-concurrency, high-traffic systems.

**Optimistic Locking** - "I bet nobody else will touch this row before I'm done, so i won't lock anything. I'll just check at the very last second."

No lock is taken while reading. Instead, the row carries a version number (or timestamp) column. The flow:

1. Transaction A reads the row: `stock = 10, version = 1`
2. Transaction A prepares its update in memory (no lock).
3. When A goes to save, it runs something like: `UPDATE product SET stock = 8, version = 2 WHERE id = 1 AND version = 1`.
4. If nobody else touched the rowin the meantime, `version` is still `1`, the `WHERE` clause matches, the update succeds, and `version` becomes `2`.
5. If someone else already updated the row first (so `version` is now `2`, not `1`), the `WHERE` clause matches zero rows - the update silently fails to apply, and the application layer detects this and throws an error (in JPA/Hibernate, this is exactly what `@Version` does - it throws an `OptimisticLockException`)

* No blocking at all - great for throughput.
* Cost: if a real conflict happen, the loser doesn't get their change saved - they have to catch the excpetion, re-fetch the fresh data, and retry. So it trades "guaranteed success but maybe slow" for "always fast but occasionally has to redo work".

**The rule of thumb:**

* Optimistic locking → best when conflicts are rare and there's a long gap between read and write (web forms, edit pages)

* Pessimistic locking → best when conflicts are frequent, or the operation is short and critical enough that you'd rather block briefly than risk a retry-loop (e.g., a bank decrementing an account balance in a tight, fast transaction).

</details>

<details>
<summary>What is **Connection Pooling** (e.g., HikariCP), and why is it critical for Java applications instead of opening a new database connection for every request?</summary>

Connection pooling is a technique of reusing "pre-created" connections for running querys against the database, instead of creating new ones with each coming query, preventing exhausting resources and the overhead of creating and closing connections each time.

**Connection Pool, mechanically:**

At application startup, the pool creates a batch of real database connections upfront (say, 10) and holds them open, sitting idle, ready to go.

When your Java code needs to talk to the DB:

1. It asks the pool for a connection: `pool.getConnection()`
2. The pool hands over one of the already-open connections - no handshake, no auth, nothing. Just an existing connection.
3. Your code runs its query.
4. Instead of `conn.close()` actually killing the connection, it just gets returned to the pool, ready for the next request to borrow.

On the costs — you're right, and let's make it concrete:

* TCP handshake with the DB server (network round trip)
* Authentication (verifying credentials)
* Session setup on the database side (allocating memory, buffers on the DB server itself — this is often the part people forget: it's not just cost on your Java side, the database has to do work too)
* All before your actual SELECT or INSERT even runs

**Why HikariCP specifically** (the current default standard, and Spring Boot's default since Spring Boot 2):

* Extremely lightweight and fast compared to older pools like Apache DBCP or C3P0 - it's often cited as the fastest JDBC pool available.
* Config that matters in an interview:
    * `maximum-pool-size` — the cap on concurrent connections (too high can overwhelm the DB server; too low creates a bottleneck where requests queue waiting for a free connection).
    * `minimum-idle` — connections kept ready even when idle, so a traffic spike doesn't have to create new ones on demand.
    * `connection-timeout` — how long a request waits for a free connection before giving up and throwing an exception.

One sharp edge worth knowing: a common real bug in Java apps is a connection leak — code that borrows a connection (e.g., inside a `try` block) but never returns it to the pool (missing `finally`/try-with-resources). Eventually the pool exhausts all its connections, and every new request hangs waiting for one that never comes back. This is exactly why try-with-resources (`try (Connection conn = ...)`) is the standard pattern — it guarantees the connection is returned no matter what.

</details>

<details>
<summary>What are **Spring Transaction Propagation** types (e.g., `REQUIRED`, `REQUIRES_NEW`), and how do they control database transactions in a Java service?</summary>

**Transaction Propagation** is the rule that determines how a transactional method behaves when it's called from within another method that's already running inside a transaction.

In other words: when method B (annotated `@Transactional`) is called by method A (which is already inside a transaction), propagation answers the question: "Should B join A's existing transaction, start a completely new one, run without any transaction at all, or throw an error because a transaction context is required/forbidden?"

It's configured via the `propagation` attribute on the annotation:
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logOrderAttempt(Order order) { ... }
```

**Setup:** In a Spring service, you can annotate a method with @Transactional. Now imagine this scenario:

```java
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);
    inventoryService.updateStock(order); // this method is ALSO @Transactional
}
```

`REQUIRED` (the default): "Join the existing transaction if one exists; if not, start a new one." Since placeOrder already opened a transaction, updateStock just joins it — they become one atomic unit.

Now let's flip it, since this is where propagation gets interesting. Imagine a different scenario:

```java
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);
    auditLogService.logOrderAttempt(order); // logs "someone tried to place this order"
}
```

`REQUIRES_NEW`: "Always start a brand new, independent transaction — and if one is already running, suspend it while this new one executes."

What's correct: Yes — since logOrderAttempt runs in its own transaction, its failure is "just" a normal Java exception by the time it propagates back to placeOrder. It's not automatically tied to the outer transaction's fate the way it would be under REQUIRED. Your call to catch-and-handle (maybe retry, maybe just log-and-continue) is a completely valid design choice.
One correction/nuance: it's not automatic — Spring doesn't magically know "this failure is unimportant, swallow it." If placeOrder's code does nothing special, the exception thrown by logOrderAttempt will simply propagate up and normally fail placeOrder's transaction too (Spring's default behavior is: uncaught RuntimeException → rollback the transaction that's currently executing, which at that point is the outer one, since the inner one already finished).

</details>

## 6. Scaling & Distributed Databases (Enterprise Level)

<details>
<summary>What is the difference between **Vertical Scaling** (Scaling Up) and **Horizontal Scaling** (Scaling Out) for databases?</summary>

**Vertical Scalling** is about upgrading the hardware (more RAM, more CPU, faster disk) on a single machine where the database is running. It's simple (no architectural changes needed) but has a hard ceiling: there's a physical limit to how much a single machine can be upgraded, and high-end hardware gets disproportionately expensive.

**Horizontal Scalling** is about spreading the load across multiple machines, rather than making one machine bigger. In databases, this is done through two main techniques: **Replication** (copying the same data across multiple nodes to spread out read traffic) and **Sharding** (splitting the data itself into distinct pieces across multiple nodes to spread out storage and write traffic)

For SQL databases it's harder for them to scale horizontally, specifically because of joins and strict ACID guarantees across tables (RDBMS prioritizes consistency). NoSQL databases were often designed from the start around loose consistency (BASE) specifically to make horizontal scalling easy.

The harder a system enforces strict consistency across related data, the harder it is to spread that data across independent machines.

</details>

<details>
<summary>What is **Database Replication** (Master-Slave / Leader-Follower), and how does it improve read availability?</summary>

Database  Replication is the process of maintaning identical copies of the same dataset across multiple database nodes. The most common patter is Master-Slave (Leader-Follower): one node - the Leader - accepts all write operations (`INSERT`/`UPDATE`/`DELETE`). Those changes are then automatically propagated automatically propagated to one or more Follower nodes, which keep their data continously synchronized with the Leader.

Followers are typically restrictor to serving read traffic. This improves read availability in two ways:

1. **Load distribution** - Instead of every `SELECT` hitting one server, read traffic can be spread across many followers in parallel, which matters a lot since most applications are read-heavy.

2. **Fault tolerance** - if the Leader goes down, one of the followers can be promoted to take over (failover), so the system keeps serving reads (and, after promotion, writes) instead of going fully offline.

The trade-off: because data has to propagate from Leader to Followers, there's usually a small replication lag, meaning a Follower might briefly serve slightly stale data — a direct real-world example of the AP/eventual-consistency trade-offs from your CAP theorem answer.

</details>

<details>
<summary>What is **Database Sharding** (Horizontal Partitioning), and how does it help handle massive datasets?</summary>

Database Sharding is the practice of splitting a single logical dataset into multiple distinct, non-overlapping subsets called shards, with each shard stored on a separated database node. Unlike Replication, where every node holds a full copy of the same data, with Sharding no single node holds the entire dataset - each node only holds its slice, and all the shards together make up the whole database.

Which row goes to which shard is determined by the shard key (or partition key) - for example, a range of user ID's, or hash of customer ID designed to distribute rows evenly.

Sharding helps handle massive datasets because it distributes both storage and write load across many machines, rather than forcing one server to hold and process everything - something Replication alone doesn't solve, since every replica still has to storage 100% of the data. The trade-off is added complexity: queries that need data from multiple shards (e.g., a join across shards boundaries, or an aggregate across the whole dataset) become significantly harder, since the database can no longer answer them from a single machine alone.

</details>

<details>
<summary>What is a transaction?</summary>

A transaction is a group of one or more database operations executed as a single logical unit of work (all-or-nothing unit) - either every operation succeds and is made permanent (`commit()`), or if any part fails, all of them are undone (`rollback()`) - ensuring the database never ends up in a partially-updated inconsistend state.

</details>

<details>
<summary>What is a commit?</summary>



</details>