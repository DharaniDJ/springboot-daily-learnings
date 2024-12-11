# JPA (Java Persistence API) Guide

## 1. Introduction about ORM

Object-Relational Mapping (ORM) is a technique that allows developers to map object-oriented domain models to relational databases. ORM frameworks like JPA (Java Persistence API) provide a way to interact with databases using Java objects, abstracting the complexities of SQL and database interactions. This approach enhances productivity and maintains a clean separation between the data access layer and business logic.

## 2. Setup of JPA Happy Flow

We will see what all things are required to run one single program using JPA.

### Dependencies

To set up JPA in a Spring Boot application, add the following dependencies to your `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

### Configuration

Configure the database connection in `application.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
```

### Entity Class

Create an entity class annotated with `@Entity`:

```java
@Entity
public class UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Constructors
    public UserDetails(){

    }

    public UserDetails(String name, String email){
        this.name = name;
        this.email = email;
    }

    // Getters and Setters
}
```

### Repository Interface

Create a repository interface extending `JpaRepository`:

```java
public interface UserDetailsRepository extends JpaRepository<UserDetails, Long> {
}
```

It exposes APIs to insert, fetch, modify, etc...

### Service and Controller

Create a service and controller to handle business logic and HTTP requests:

```java
@Service
public class UserDetailsService {
    @Autowired
    private UserDetailsRepository userDetailsRepository;

    public List<UserDetails> getAllUsers() {
        return UserDetailsRepository.findAll();
    }

    public UserDetails saveUser(UserDetails user) {
        return UserDetailsRepository.save(user);
    }
}
```

```java
@RestController
@RequestMapping("/api/")
public class UserController {
    @Autowired
    private UserDetailsService userDetailsService;

    @GetMapping(path = "/test-jpa")
    public List<UserDetails> getUsers() {
        UserDetails userDetails = new UserDetails("xyx", "xyz@conceptandcoding.com");
        userDetailsService.saveUser(userDetails);
        return userDetailsService.getAllUser();
    }
}
```

### Output
![output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/output.png)

To enable the console, we can add, below two properties in "application.properties"
```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

You can open the console in `http://localhost:8080/h2-console`. This looks very simple, but what's happening inside?

![h2-console](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/h2-console.png)

## 3. Architecture of JPA

JPA architecture consists of several key components:

- **Entity**: Represents a table in the database.
- **EntityManager**: Interface used to interact with the persistence context.
- **EntityManagerFactory**: Factory class for `EntityManager` instances.
- **Persistence Unit**: A set of all entity classes that are managed by `EntityManager` in an application.
- **Persistence Context**: A set of entity instances in which for any persistent entity identity, there is a unique entity instance.

![JPAArchitecture](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JPAArchitecture.png)

- Persistence unit is where we provide our configurations like `DB Connections`, `what dialect, driver we want to use`. So it is basically `application.properties`. So whenever you start an application, spring framework uses that persistance unit and it creates an object of `EntityManagerFactory` (Kind of Java Object). Now this EntityManagerFactor uses to create `EntityManager`.

- For one persistence unit, you have one object of `EntityManagerFactory`. If you have more that one persistence unit, the equivalent number of factories would be created. **If your application has more that one DB, then it is advisable not to use `application.properties`. Use `@Configuration` class and provide both the persistence unit or configuration properly and connect to a proper DB.**

- Whenever you required an `EntityManager` object, it goes to the factory and get one. But what is `EntityManager`? You can say it's a session in hibernate. In JPA terms it's called as `EntityManager`. `EntityManager` exposes APIs like persist, query, insert ...etc. `JPARepository` internally connects with `EntityManager` within its implementation. `Hibernate` provided an implementation for this JPARepository, we have a class called `session impl` class.

- Each `EntityManager` object creates a `PersistenceContext` which holds the entity. So Persistence context, you can say that it's a placeholder for entities whatever the entity manager is working on. **For every `EntityManager` there is a `PersistenceContext` and this `PersistenceContext` only stores the entity which entity manager is working one.** This `PersistenceContext` helps into **first level caching**, because it only stores the entity even though it's there in the DB, it is also working in memory.

- Once it is done, ultimately dialect comes into the picture. It converts JQL(ORM Sytax) to SQL. Dialects talks to JDBC and JDBC talks to DB

## 4. What is Persistence Unit in JPA

A Persistence Unit defines a set of all entity classes which share same configuration that are managed by `EntityManager` in an application. It is defined in the `persistence.xml` file, which is located in the `META-INF` directory. The `persistence.xml` file contains configuration details such as the database connection properties , entity classes, and JPA provider(hibernate etc.) etc...

Example `persistence.xml`:

![persistencexml](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/persistencexml.png)


## 5. What is EntityManagerFactory

- Using Persistence Unit configuration, EntityManagerFactory object get created during application startup.
- If any property is not provided, default one is picked and set.
- 1 EntityManagerFactory for 1 Persistence Unit.
- This class act as a Factory to create an object of EntityManager.

Example `LocalContainerEntityManagerFactoryBean.java`:

- In `createNativeEntityManagerFactory class`, from the persistence unit, it is creating the entity manager factory object.

![LocalContainerEntityManagerFactoryBean](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/LocalContainerEntityManagerFactoryBean.png)

What if we want to manually create an object of EntityManagerFactory?

![AppConfig](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/AppConfig.png)


## 6. Transaction Manager association with EntityManagerFactory

In Spring, the `PlatformTransactionManager` is associated with the `EntityManagerFactory` to manage transactions. The `JpaTransactionManager` is a specific implementation for JPA.

Let's say we have 2 DB's H2 and MySQL. We have one persistence unit, one entity manager factory, and one transaction manager for each H2 and MySQL.

During persistence unit, we have specified the value for `transaction-type` value either:
- RESOURCE_LOCAL (defaul) (Scope to only one DB).
- JTA (Java Transaction API) (Scope span across multiple DBs).

During application startup, after `EntityManagerFactory` object created, based on RESOURCE_LOCAL OR JTA, transaction manager object gets created.

Example:

UseCase1 : Transaction Manager for managing txn for 1 DB

![ORM-UseCase1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ORM-UseCase1.png)

We can create it manually too:

![ORM-UseCase1-Manual](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ORM-UseCase1-Manual.png)

UseCase2 : Transaction Manager for creating txn which can span across multiple DB

## 7. EntityManager and PersistenceContext

### EntityManager

`EntityManager` is the primary interface in JPA used to interact with the persistence context. It provides methods for CRUD operations, querying, and transaction management.

    - persist() (for saving)
    - merge() (for updating)
    - find() (for fetching)
    - remove() (for deleting)
    - createQuery() (for executing JPQL queries)

- EntityManager interface methods are implemented by JPA Vendors like Hibernate etc...
- EntityManagerFactory helps to create an object of EntityManager.

- JPARepository internally is connecting to EntityManager. Why can't we directly interact with EntityManager?
- JPARepository provides some predefined methods like findAll, deleteAll etc.
- Also provides pagination and sorting capability...
- Also @Transactional support on operations which needs it on operations like insert, modify, delete.
- Start and close EntityManager resource, so no need to manage it.

![JpaRepository-Transactional](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JpaRepository-Transactional.png)

- EntityManager insert/update/delete operations are Transaction bounded. Means, it first check if `Transaction` is open, if not, it will throw exception. But not all(read operations are not transaction bounded).

What if, I try to directly call EntityManager persist method, instead of spring framework.

```java
@Service
public class UserDetailsService {
    @PersistenceContext
    EntityManager entityManager;

    public void saveUser(UserDetails user){
        entityManager.persist(user);
    }
}
```

The above code will return `TransactionRequiredException` error saying `No EntityManager with actual transaction available for current thread`.

Example:

```java
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaExample {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("example-unit");
        EntityManager em = emf.createEntityManager();

        em.getTransaction().begin();
        User user = new User();
        user.setName("John Doe");
        user.setEmail("john.doe@example.com");
        em.persist(user);
        em.getTransaction().commit();

        em.close();
        emf.close();
    }
}
```

### PersistenceContext

`@PersistenceContext` is used to inject an `EntityManager` into a Spring-managed bean.

- Consider its a first level cache.
- For each EntityManager, PersistenceContext object is created, which hold list of Entities its working on.
- Also manage the life cycle of entity.

Example:

```java
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    @PersistenceContext
    private EntityManager entityManager;

    @Transactional
    public void saveUser(User user) {
        entityManager.persist(user);
    }
}
```

## 8. Lifecycle of Entity

The lifecycle of an entity in JPA consists of four states:

1. **New/Transient**: The entity is created but not yet associated with the persistence context.
2. **Managed**: The entity is associated with the persistence context and changes to the entity are tracked.
3. **Detached**: The entity is no longer associated with the persistence context.
4. **Removed**: The entity is marked for removal from the database.

![lifecycle](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/lifecycle.png)
