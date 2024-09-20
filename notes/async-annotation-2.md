# Async Annotation in Spring Boot

## Introduction

The `@Async` annotation in Spring Boot is used to execute methods asynchronously. This means that the method will run in a separate thread, allowing the main thread to continue processing without waiting for the method to complete. This is particularly useful for tasks that are time-consuming and can be performed in the background, such as sending emails, processing files, or making remote API calls.

## Conditions for @Async Annotation to work properly?

1. Different Class:
If `@Async` annotation is applied to the method within the same class from which it is being called, then proxy mechanism is skipped because internal method calls are NOT INTERCEPTED. So, the method with `@Async` annotation should be called from a different class.

2. Public method:
Method annotated with `Async` must be public. And again, AOP interception works only on public methods.

![async-example-1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/async-example-1.png)

## How @Async and Transaction Management works together?

- **Usecase 1**: ❌ Transaction Context do not transfer from caller thread to new thread which got created by Async.

![transaction-context](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/transaction-context.png)

In `UserController`, we have `updateUserMethod` function where we are calling `userService.updateUser()`. For `updateUser` method in `UserService` class, we've put `@Transactional` annotation. In `updateUser()` method, we are calling `updateUserBalance()` from `UserUtility` class, which is `@Async`. The problem here is that, whenever we create a new thread, it do not carry forward the `Transactional Context`(information like `propagation level`,`thread pool`, ...etc).

- **Usecase 2**: <span style="color:yellow">Use with Precaution</span>, as new thread will be created and have transaction management too but context is not same as parent thread. So Propagation will not work as expected.

 ![async-usecase2](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/async-usecase2.png)

In `UserController`, we have `updateUserMethod` function where we are calling `userService.updateUser()`. For `updateUser` method in `UserService` class, we've put `@Transactional` as well as `@Async`annotation. Suppose we have `@Transactional` in `updateUserMethod()` in `UserController`. Because of `@Async` in `updateUser()`, the `Transactional Context` will not get carry forward, like `Propagation` will not work as expected.

- **Usecase 3**: ✅

 ![async-usecase3](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/async-usecase3.png)

In `UserController`, we have `updateUserMethod` function where we are calling `userService.updateUser()`. For `updateUser` method in `UserService` class, we've put `@Async`annotation. Your main thread is free right now, it do not have to wait for `updateUser()` method. Now it calls `userUtility.updateUser()` which has `@Transactional` and now everything runs in a transaction, anything fails rollback will happen. In summary, what we have done is, we have used a separate thread to do the transactional work and our main thread is free totally.

## Enabling Async Support

To use the `@Async` annotation, you need to enable async support in your Spring Boot application. This can be done by adding the `@EnableAsync` annotation to one of your configuration classes.

Example:
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;

@Configuration
@EnableAsync
public class AsyncConfig {
    // Configuration code here
}
```

## Async method return type

You can apply the `@Async` annotation to any method that you want to run asynchronously. The method should return a `Future`, `CompletableFuture`, or `ListenableFuture` if you need to get the result of the asynchronous operation.

### **Using Future**(Deprecated now):

![future-return-type](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/future-return-type.png)

When we are using `@Async` annotation, we know that it will create a new thread. If we want to get the output of that new thread, that's where `Future` and `CompletableFuture` comes into picture.

### **Using CompletableFuture**(Introduced in Java8):

![completable-future-return-type](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/completable-future-return-type.png)

## Exception Handling:

![execption-handling](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/execption-handling.png)

**For methods that do not return anything, how are you going to handle the exception in your main thread?**
1. Use `try-catch` block within the `Async` method itself.
2. You could use Spring boot framework code or implement Custom `AsyncExecptionHandler`(Suggested way)

<ins>Spring boot framework code</ins>

![spring-boot-framework-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/spring-boot-framework-code.png)

<ins>Implement Custom AsyncExceptionHandler</ins>

![custom-async-exception-handler](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-async-exception-handler.png)

- we just need to implement `AsyncConfigurer` and in `getAsyncUncaughtException()` method, return `AsyncUncaughtExceptionHandler`. By doing, our custom AsyncExceptionHandler will get returned.

## Important Interview Questions on Async Annotation

### 1. What is the purpose of the `@Async` annotation in Spring Boot?

The `@Async` annotation is used to execute methods asynchronously, allowing the main thread to continue processing without waiting for the method to complete. This is useful for tasks that can be performed in the background, improving the responsiveness and performance of the application.

### 2. How do you enable async support in a Spring Boot application?

Async support can be enabled by adding the `@EnableAsync` annotation to one of your configuration classes.

Example:
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;

@Configuration
@EnableAsync
public class AsyncConfig {
    // Configuration code here
}
```

### 3. What are the return types supported by the `@Async` annotation?

The `@Async` annotation supports methods that return `void`, `Future`, `CompletableFuture`, or `ListenableFuture`. These return types allow you to handle the result of the asynchronous operation if needed.

### 4. Can you use the `@Async` annotation on private methods?

No, the `@Async` annotation cannot be used on private methods. It can only be applied to public methods. This is because Spring uses proxies to manage asynchronous execution, and private methods are not accessible through proxies.

### 5. How do you handle exceptions in asynchronous methods?

Exceptions in asynchronous methods can be handled by using the `CompletableFuture.exceptionally` method or by implementing a custom `AsyncUncaughtExceptionHandler`.

Example:
```java
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;

import java.lang.reflect.Method;

@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new CustomAsyncExceptionHandler();
    }

    class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {
        @Override
        public void handleUncaughtException(Throwable throwable, Method method, Object... obj) {
            System.err.println("Exception message - " + throwable.getMessage());
            System.err.println("Method name - " + method.getName());
            for (Object param : obj) {
                System.err.println("Parameter value - " + param);
            }
        }
    }
}
```

### 6. How do you configure the thread pool for asynchronous methods?

You can configure the thread pool for asynchronous methods by defining a `ThreadPoolTaskExecutor` bean in your configuration class.

Example:
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
public class AsyncConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
}
```

In this example, a custom thread pool is configured with a core pool size of 5, a maximum pool size of 10, and a queue capacity of 25.

By understanding and using the `@Async` annotation, you can improve the performance and responsiveness of your Spring Boot applications by offloading time-consuming tasks to separate threads.
