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

To specifically mention which `TransactionManager` to use, we need to create a bean of `PlatformTransactionManager` in `AppConfig` in which we create an object of our need.

### AppConfig.java
```java
@Configuration
public class AppConfig {
    @Bean
    public DataSource datasource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:testdb");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }

    @Bean
    public PlatformTransactionManager userTransactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);    // Specifically tell by creating a bean of a platform transaction manager and create an object of whatever you have to work with 
    }
}
```

```java
@Component
public class UserDeclarative {
    // Spring boot will try to find a bean with name userTransactionManager
    @Transactional(transactionManager = "userTransactionManager")
    public void updateUserProgrammatic() {
        // SOME DB OPERATIONS
        System.out.println("Insert Query ran");
        System.out.println("Update Query ran");
    }
}
```

## Programmatic Approach of Transaction Management (Transaction Template)

The programmatic approach involves using the `TransactionTemplate` class. This approach provides more control over transaction management compared to the declarative approach.

- Transaction Management through code
- Flexible but difficult to maintain. (It is difficult to use `Transaction` at 100 places, let's say)

Why Flexible? Lets consider the below code:

```java
@Component
public class User {

    @Transactional
    public void updateUser(){
        //1. update DB
        //2. External API call
        //3. update DB
    }
}
```

In this example, we have `updateUser` method. we have 3 operations, 1st update DB is initial set of DB operations and after once this initial operations are success, we need to make some external API call. And after this external API call, I need to update my DB again. Now if we put `@Transactional` at the top of this method, it will cause an issue. Because `External API call` is involved here, let's say this call is taking some time(3-4 seconds may be). Now we are holding the DB connections for that time. And at the time of peak traffic where we have to open a large number of DB connections to update it. This `External API call` could be bottleneck, because these calls are the time taking process and because of this our `DB connection` is in hold for that amount of time. we cannot remove the call because of the dependency, we have to keep them together.

![programmatic-approaches](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/programmatic-approaches.png)

## Different Types of Propagation

Propagation defines how transactions relate to each other. When we try to create a new Transaction, it first check the PROPAGATION value set, and this tell whether we have to create new transaction or not.

Spring supports several types of propagation:

### REQUIRED Propagation

The `REQUIRED` propagation is the default propagation type. It means that the method must run within a transaction. If a transaction already exists, the method will run within that transaction. If no transaction exists, a new one will be created.

![required-propagation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/required-propagation.png)

### REQUIRES_NEW Propagation

The `REQUIRES_NEW` propagation means that a new transaction will always be started, and if an existing transaction is running, it will be suspended.

![required-new-propagation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/required-new-propagation.png)

### SUPPORTS Propagation

The `SUPPORTS` propagation means that the method will run within a transaction if one exists. If no transaction exists, the method will run without a transaction.

![supports-propagation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/supports-propagation.png)

### NOT_SUPPORTED Propagation

The `NOT_SUPPORTED` propagation means that the method should not run within a transaction. If a transaction exists, it will be suspended.

![not-supported-propagation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/not-supported-propagation.png)

### MANDATORY Propagation

The `MANDATORY` propagation means that the method must run within an existing transaction. If no transaction exists, an exception will be thrown.

![mandatory-propagation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/mandatory-propagation.png)

### NEVER Propagation

The `NEVER` propagation means that the method should not run within a transaction. If a transaction exists, an exception will be thrown.

![never-propagation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/never-propagation.png)

By understanding and using these propagation types, you can control the transactional behavior of your methods more precisely, ensuring that your application's data integrity and consistency are maintained.
```