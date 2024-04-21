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

### Constructor Injection Exception Diagram
![Constructor Injection Exception Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ConstructorInjectionException.png)

### Constructor Injection Example Diagram
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

## Circular Dependency Problem and Its Solutions
### Problem
Circular dependencies occur when two or more beans depend on each other, creating a cycle that Spring cannot resolve.

![Circular Dependency Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/CircularDependency.png)

### Solutions
1. **Refactor Code**: Break the circular dependency by introducing an intermediary bean or redesigning the dependencies. For example, common code in which both are dependent, can be taken out to seperate class. This way we can break the circular dependency.
2. **Use `@Lazy` Annotation**: Mark one of the dependencies as lazy-initialized to break the cycle. Spring will create proxy bean instead of creating the bean instance immediately during application startup.

### Example -  `@Lazy` on Field Injection
```java
@Component
public class Order {

    @Autowired
    Invoice invoice;

    public Order() {
        System.out.println("Order initialized");
    }
}

@Component
public class Invoice {

    @Lazy   // Put Proxy
    @Autowired
    Order order;

    public Invoice() {
        System.out.println("Invoice initialized");
    }
}

// Invoice initialized
// Order initialized
```

### Example -  `@Lazy` on Setter Injection
```java
@Component
public class Order {

    Invoice invoice;

    public Order() {
        System.out.println("Order initialized");
    }

    @Autowired
    public void setInvoice(Invoice invoice){
        this.invoice = invoice;
    }
}

@Component
public class Invoice {

    Order order;

    public Invoice() {
        System.out.println("Invoice initialized");
    }

    @Lazy
    @Autowired
    public void setOrder(Order order){
        this.order = order;
    }
}
```
### Example -  Using `@PostConstruct`
```java
// Not Recommended

@Component
public class Order {

    @Autowired
    Invoice invoice;    // During Dependency Injection, this will have order as null

    public Order() {
        System.out.println("Order initialized");
    }

    @PostConstruct      // After Bean is constructed, this will set the order to current instance.
    public void initialize(){
        invoice.setOrder(this);
    }
}

@Component
public class Invoice {

    public Order order;     // By default, order is set to null as there is no @Autowired

    public Invoice() {
        System.out.println("Invoice initialized");
    }

    public void setOrder(Order order){
        this.order = order;
    }
}
```

## Unsatisfied Dependency Problems and Its Solutions
### Problem
Unsatisfied dependencies occur when Spring cannot find a bean to inject into a dependent bean.

### Solutions
1. **Use `@Primary` Annotation**: Specificy Spring to give 1st priority when multiple beans of the same type exist.
2. **Use `@Qualifier` Annotation**: Specify which bean to inject when multiple beans of the same type exist.

### Example for Unsatisfied Dependency
![Unsatisfied Dependency Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/UnsatisfiedDependency.png)

### Example for `@Primary` Annotation
```java
// Using @Primary Annotation
@Primary
@Component
public class OnlineOrder implements Order {
    public OnlineOrder() {
        System.out.println("Online Order initialized");
    }
}
```

### Example for `@Qualifier` Annotation
```java
// Using @Qualifier Annotation

@Component
public class User {
    @Qualifier("onlineOrderName")
    @Autowired
    Order order;

    public User() {
        System.out.println("User initialized");
    }
}

@Component
@Qualifier("onlineOrderName")
public class OnlineOrder implements Order {
    public OnlineOrder() {
        System.out.println("Online Order initialized");
    }
}

```