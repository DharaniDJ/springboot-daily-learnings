# Dependency Injection in Spring Boot

## Problem Exist Today

In traditional programming, components are tightly coupled, making it difficult to test, maintain, and extend the system. This tight coupling leads to:

*   **Rigidity**: Changing one component affects others, making it hard to modify the system.
*   **Fragility**: A small change can break the entire system.
*   **Immobility**: Components are hard to reuse or replace.

### Example
```java
public class User{
    Order order = new Order();

    public User(){
        System.out.println("initializing user");
    }
}
```

```java
public class Order{

    public Order(){
        System.out.println("initializing order");
    }
}
```

<ins>__Issues with above class structure:__</ins>

1. Both User and Order class are __Tightly coupled__.

    Suppose, Order object creation logic gets changed(lets say in future object becomes an interface and it has many concrete class), then USER class has to be changed too.

    ![Tightly Coupled Code Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/TightlyCoupled.png)

2. It breaks __Dependency Inversion__ rule of S.O.L.I.D principle

    i. This principle says that DO NOT depend on concrete implementation, rather depend on abstraction.

### Example
    ```java
    public class User{
        Order order;

        public User(Order orderObj){
            this.order = orderObj;
        }
    }
    ```

Dependency Injection (DI) addresses these issues by decoupling the creation of dependencies from their usage. Through Dependency Injection, we achieve Dependency Inversion Principle.

## What is Dependency Injection?

- Dependency Injection (DI) is a design pattern that decouples components, making the system more flexible, testable, and maintainable.
- Using Dependency Injection, we can make our class independent of its dependencies.
- It helps to remove the dependency on concrete implementation and inject the dependencies from external source.

### Example
```java
@Component
public class User{

    @Autowired      // @Autowired, first look for a bean of the required type. If bean found, Spring will inject it.
    Order order;
}
```

```java
@Component
public class Order{
}
```

## Types of Dependency Injection

There are three types of Dependency Injection:

*   **Field Injection**: Injecting dependencies directly into fields.
*   **Setter Injection**: Injecting dependencies through setter methods.
*   **Constructor Injection**: Injecting dependencies through constructors.

## Field Injection
- Dependency is set into the fields of the class directly
- Spring uses __reflection__, it iterates over the fields and resolve the dependency.

### Advantages

*   **Easy to implement**: Simply annotate the field with `@Autowired`.
*   **Less boilerplate code**: No need to create constructors or setter methods.

### Disadvantages

*   **Tight coupling**: Fields are still tightly coupled to the dependent component.
*   **Limited testability**: Difficult to mock dependencies in unit tests. we do not have any constructor or setter method to put the mock object. We use Mockito `@Mock` and `@InjectMocks` to do this.
*   **Hidden dependencies**: Dependencies are not explicitly declared.
*   **Immutable fields**: Field injection cannot be used with immutable fields (i.e., `final` fields) because Spring cannot inject dependencies into fields that are declared as `final`. This makes field injection unsuitable for classes that require immutability.
*   **Circular dependencies**: More prone to circular dependency issues.
*   **Null Pointer Exception**: More prone to Null Pointer Exception.

### Example
```java
// Example of Field Injection
@Component
public class User {
    @Autowired
    Order order;

    public User(){
        System.out.println("User initialized");
    }
}

@Component
@Lazy
public class Order {
    public Order(){
        System.out.println("Order initialized");
    }
}

// Since we haven't provided @Configuration file, so spring might call its default constructor

// When we start the application, it prints in the below order.

// User initialized
// Order initialized
```

### Example of Null Pointer Exception
```java
@Component
public class User {
    @Autowired
    public Order order;

    public User(){
        System.out.println("User initialized");
    }

    public void process(){
        order.process();
    }
}

User userObj = new User();
userObj.process();      // throws NPE
```

## Setter Injection
- Dependency is set into the fields using the setter method.

### Advantages

*   **Explicit dependencies**: Dependencies are explicitly declared through setter methods. Dependency can be changed any time after the object creation(as object can not be marked as final).
*   **Easier testing**: Setter methods make it easier to test components, as we can pass mock object in the dependency easily.

### Disadvantages

*   **Immutable fields**: Fields cannot be marked as final. (We can not make it immutable).
*   **More boilerplate code**: Need to create setter methods for each dependency.
*   **Less flexible**: Setter methods can make the component less flexible.
*   **Optional dependencies**: Can lead to incomplete object initialization.
*   **Readability issues**: Difficult to read and maintained, as per standard, object should be initialized during object creation, so this might create code readability issue.

### Example
```java
// Example of Setter Injection
@Component
public class User {

    public Order order;

    public User(){
        System.out.println("User initialized");
    }

    @Autowired
    public void setOrderDependency(Order order){
        this.order = order;
    }
}
```

## Constructor Injection
- Dependency get resolved at the time of initialization of the Object itself.
- Its recommended to use.
- At the time of object creation, the dependencies are also resolved. Since constructor is used to creating an instance of an object. It resolves all the dependencies and then initialize it at the time of object creation.
- When only 1 constructor is present, then using `@Autowired` on constructor is not manditory. (from Spring version 4.3)

```java
@Component
public class User {
    Order order;

    public User(Object order){
        this.order = order;
        System.out.println("User initialized");
    }
}

@Component
@Lazy
public class Order {

    public Order(){
        System.out.println("Order initialized");
    }
}

// When we start the application, it prints in the below order.

// Order initialized
// User initialized

```

- When more than 1 constructor is present, then using `@Autowired` on constructor is manditory.

- When we have multiple constructors, then Spring will not be able to decide which constructor to use.

![Constructor Injection Exception Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ConstructorInjectionException.png)


![Constructor Injection Example Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ConstructorInjectionExample.png)

```
Invoice initialized
User initialized with only Invoice
```

### Advantages

*   All mandatory dependencies are created at the time of initialization itself. Makes 100% sure that our object is fully initialized with mandatory dependency.
    *   Avoid Null Pointer Exception(NPE) during runtime.
    *   Unnecessary null checks can be avoided too.
* We can create immutable object using Constructor injection. Because Constructor is only called once at the time of instance creation. After that we wont be able to change it.

```java
@Component
public class User {
    public final Order order;

    @Autowired
    public User(Order order){
        this.order = order;
        System.out.println("User initialized");
    }
}
```

*   **Fail Fast**: If there is any missing dependency, it will fail during compilation itself,rather that failing during run time.

```java
// For this example, we are getting to know about the error at the time of run time.

@Component
public class User {
    // Forgot to add `@Autowired`, so order is assigned to null
    public Order order;

    public User(){
        System.out.println("User initialized");
    }

    @PostConstruct
    public void init(){
        System.out.println(order == null);
    }
}

@Component
public class Order {
    public Order(){
        System.out.println("Order initialized");
    }
}
```

*   **Testability**: Unit testing is easy. We do not need `@Mock` or `@InjectMocks`
```java
class UserTest {
    private Order orderMockObj;

    private User user;

        @BeforeEach
        public void setup() {
            this.orderMockObj = Mockito.mock(Order.class);
            this.user = new User(orderMockObj);
        }
}
```

### Disadvantages
*   **More boilerplate code**: Requires constructors with parameters.
*   **Complex constructors**: Can lead to constructors with many parameters.



=============================================================================================

```markdown
# Dependency Injection in Spring Boot

## Problem Exist Today for Which Dependency Injection is Required
In traditional programming, objects are responsible for creating their dependencies. This tight coupling makes the code hard to test, maintain, and extend. Changes in one part of the application can have a ripple effect, requiring changes in multiple places. Dependency Injection (DI) addresses these issues by decoupling the creation of dependencies from their usage.

## What is Dependency Injection and Its Types
Dependency Injection is a design pattern that allows an object to receive its dependencies from an external source rather than creating them itself. This promotes loose coupling and enhances testability and maintainability.

### Types of Dependency Injection
1. **Field Injection**: Dependencies are injected directly into fields.
2. **Setter Injection**: Dependencies are injected through setter methods.
3. **Constructor Injection**: Dependencies are injected through the constructor.

## Field Injection and Its Advantages and Disadvantages
### Advantages
- Simplicity: Easy to implement and understand.
- Less boilerplate code: No need for constructors or setters.

### Disadvantages
- Limited testability: Difficult to mock dependencies in unit tests.
- Hidden dependencies: Dependencies are not visible in the class's public API.
- Circular dependencies: More prone to circular dependency issues.

### Example
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MyService {
    @Autowired
    private MyRepository myRepository;

    // Class implementation
}
```

## Setter Injection and Its Advantages and Disadvantages
### Advantages
- Flexibility: Allows changing dependencies at runtime.
- Better testability: Easier to mock dependencies in unit tests.

### Disadvantages
- Optional dependencies: Can lead to incomplete object initialization.
- More boilerplate code: Requires setter methods.

### Example
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MyService {
    private MyRepository myRepository;

    @Autowired
    public void setMyRepository(MyRepository myRepository) {
        this.myRepository = myRepository;
    }

    // Class implementation
}
```

## Constructor Injection and Its Advantages and Disadvantages
### Advantages
- Immutability: Dependencies are set at object creation and cannot be changed.
- Mandatory dependencies: Ensures all required dependencies are provided.
- Better testability: Easier to mock dependencies in unit tests.

### Disadvantages
- More boilerplate code: Requires constructors with parameters.
- Complex constructors: Can lead to constructors with many parameters.

### Example
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MyService {
    private final MyRepository myRepository;

    @Autowired
    public MyService(MyRepository myRepository) {
        this.myRepository = myRepository;
    }

    // Class implementation
}
```

## Circular Dependency Problem and Its Solutions
### Problem
Circular dependencies occur when two or more beans depend on each other, creating a cycle that Spring cannot resolve.

### Solutions
1. **Refactor Code**: Break the circular dependency by introducing an intermediary bean or redesigning the dependencies.
2. **Use `@Lazy` Annotation**: Mark one of the dependencies as lazy-initialized to break the cycle.

### Example
```java
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
public class A {
    private final B b;

    @Autowired
    public A(@Lazy B b) {
        this.b = b;
    }
}

@Component
public class B {
    private final A a;

    @Autowired
    public B(A a) {
        this.a = a;
    }
}
```

## Unsatisfied Dependency Problems and Its Solutions
### Problem
Unsatisfied dependencies occur when Spring cannot find a bean to inject into a dependent bean.

### Solutions
1. **Check Bean Definitions**: Ensure all required beans are defined and annotated correctly.
2. **Use `@Qualifier` Annotation**: Specify which bean to inject when multiple beans of the same type exist.

### Example
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
public class MyService {
    private final MyRepository myRepository;

    @Autowired
    public MyService(@Qualifier("specificRepository") MyRepository myRepository) {
        this.myRepository = myRepository;
    }
}
```

This guide provides an overview of Dependency Injection in Spring Boot, its types, and common problems with their solutions. The sample code snippets should help you understand these concepts in practice.
```