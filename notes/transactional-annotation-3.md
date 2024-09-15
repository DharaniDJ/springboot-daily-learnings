# Transactional Annotation in Spring Boot

## Isolation Level and its different types

## Isolation Definition

Isolation in database systems defines how transaction integrity is visible to other transactions and systems. It determines how and when the changes made by one operation become visible to other concurrent operations. The isolation level controls the degree of locking and the visibility of changes in a transactional system.

![isolation-level](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/isolation-level.png)

Default isolation level, depends upon the DB which we are using.
Like most relational Databases uses `READ_COMMITTED` as default isolation, but again it depends upon DB to DB.

## Dirty Read Problem

A dirty read occurs when a transaction reads data that has been written by another transaction that has not yet been committed. This can lead to inconsistencies if the other transaction is rolled back.

Example:
- Transaction A updates a record.
- Transaction B reads the updated record before Transaction A commits.
- Transaction A rolls back, leaving Transaction B with invalid data.

![dirty-read](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/dirty-read.png)

## Non-Repeatable Read Problem

A non-repeatable read occurs when a transaction reads the same row twice and gets different data each time. This happens because another transaction has modified the data between the two reads.

Example:
- Transaction A reads a record.
- Transaction B updates the same record and commits.
- Transaction A reads the record again and sees the updated data.

![non-repeatable-read](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/non-repeatable-read.png)

## Phantom Read Problem

A phantom read occurs when a transaction reads a set of rows that match a condition, and another transaction inserts or deletes rows that match the same condition. This causes the first transaction to see a different set of rows if it re-executes the query.

Example:
- Transaction A reads rows that match a condition.
- Transaction B inserts a new row that matches the condition and commits.
- Transaction A re-executes the query and sees the new row.

![phantom-read](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/phantom-read.png)

## DB Locking Type (Shared and Exclusive Locks)

### Shared Locks

Shared locks allow multiple transactions to read a resource but not modify it. They are used to prevent dirty reads and ensure that data being read is not changed by other transactions.

### Exclusive Locks

Exclusive locks prevent other transactions from reading or modifying a resource. They are used to ensure that a transaction has exclusive access to a resource, preventing dirty reads, non-repeatable reads, and phantom reads.

![db-locking-types](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/db-locking-types.png)

## READ_UNCOMMITTED Isolation Level

The `READ_UNCOMMITTED` isolation level allows transactions to read uncommitted changes made by other transactions. This level provides the least isolation and can lead to dirty reads.

Example:
```java
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public void performTransaction() {
    // business logic here
}
```

Example Scenario:

Suppose you have a database table called Accounts with columns AccountID and Balance.

Transaction A:

	1.	Begins.
	2.	Reads Balance from AccountID = 101 and sees $500.

Transaction B:

	1.	Begins.
	2.	Updates Balance of AccountID = 101 to $600 but does not commit immediately.

Transaction A:

	1.	Reads Balance of AccountID = 101 again.
	2.	Sees the uncommitted balance of $600, illustrating a dirty read.

Transaction B:

	1.	Decides to rollback the transaction due to some error.

Transaction A:

	1.	If it reads AccountID = 101 again, it might now see the original balance of $500, leading to inconsistency in the view of data across different points in time.

Effects Illustrated:

	•	Dirty Reads: Transaction A sees the changes made by Transaction B before those changes are committed, which can lead to problems if B rolls back its changes.
	•	Non-Repeatable Reads: Transaction A sees a different value for the same data within the same transaction because it can read uncommitted data that is being changed by other transactions.
	•	Phantom Reads: If Transaction B adds a new account and does not commit, Transaction A can still see this new account if it queries the table before B commits or rolls back.

Summary:

Read Uncommitted is used in scenarios where the need for the most current data is not critical and performance is prioritized over accuracy. This isolation level can be useful in situations where data is mostly read and rarely updated, or where some inconsistency is tolerable, such as with real-time data analytics where speed is more critical than precision. However, it’s generally not recommended for most business applications due to the potential for data anomalies and inconsistencies.

This Isolation level is only used when you have `only read` application.

## READ_COMMITTED Isolation Level

The `READ_COMMITTED` isolation level ensures that a transaction can only read changes that have been committed by other transactions. This level prevents dirty reads but allows non-repeatable reads and phantom reads.

Example:
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void performTransaction() {
    // business logic here
}
```

Example Scenario:

Suppose you have a database table called Orders with a column Status.

Transaction A:

	1.	Begins.
	2.	Reads an Order with OrderID = 123 and sees Status = Pending.
	3.	Does some other work, taking some time.

Transaction B:

	1.	Begins.
	2.	Updates OrderID = 123 from Status = Pending to Status = Confirmed.
	3.	Commits the change.

Transaction A:

	1.	Reads OrderID = 123 again.
	2.	Now sees Status = Confirmed, different from the first read, illustrating a non-repeatable read.

Preventing Dirty Reads:

	•	During its operation, if Transaction A tries to read any data being modified by Transaction B before B has committed, it will not see the changes. This prevents Transaction A from seeing any uncommitted, potentially unreliable data (dirty reads).

Allowing Non-Repeatable and Phantom Reads:

	•	Since Transaction A can see changes made by Transaction B once they are committed, if A reads the same data multiple times, it might see different values, hence, non-repeatable reads.
	•	If Transaction B adds a new order (say OrderID = 124) and commits while Transaction A is still running, and then Transaction A queries all orders, it will see the new order, demonstrating a phantom read.

Summary:

Read Committed provides a level of isolation that prevents transactions from reading uncommitted data (dirty reads) but does not isolate a transaction from the effects of committed changes by other transactions, thus allowing non-repeatable reads and phantom reads. This level is often used as a default in many database systems like SQL Server and PostgreSQL because it offers a good trade-off between data consistency and system performance, allowing higher concurrency.

## REPEATABLE_READ Isolation Level

The `REPEATABLE_READ` isolation level ensures that if a transaction reads a row, it will see the same data if it reads that row again, even if other transactions modify the data. This level prevents dirty reads and non-repeatable reads but allows phantom reads.

Example:
```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void performTransaction() {
    // business logic here
}
```

Example Scenario:

Suppose you have a database table called Products with columns ProductID and Stock.

Transaction A:

	1.	Begins.
	2.	Reads Stock for ProductID = 200 and sees 150 units.

Transaction B:

	1.	Begins.
	2.	Attempts to update Stock of ProductID = 200 to 130 units. This update is blocked until Transaction A completes because A has already read the ProductID = 200.

Transaction A:

	1.	Reads Stock for ProductID = 200 again.
	2.	Still sees 150 units, confirming a repeatable read.

Transaction B:

	1.	Continues to wait or is blocked until Transaction A commits or rolls back.

Transaction A:

	1.	Commits.

Transaction B:

	1.	Now proceeds with the update, changing Stock to 130 units.
	2.	Commits the change.

Transaction A:

	1.	If it were to start again and read ProductID = 200, it would see the new stock of 130 units, but during its original operation, it saw a consistent and repeatable view.

Handling Phantom Reads:

	•	Transaction A also performs a range query to count products with Stock above 100.
	•	Transaction C adds a new product with Stock = 120 and commits.
	•	If Transaction A repeats the range query, it might now count the new product, thus encountering a phantom read.

Summary:

Repeatable Read greatly reduces anomalies in transaction processing by ensuring that any data read during a transaction cannot be changed by other transactions until the first transaction is complete. However, it does not entirely prevent all forms of inconsistency, such as phantom reads, where new rows can still alter the results of a query if they are committed during the transaction. This level of isolation is higher than Read Committed and is often used when consistency is more critical, though it can lead to lower concurrency due to more restrictive locking.

## SERIALIZABLE Isolation Level

The `SERIALIZABLE` isolation level provides the highest level of isolation. It ensures that transactions are executed in a way that they appear to be serialized, one after the other. This level prevents dirty reads, non-repeatable reads, and phantom reads but can lead to reduced concurrency and performance.

Example:
```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void performTransaction() {
    // business logic here
}
```

Example Scenario:

Suppose you have a database table called Employees with columns EmployeeID and Salary.

Transaction A:

	1.	Begins.
	2.	Queries the sum of salaries for all employees whose salaries are above $50,000.

Transaction B:

	1.	Begins.
	2.	Tries to insert a new employee record with a Salary of $55,000 or update an existing employee’s salary to be above $50,000.

In a Serializable environment:

	•	Transaction B is prevented from making changes to salaries (either through inserts or updates that would affect the result of Transaction A’s query) until Transaction A completes. This ensures that the sum of salaries calculated by Transaction A remains consistent even if queried multiple times.

Transaction A:

	1.	Completes and commits, finding that the total sum of salaries above $50,000 is, for example, $1,000,000.

Transaction B:

	1.	Now allowed to proceed.
	2.	Commits the insertion or update.

Prevention of Anomalies:

	•	Dirty Reads: If Transaction B hasn’t committed yet, Transaction A won’t see B’s changes.
	•	Non-Repeatable Reads: Transaction A can run the same query multiple times before it completes and will get the same result every time, as Transaction B’s changes are blocked until A completes.
	•	Phantom Reads: Even if Transaction B wants to add new rows that would affect A’s query, these changes are not visible to Transaction A until B commits after A has finished, ensuring no phantom data affects the consistency of A’s query results.

Summary:

The Serializable isolation level provides the strongest guarantees of consistency by ensuring that transactions are completely isolated from one another, as though they are being processed one at a time. This isolation comes at the potential cost of performance due to increased locking and reduced concurrency. However, it is crucial for scenarios where data integrity and consistency are paramount, such as in financial transactions or any application where even the smallest data anomaly cannot be tolerated.

![locking-strategy](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/locking-strategy.png)

By understanding and using these isolation levels, you can control the visibility and consistency of data in your transactional system, ensuring that your application's data integrity is maintained.
