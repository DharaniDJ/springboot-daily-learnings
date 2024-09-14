# Transactional Annotation in Spring Boot

## Transaction Manager and its Hierarchy

In Spring Boot, transaction management is a critical aspect of ensuring data integrity and consistency. The `PlatformTransactionManager` interface is the central interface in Spring's transaction infrastructure. It provides a way to programmatically manage transactions. Spring Boot provides several implementations of this interface, such as:

- `DataSourceTransactionManager`: Manages transactions for a single JDBC `DataSource`.
- `JpaTransactionManager`: Manages transactions for JPA (Java Persistence API) EntityManagerFactory.
- `JtaTransactionManager`: Manages transactions for JTA (Java Transaction API) EntityManagerFactory. JTA manages Distributed Transactions.
- `HibernateTransactionManager`: Manages transactions for a Hibernate SessionFactory.

The hierarchy of transaction managers allows Spring Boot to support different types of transactional resources, ensuring that the appropriate transaction manager is used based on the context.

![transaction-managers-hierarchy](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/transaction-managers-hierarchy.png)

## Types of Transactional Management

![transaction-management-types](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/transaction-management-types.png)

## Declarative Approach of Transaction Management

The declarative approach to transaction management in Spring Boot is achieved using annotations. The `@Transactional` annotation is the most commonly used annotation for this purpose. It can be applied at the class or method level. When applied, it ensures that the annotated method or all methods within the annotated class are executed within a transactional context.

Example:
```java
@Service
public class MyService {

    @Transactional
    public void performTransaction() {
        // business logic here
    }
}
```

In this example, the `performTransaction` method is executed within a transaction. If any exception occurs, the transaction will be rolled back.

## Programmatic Approach of Transaction Management (Transaction Template)

The programmatic approach involves using the `TransactionTemplate` class. This approach provides more control over transaction management compared to the declarative approach.

Example:
```java
@Service
public class MyService {

    private final TransactionTemplate transactionTemplate;

    @Autowired
    public MyService(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }

    public void performTransaction() {
        transactionTemplate.execute(status -> {
            // business logic here
            return null;
        });
    }
}
```

In this example, the `performTransaction` method uses the `TransactionTemplate` to execute business logic within a transaction. The `execute` method takes a `TransactionCallback` which contains the code to be executed within the transaction.

## Different Types of Propagation

Propagation defines how transactions relate to each other. Spring supports several types of propagation:

### REQUIRED Propagation

The `REQUIRED` propagation is the default propagation type. It means that the method must run within a transaction. If a transaction already exists, the method will run within that transaction. If no transaction exists, a new one will be created.

### REQUIRES_NEW Propagation

The `REQUIRES_NEW` propagation means that a new transaction will always be started, and if an existing transaction is running, it will be suspended.

### SUPPORTS Propagation

The `SUPPORTS` propagation means that the method will run within a transaction if one exists. If no transaction exists, the method will run without a transaction.

### NOT_SUPPORTED Propagation

The `NOT_SUPPORTED` propagation means that the method should not run within a transaction. If a transaction exists, it will be suspended.

### MANDATORY Propagation

The `MANDATORY` propagation means that the method must run within an existing transaction. If no transaction exists, an exception will be thrown.

### NEVER Propagation

The `NEVER` propagation means that the method should not run within a transaction. If a transaction exists, an exception will be thrown.

By understanding and using these propagation types, you can control the transactional behavior of your methods more precisely, ensuring that your application's data integrity and consistency are maintained.
```