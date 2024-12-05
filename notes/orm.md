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

## 3. Architecture of JPA

JPA architecture consists of several key components:

- **Entity**: Represents a table in the database.
- **EntityManager**: Interface used to interact with the persistence context.
- **EntityManagerFactory**: Factory class for `EntityManager` instances.
- **Persistence Unit**: A set of all entity classes that are managed by `EntityManager` in an application.
- **Persistence Context**: A set of entity instances in which for any persistent entity identity, there is a unique entity instance.

![JPAArchitecture](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JPAArchitecture.png)

## 4. What is Persistence Unit in JPA

A Persistence Unit defines a set of all entity classes that are managed by `EntityManager` in an application. It is defined in the `persistence.xml` file, which is located in the `META-INF` directory. The `persistence.xml` file contains configuration details such as the data source, entity classes, and JPA provider.

Example `persistence.xml`:

```xml
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
    <persistence-unit name="example-unit">
        <class>com.example.User</class>
        <properties>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:testdb"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value="password"/>
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
        </properties>
    </persistence-unit>
</persistence>
```

## 5. What is EntityManagerFactory

`EntityManagerFactory` is a factory class for `EntityManager` instances. It is responsible for creating and managing multiple `EntityManager` instances. It is typically created once per application and is thread-safe.

Example:

```java
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaExample {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("example-unit");
        // Use the EntityManagerFactory
        emf.close();
    }
}
```

## 6. Transaction Manager association with EntityManagerFactory

In Spring, the `PlatformTransactionManager` is associated with the `EntityManagerFactory` to manage transactions. The `JpaTransactionManager` is a specific implementation for JPA.

Example configuration:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;

import javax.persistence.EntityManagerFactory;

@Configuration
public class JpaConfig {

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        em.setPersistenceUnitName("example-unit");
        return em;
    }

    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

## 7. EntityManager and PersistenceContext

### EntityManager

`EntityManager` is the primary interface used to interact with the persistence context. It provides methods for CRUD operations, querying, and transaction management.

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

### Example

```java
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaExample {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("example-unit");
        EntityManager em = emf.createEntityManager();

        // New/Transient state
        User user = new User();
        user.setName("John Doe");
        user.setEmail("john.doe@example

Similar code found with 2 license types