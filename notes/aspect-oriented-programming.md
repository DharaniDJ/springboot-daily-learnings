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
    @Before ("execution(public String com.conceptandcoding.learningspringboot.Employee.fetchEmployee())")
    public void beforeMethod() {
        System.out.println("inside beforeMethod Aspect");
    }
}
```
### Explanation
In the above example, after the application started properly, if we hit the api `/fetchEmployee`, before the execution of this `fetchEmployee` method, `beforeMethod()` will get invoke and `inside beforeMethod Aspect` gets printed and then returns `Employee Fetched`.  AOP helps you to intercept the method right before this method get invoked. It intercepted it and it performs certain task and then it calls the method.

## Some important AOP concepts:
![AOP Concepts Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/AOP_Concepts.png)


## Pointcuts and Its Different Types

Pointcuts are expressions that match join points. They define where an `advice` should be applied. Spring AOP provides several types of pointcut expressions:

### Execution
The `execution` pointcut is used to match particular method in a particular class.

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}
```
This pointcut matches the execution of any method in the `com.example.service` package.

![Execution_Pointcut_1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/Execution_Pointcut_1.png)

![Execution_Pointcut_2](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/Execution_Pointcut_2.png)

### Within
The `within` pointcut is used to match all methods within any class or package.

This pointcut will run for each method in the class Employee.
```java
@Before("within(com.conceptandcoding.learningspringboot.Employee)")
```

This pointcut will run for each method in this package and subpackage.
```java
@Before("within(com.conceptandcoding.learningspringboot..*)")
```

### @within
The `@within` pointcut is used to match any method in a class which has this annotation.

This pointcut will run for each method in the class Employee.
```java
@Before("@within(com.conceptandcoding.learningspringboot.Employee)")
```

This pointcut will run for each method in this package and subpackage.
```java
@Before("@within(com.conceptandcoding.learningspringboot..*)")
```

![@within-Pointcut](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/@within-Pointcut.png)

### Explanation

`@Before("@within(org.springframework.stereotype.Service)")` - Will look for classes that has `@Service` annotation

### @annotation
The `@annotation` pointcut is used to match any method that is annotated with given annotation.


```java
@RestController
@RequestMapping(value="/api/")
public class Employee {
    @GetMapping(path="/fetchEmployee")
    public String fetchEmployee(){
        return "Employee fetched";
    }
}
```

```java
@Component
@Aspect
public class LoggingAspect {

    @Before("@annotation(org.springframework.web.bind.annotation.GetMapping)")
    public void beforeMethod(){
        System.out.println("inside beforeMethod Aspect");
    }
}
```

### Explanation

`@annotation(org.springframework.web.bind.annotation.GetMapping)` - whenever we invoke any method, before running, springboot will check if there is any pointcut matching. When we invoke `fetchEmployee`, since it has `@GetMapping` annotation, `beforeMethod` in the `LoggingAspect` will run before `fetchEmployee`

### Args
The `args` pointcut is used to match join points based on the arguments passed to the method.

```java
@Pointcut("args(java.lang.String, ..)")
public void methodsWithStringArgs() {}
```
This pointcut matches methods that take a `String` as the first argument.

![args-Pointcut](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/args-Pointcut.png)

### @args
The `@args` pointcut is used to match any method with particular parameters and that parameter class is annotated with particular annotation.

![@args-Pointcut](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/@args-Pointcut.png)

From the above image, Spring boot will look for any method that accept any argument and that argument class should have the annotation `Service`. From `EmployeeUtil` class, `employeeHelperMethod` function accepts `EmployeeDAO` as argument which is annotated with `@Service`. Since it got matched, it will execute `beforeMethod` function.

### Target
The `target` pointcut is used to match methods based on a particular instance of a class.

![target-Pointcut](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/target-Pointcut.png)

From the above image, whenever an instance of `EmployeeUtil` class is used to call any method, this point cut will get matched. In `EmployeeController` class, we have `fetchEmployee` method inside which we use `EmployeeUtil` instance and call `EmployeeHelperMethod`. Because of this target point cut got matched and `beforeMethod` executed first.

![target-Pointcut-Interface](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/target-Pointcut-Interface.png)

## Combining multiple pointcuts
![combining-pointcuts](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/combining-pointcuts.png)

`@Before("execution(* com.conceptandcoding.learningspringboot.EmployeeController.*())")` - Any method that does not accept any arguemnt in `EmployeeController` class with any return type

`@within("org.springframework.web.bind.annotation.RestController")` - Any class which has `RestController` annotation

## Named Pointcuts

![named-pointcuts](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/named-pointcuts.png)


## Advice and Its Types

Advice is the action taken by an aspect at a particular join point. Spring AOP supports several types of advice:

![Advice](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/Advice.png)


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

![@around-advice](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/@around-advice.png)

`joinPoint.proceed()` - Invoke your method that got match with the point cut.

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

### Code Generation Library(CGLIB) Proxies
If the target object does not implement any interfaces, Spring uses CGLIB to create a proxy by subclassing the target object.

By now, few questions we all should have:
1. How this interception works?
2. What if we have 1000s of pointcut, so whenever I invoke a method, does matching happens with 1000s of pointcuts?

Let's understand the AOP flow, to get an answer of above doubts:

## How Interception Works

Interception in Spring AOP works by creating a proxy of the target object. When a method on the target object is called, the call is intercepted by the proxy, which then applies the advice before or after delegating the call to the target object.

![interception-working](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/interception-working.png)

![interception-explanation1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/interception-explanation1.png)

![application-startup](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/application-startup.png)

![Interception-code-explanation](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/Interception-code-explanation.png)

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
