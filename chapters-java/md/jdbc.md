# JDBC

<details>
<summary>What is JDBC, and what are its benefits?</summary>

JDBC (Java Database Connectivity) is a standard Java API - not a framework - that defines a set of interfaces (`Connection`, `Statement`, `ResultSet`) for connecting and interacting with relational databases. JDBC itself only provides the contract; the actual implementation is supplied bu a driver, a vendor-specific JAR (e.g., for PostgreSQL, MySQL, Oracle) that knows how to speak that database's native protocol. This separation gives developers three key benefits: **database portability**, since switching databases often means swapping the driver with minimal changes to application code; a **consistent**, **unified**, **API**, so once you know how to use `Connection`, `Statement`, and `ResultSet`, you can work with any databse that has JDBC  drives; and **platform independence**, because JDBC is pure Java shipped as part of the JDK, unlike alternatives like ODBC that depend on native, OS-specific libraries.

</details>

<details>
<summary>JDBC architecture </summary>

1. `DriverManager` - the "factory"/registry. You give it a JDBC URL (e.g., `jdbc:postgresql://localhost:5432/mydb`), and figures out which registered driver understands that URL, delegates to it, and returns a `Connection`.

```java
Connection conn = DriverManager.getConnection(url, user, password);
```

It's the entry point of raw JDBC. Its job is simple: manage the list of registered drivers and hand you back a `Connection` when you ask for one.

2. `Driver` - the vendor implementation. It registers itself with `DriverManager` (usually automatically nowadays) so `DriverManager` knows it exists and whar URL prefixes it handles.

3. `Connection` - represents your live session with the database. From it, you create the objects that actually run SQL.

4. `Statement` / `PreparedStatement` / `CallableStatement` - created from the `Connection`, these are what execute your SQL, (plain queries, parameterized queries, and stored procedure calls, respectively).

5. `ResultSet` - what you get back after executing a `SELECT`; It's a cursor over the rows returned, which you iterate to pull out data.

6. `SQLException` - the checked exception JDBC throws for basically anything that goes wrong (connection failure, bad SQL, constraint violation, etc).

The flow: `DriverManager` (using a `Driver`) -> gives you a `Connection` -> which creates a `Statement` -> which executes SQL -> and returns a `ResultSet`.

</details>

<details>
<summary>Main JDBC interfaces/classes</summary>

The core JDBC interfaces are:

`DriverManager` is the entry point - given a JDBC URL, it locates the right registered `Driver` (the vendor-specific implementation for a specific database) and returns a `Connection`, you create one of three statements types to execute SQL: `Statement` for plain static queries, `PreparedStatements` for parameterized queries, and `CallableStatement` for calling stored procedures. Executing a query returns a `ResultSet`, a cursor you iterate to read the returned rows. Finally, `SQLException` is the checked exception JDBC throws whenever something goes wrong along that chain - connection failures, invalid SQL, or constraint violations.

</details>

<details>
<summary>What are the steps to connect to a database and run a query using raw JDBC?</summary>

Once i know what database i'm using, i add the driver on the `pom.xml`; then i need to define the JDBC URL; then using `DriverManager` i get a `Connection`, passing the url, user and password; then i use the `Connection` to create an `Statement`, lets say a normal `Statement`, and use it to run the SQL; after running the SQL i'll get a ResultSet that iterate through to pull out the data; and close the connection once the job is done.

```java
String url = "jdbc:postgresql://localhost:5432/mydb";
String user = "postgres";
String password = "secret";

try (Connection conn = DriverManager.getConnection(url, user, password);
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("SELECT id, name FROM employees")) {

    while (rs.next()) {
        int id = rs.getInt("id");
        String name = rs.getString("name");
        System.out.println(id + " - " + name);
    }

} catch (SQLException e) {
    e.printStackTrace();
}
```
* `try-with-resources` — `Connection`, `Statement`, and `ResultSet` all implement `AutoCloseable`, so they get closed automatically (in reverse order) even if an exception is thrown. This is the modern, idiomatic way to write this — no manual `finally { conn.close(); }` blocks.

* `SQLException` is checked — you're forced to either catch it or declare `throws SQLException`, which is why you see it everywhere in JDBC code.

* Note the driver itself doesn't need to be manually loaded with `Class.forName(...)` anymore (that was required pre-Java 6) — since JDBC 4.0, drivers on the classpath register themselves automatically via the `ServiceLoader` mechanism. Worth mentioning if asked "how do you register a driver" (question #5 on our list) — I'll flag it again when we get there.

</details>

<details>
<summary>How does a JDBC driver get registered, and what's the difference between the old way and the modern way?</summary>

Driver registration in JDBC relies on a static initializer block: every JDBC driver class registers an instance of itself with `DriverManager` inside a `static {}` block, which runs the moment the class is loaded - no instantiation required. In the old pre-JDBC 4.0 approach, developers trigered this manually with `Class.forName("com.mysql.cj.jdbc.Driver")`, forcing the class to load and its static method to fire. Since JDBC 4.0 (Java 6+), this is automatic: driver JARs include a `META-INF/services/java.sql.Driver` file listing the driver class, and the JVM's `ServiceLoader` mechanism scans the classpath at startup and loads any driver it finds - triggering the same static registration, but without any explicit code from the developer.

```java
public class Driver implements java.sql.Driver {
    static {
        DriverManager.registerDriver(new Driver());
    }
}
```

</details>

<details>
<summary>What are the driver types (Type 1–4)?</summary>

JDBC drivers come in four types, distinguished by how much native (non-java) code they rely on and how directly they communicate with the data base. **Type 1**, the JDBC-ODBC bridge, translates JDBC calls into ODBC calls and relies on native ODBC drivers - it's obsolote and was removed from the JDK in Java 8. **Type 2**, the Native-API driver, uses vendor-provided native clients libraries, offering better performance than Type 1 but still requiring platform-specific binaries. **Type 3**, the Network protocol driver, is pure on the java client but routes requests through a middleware server that translates them for the database - rarely used today due to added complexity and network hop. **Type 4**, the Thin Driver, is 100% Java and communicates directly with the database using its native network protocol, with no bridge or middleware required. Type 4 is what virtually all modern JDBC drives (PostgreSQL, MySQL, etc.) use, since it fully delivers on JDCB's promise of being pure Java and platform-indepentdent.

</details>

<details>
<summary>`Statement` vs `PreparedStatement`</summary>

`Statement` is used to execute static SQL with no parameters, the full SQL text is sent as a plain string, which means any user input has to be manually concatenated into that string, `PreparedStatement` instead sends a precompliked SQL template containing `?` placeholders, and values are bound to those placeholders separately, afterward. This distinction matters for two reasons. First, **security**: because the SQL structure is locked in before any data is attached, malicious input like `'OR '1'= 1` can never alter the query's logic - the database always treats bound values as literal data, not executable SQL, which is what prevents SQL injection. Second, **performance**: since the query is compiled once, executing the same `PreparedStatment` repeatedly lets the database reuse its execution plan instead of re-parsing identical SQL each time. For any query involving user input or repeated execution, `PreparedStatement` is the standard, safer choince.

```java
String sql = "SELECT * FROM users WHERE username = ? AND password = ?";

try (Connection conn = DriverManager.getConnection(url, user, password);
     PreparedStatement ps = conn.prepareStatement(sql)) {

    ps.setString(1, userInput);
    ps.setString(2, passwordInput);

    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("username"));
        }
    }
}
```

</details>

<details>
<summary>`CallableStatement` / stored procedures</summary>

`CallableStatement` is used to invoke stored procedures and extends `PreparedStatement`, using the JDBC escape syntax `{call procedureName(?, ?)}`. Its parameters can be IN (passed into the procedure), OUT (returned from the procedure), or INOUT (both). Since it extends `PreparedStatement`, IN and INOUT parameters are set using the same inherited setters like `setString()` or `setInt()` - there's nothing special about them. What's unique to `CallableStatement` is `registerOutParameter(index, sqlType)`, which must be called before execution to tell the driver to reserve that slot and expect a value back; after execution, that value is read using the corresponding getter, `like getBigDecimal(index)`.

```java
String sql = "{call calculateTotal(?, ?)}"; // param 1 = IN, param 2 = OUT

try (Connection conn = DriverManager.getConnection(url, user, password);
     CallableStatement cs = conn.prepareCall(sql)) {

    cs.setInt(1, orderId);                          // IN: send order ID in
    cs.registerOutParameter(2, Types.DECIMAL);       // OUT: expect a value back here

    cs.execute();

    BigDecimal total = cs.getBigDecimal(2);          // now read the OUT value
    System.out.println("Total: " + total);
}
```

</details>

<details>
<summary>`ResultSet` basics (get it, check if empty, common methods)</summary>

`ResultSet` represents a cursor over the rows returned by the `SELECT` query, obtained by calling `executeQuery(sql)`on a `Statement` or `PreparedStatement` (as oposed to `executeUpdate(sql)`), which is used for `INSERT`/`UPDATE`/`DELETE` and returns an int representing the number of affected rows. A `ResultSet` is never null afeter a succesful query - even zero matching rows produce a valid, empty `ResultSet`. It's initially positioned before the first row, so `rs.next()` must be called to advance the cursor; it returns `true` if a row was found or `false` if there are no more rows. This makes `rs.next()` the standard way to both check for data and iterate - used in a `while` loop for multiple rows, or in an `if` statement when only a single row is expected.

```java
// Iterating multiple rows
while (rs.next()) {
    String name = rs.getString("name");
    // process each row
}

// Checking/getting a single expected row
if (rs.next()) {
    String name = rs.getString("name");
} else {
    // no matching row found
}
```

</details>

<details>
<summary>Closing a connection properly </summary>

Closing a JDBC connection matters because connections are expensive to stablish - opening one involves a TCP handshake, authentication, and session setup on the database server itself, all before any actual query runs. In a typical production setup using a connection pool (like HikariCP), `conn.close()` doesn't actually destroy that underlying connection, it simply returns it to the pool for reuse. This makes forgetting to close a connection specially dangerous: it causes a **connection leak**, where borrowed connections never make it back to the pool, and eventually the pool has none left to hand out - leaving every new request hanging. The standard wayy to guarantee a connection (and any `Statement` or `ResultSet` opened from it) always get closed, even if an exception occurs mid-query, is `try-with-resources`, since these classes implement `AutoCloseable`.

</details>

<details>
<summary>`commit()` / `rollback()` / `setAutoCommit()` </summary>

`commit()` makes changes of a transaction permanent, `rollback()` undoes uncommited changes back to the last commit point (or transaction start).

By default, a JDBC `Connection` has `autoCommit` set to true, meaning every individual statement is commited immediately and automatically as its own transaction. To group multiple statements into a single atomic transaction, you must first call `conn.setAutoCommit(false)`, which diables this automatic behavior. From that point on, changes remain uncommited until you explicitly call `conn.commit()`, making them permanent, or `conn.rollback()`, undoing everything back to the start of the transaction- typically triggered inside a `catch` block when something falls partway through. This is the JDBC-level mechaism for enforcing Atomicity: operations like debiting one account and crediting another can be wrapped so that either both succed or neither does. 

```java
try (Connection conn = DriverManager.getConnection(url, user, password)) {
    conn.setAutoCommit(false);  // start a manual transaction

    try (PreparedStatement debit = conn.prepareStatement(
            "UPDATE accounts SET balance = balance - ? WHERE id = ?");
         PreparedStatement credit = conn.prepareStatement(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?")) {

        debit.setBigDecimal(1, amount);
        debit.setLong(2, accountA);
        debit.executeUpdate();

        credit.setBigDecimal(1, amount);
        credit.setLong(2, accountB);
        credit.executeUpdate();

        conn.commit();  // both succeeded → make it permanent

    } catch (SQLException e) {
        conn.rollback();  // something failed → undo both
        throw e;
    }
}
```

</details>

<details>
<summary>JDBC URL format</summary>

A JDBC URL follows the format `jdbc:<subprotocol>://<host>:<port>/database`, for example `jdbc:postgresql://localhost:5432/my-db`. The `jdbc:` prefix identifies it as a JDBC connection string, and the subprotocol (e.g., `postgresql`, `mysql`) tells `DriverManager` which registered driver should handle this connection - each driver "claims" its subprotocol and matches incoming URLs against it. Everything after the subprotocol (host, port, databse name) is defined by the individual driver vendor rather than JDBC itself, which is why formats can vary noticiably between databases - Oracle's format, for instance look quite different (`jdbc:oracle:thin:@host:port:sid`) from PostgreSQL's or MySQL's.

</details>

<details>
<summary>Retrieving a generated Primary Key (e.g., after an INSERT where the ID is auto-generated by the database)</summary>

To retrieve a database-generated primary key after an `INSERT` (e.g., an auto-increment ID), you can't rely on `executeUpdate()`'s return value, since that only gives you the count of affected rows. Instead, you flag the  intent when creating the statement, using the overload `conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEY)`. After calling `executeUpdate()` as usual, you call `ps.getGeneratedKeys()`, which returns a resultset containing the generated key - retrieved the same way as any other result, by calling `rs.next()` and then a getter like `rs.getLong(1)`.

```java
String sql = "INSERT INTO employees (name) VALUES (?)";

try (PreparedStatement ps = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
    ps.setString(1, "Alice");
    ps.executeUpdate();

    try (ResultSet rs = ps.getGeneratedKeys()) {
        if (rs.next()) {
            long newId = rs.getLong(1);
            System.out.println("New employee ID: " + newId);
        }
    }
}
```

</details>

<details>
<summary>Scrollable ResultSet</summary>

By default, a JDBC `ResultSet` is `TYPE_FORWARD_ONLY` - the cursor can only move forward via `rs.next()`, with no way to go back. A **scrollable** `ResultSet`, created by passing extra type/concurrency constants to `createStatement()` (e.g., `ResultSet.TYPE_SCROLL_INSENTIVE`, `ResultSet.CONCUR_READ_ONLY`), allows free cursor movement - `rs.previous()`, `rs.first()`, `rs.last()`, `rs.absolute(n)` - instead of only forward iteration. `TYPE_SCROLL_INSENSITIVE` takes a snapshot at query time and won't reflect changes made by other transactions afterward, while the rarely-used `TYPE_SCROLL_SENSITIVE` can reflect live updates as you scroll. The concurrency mode (`CONCUR_READ_ONLY` vs `CONCUR_UPDATABLE`) separately controls whether the `ResultSet` itself can be modified in place.

```java
Statement stmt = conn.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_READ_ONLY
);
ResultSet rs = stmt.executeQuery("SELECT id, name FROM employees");

rs.last();                    // jump to the last row
int totalRows = rs.getRow();  // get current row number
rs.first();                   // jump back to the first row
rs.previous();                // move backward
rs.absolute(5);               // jump directly to row 5
```

</details>

<details>
<summary>What is a `Savepoint`?</summary>

A `Savepoint` is a named marker placed within an ongoing transaction, created with `conn.setSavepoint("name")`, that allows a partial rollback. While a plain `rollback()` undoes the entire transaction back to its start, `rollback(savepoint)` undoes only the changes made after that marker, leaving everything before it inteact and still part of the same open, uncommited transaction. This is useful in multi-step transactions where a later step might fall but earlier steps should be preserved - for example, keeping an order insert intact even if applying a discount afterward fails - without needing to redo the entire transaction from scratch. The transaction strill isn't permanent until a final `commit()` is called.

</details>

<details>
<summary>Does commit() closes a transaction?</summary>

Yes - `commit()` ends the current transaction. All the changes made since the last `commit()` (or since the transaction began) become permanent, and any locks held by that transaction are released.

But there is a subtlety: If `autoCommit` is still `false`, the `Connection` doesn't just "stop" after that - a new transaction implicitly begins right away, ready for whatever statements you run next. You don't need to call anything to "start" it again; it just continues in manual-commit mode until you either `commit()` or `rollback()` again.

So the lifecycle looks like: `setAutoCommit(false)` -> [statements] -> `commit()` (transaction #1 ends, transaction #2 silently begins) -> [more statements] -> `commit()` (transaction #2 ends, transaction #3 begins) -> ... and so on, until you either close the connection or flip `autoCommit` back to `true`.

</details>

<details>
<summary>What are `DatabaseMetaData` / `ResultSetMetaData`?</summary>

`ResultSetMetaData`, obtainedvia `rs.getMetaData()`, provides structural information about a query's results at runtime - column count, column names, and column types - without needing to know the query's shape in advance. This is essential for writing generic code that processes arbitrary queries, such as database viewer tool or generic CVS export utility, sice it allows iterating over columns dynamicallly (`meta.getColumnCount()`, `meta.getColumnName(i)`) rather than hardcoding column names.

```java
ResultSet rs = stmt.executeQuery("SELECT * FROM employees");
ResultSetMetaData meta = rs.getMetaData();

int columnCount = meta.getColumnCount();

for (int i = 1; i <= columnCount; i++) {  // JDBC columns are 1-indexed, like ResultSet
    String columnName = meta.getColumnName(i);
    String columnType = meta.getColumnTypeName(i);
    System.out.println(columnName + " : " + columnType);
}

// Now you can print any row generically, without knowing column names in advance
while (rs.next()) {
    for (int i = 1; i <= columnCount; i++) {
        System.out.print(rs.getObject(i) + "\t");
    }
    System.out.println();
}
```

`DatabaseMetaData`, obtained via `conn.getMetaData()`, provides the same kind of structural insight one level up about the database itself - available tables, primary keys, supported data types, and driver/database version info.

</details>

<details>
<summary></summary>

Batch updates solve the performance cost of repeated network round trips when executing many similar statemetns (e.g., inserting 1000 rows). Instead of calling `executeUpdate()` separately for each one - which mean one network trip to the database per statement (send SQL, wait for the DB to respond and repeat) - you use ps.`addBatch()` to queue each set of bound parameters locally on the client side, without hitting the dabase yet. Once all statements are queued, a single call to `ps.executeBatch()` sends them all to the database in one network trip, returning an `int[]` with affected-row count or each statement. Batching is typically paired with `setAutoCommit(false)` and a final `commit()`, both to preserve atomicity across the batch and to avoid undermining the performance benefit with per-statement auto-commits.

</details>