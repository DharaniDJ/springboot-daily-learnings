# Spring Boot Aspect Oriented Programming (AOP)

Aspect Oriented Programming (AOP) is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns. In Spring Boot, AOP is used to define common behaviors that can be applied across various points in an application.

- In simple terms, it helps to intercept the method invocation. And we can perform some task before and after the method.
- AOP allow us to focus on business logic by handling boilerplate and repetitive code like logging, transaction management etc.
- So, Aspect is a module which handle this repetitive or boilerplate code.
- Helps in achieving reusability, maintainability of the code.

Used during:
- Logging
- Security
- Transaction Management etc...

## Dependency you need to add in pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### Example
```java
@RestController
@RequestMapping(value="/api/")
public class Employee {
    @GetMapping (path="/fetchEmployee")
    public String fetchEmployee() {
        return "Employee Fetched";
    }
}
```
```java
@Component
@Aspect
public class LoggingAspect {
    @Before ("execution(Employee.fetchEmployee())")
    public void beforeMethod() {
        System.out.println("inside beforeMethod Aspect");
    }
}
```
### TODO
In the above example, after the application started properly

## Pointcuts and Its Different Types

Pointcuts are expressions that match join points. They define where an advice should be applied. Spring AOP provides several types of pointcut expressions:

### Execution
The `execution` pointcut is used to match method execution join points.

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}
```
This pointcut matches the execution of any method in the `com.example.service` package.

### Within
The `within` pointcut is used to match join points within certain types.

```java
@Pointcut("within(com.example.service..*)")
public void withinServiceLayer() {}
```
This pointcut matches any join point within the `com.example.service` package and its sub-packages.

### Args
The `args` pointcut is used to match join points based on the arguments passed to the method.

```java
@Pointcut("args(java.lang.String, ..)")
public void methodsWithStringArgs() {}
```
This pointcut matches methods that take a `String` as the first argument.

### Target
The [`target`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fc%3A%2FUsers%2Fdharani.chrinta%2FDeveloper%2FPersonal%2Fspring-boot-daily-learnings%2Ftarget%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "c:\Users\dharani.chrinta\Developer\Personal\spring-boot-daily-learnings\target") pointcut is used to match join points based on the target object type.

```java
@Pointcut("target(com.example.service.MyService)")
public void targetMyService() {}
```
This pointcut matches join points where the target object is of type `MyService`.

## Advice and Its Types

Advice is the action taken by an aspect at a particular join point. Spring AOP supports several types of advice:

### Before
The `@Before` advice runs before the method execution.

```java
@Before("execution(* com.example.service.*.*(..))")
public void beforeAdvice(JoinPoint joinPoint) {
    System.out.println("Before method: " + joinPoint.getSignature());
}
```

### After
The `@After` advice runs after the method execution, regardless of its outcome.

```java
@After("execution(* com.example.service.*.*(..))")
public void afterAdvice(JoinPoint joinPoint) {
    System.out.println("After method: " + joinPoint.getSignature());
}
```

### Around
The `@Around` advice runs before and after the method execution.

```java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("Before method: " + joinPoint.getSignature());
    Object result = joinPoint.proceed();
    System.out.println("After method: " + joinPoint.getSignature());
    return result;
}
```

## Join Point

A join point is a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Join point kind: " + joinPoint.getKind());
        System.out.println("Signature declaring type: " + joinPoint.getSignature().getDeclaringTypeName());
        System.out.println("Signature name: " + joinPoint.getSignature().getName());
        System.out.println("Arguments: " + Arrays.toString(joinPoint.getArgs()));
        System.out.println("Target class: " + joinPoint.getTarget().getClass().getName());
    }
}
```

## Proxy Creation

Spring AOP uses proxies to implement the aspects. A proxy is an object that wraps the target object and intercepts method calls to add additional behavior.

### JDK Dynamic Proxies
Spring uses JDK dynamic proxies if the target object implements at least one interface.

### CGLIB Proxies
If the target object does not implement any interfaces, Spring uses CGLIB to create a proxy by subclassing the target object.

## How Interception Works

Interception in Spring AOP works by creating a proxy of the target object. When a method on the target object is called, the call is intercepted by the proxy, which then applies the advice before or after delegating the call to the target object.

### Example
```java
@Aspect
@Component
public class PerformanceAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long executionTime = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");
        return result;
    }
}
```
In this example, the `PerformanceAspect` measures the execution time of methods in the `com.example.service` package.

## Conclusion

Aspect Oriented Programming in Spring Boot allows for the separation of cross-cutting concerns, making the code more modular and easier to maintain. By understanding pointcuts, advice, join points, proxy creation, and interception, you can effectively use AOP to enhance your Spring Boot applications.

Test Message