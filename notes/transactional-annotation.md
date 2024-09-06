# Spring Boot @Transactional Annotation

The `@Transactional` annotation in Spring Boot is used to manage transactions in your application. It ensures that a block of code is executed atomically, meaning either all operations within the block succeed or none of them do. This guide will cover various aspects of using the `@Transactional` annotation.

## Critical Section

A critical section refers to a block of code that should be executed atomically. By using the `@Transactional` annotation, you can mark a method or a class as a critical section, allowing Spring Boot to handle the transaction management for you.

```markdown
In this guide, we will explore the usage of the `@Transactional` annotation in Spring Boot for managing transactions in your application.

## Critical Section

A critical section refers to a block of code that should be executed atomically, ensuring that either all the operations within the block succeed or none of them do. By using the `@Transactional` annotation, you can mark a method or a class as a critical section, allowing Spring Boot to handle the transaction management for you.
```

## Class Level Transactional Annotation

When you apply the `@Transactional` annotation at the class level, it applies to all public methods within the class. This is useful when you want all methods in a service to participate in a transaction.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class UserService {

    public void createUser(User user) {
        // Code to create a user
    }

    public void updateUser(User user) {
        // Code to update a user
    }
}
```

In this example, both `createUser` and `updateUser` methods will be executed within a transaction.

## Method Level Transactional Annotation

You can also apply the `@Transactional` annotation at the method level to specify transactional behavior for individual methods.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {
        // Code to place an order
    }

    public void cancelOrder(Order order) {
        // Code to cancel an order
    }
}
```

In this example, only the `placeOrder` method will be executed within a transaction, while the `cancelOrder` method will not.

## How Internally AOP Does Transaction Management

Spring AOP (Aspect Oriented Programming) is used to manage transactions internally. When a method annotated with `@Transactional` is called, Spring creates a proxy for the target object. This proxy intercepts the method call and manages the transaction according to the specified attributes.

### Steps Involved:
1. **Proxy Creation**: Spring creates a proxy for the target object.
2. **Method Interception**: The proxy intercepts the method call.
3. **Transaction Management**: The proxy starts a transaction before the method execution and commits or rolls back the transaction after the method execution based on the outcome.

### Example
```java
@Aspect
@Component
public class TransactionAspect {

    @Around("@annotation(org.springframework.transaction.annotation.Transactional)")
    public Object manageTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        // Start transaction
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

        Object result;
        try {
            result = joinPoint.proceed();
            // Commit transaction
            transactionManager.commit(status);
        } catch (Throwable ex) {
            // Rollback transaction
            transactionManager.rollback(status);
            throw ex;
        }

        return result;
    }
}
```

## Code Demo

### Entity Class
```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    // Getters and Setters
}
```

### Repository Interface
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {}
```

### Service Class
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
    }

    public void updateUser(User user) {
        userRepository.save(user);
    }
}
```

### Controller Class
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping
    public void createUser(@RequestBody User user) {
        userService.createUser(user);
    }

    @PutMapping
    public void updateUser(@RequestBody User user) {
        userService.updateUser(user);
    }
}
```

## Conclusion

The `@Transactional` annotation in Spring Boot simplifies transaction management by allowing you to define transactional behavior declaratively. By understanding how to use it at both the class and method levels, and how Spring AOP manages transactions internally, you can effectively manage transactions in your Spring Boot applications.
