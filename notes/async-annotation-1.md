# Spring Boot @Async Annotation

## Introduction
The `@Async` annotation in Spring Boot is used to execute methods asynchronously. This means that the method will run in a separate thread, allowing the main thread to continue processing. This is particularly useful for tasks that are time-consuming and can be performed in the background, such as sending emails or processing large datasets.

## ThreadPool
- By default, Spring Boot uses a simple thread pool to manage asynchronous tasks. However, you can customize the thread pool to better suit your application's needs.
- It's a collection of thread (aka workers), which are available to perform the submitted tasks.
- Once task completed, worker thread get back to ThreadPool and wait for new task to assigned.
- Means threads can be reused.

![thread-pool](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/thread-pool.png)

In Java, thread pool is created using `ThreadPoolExecutor` Object

```Java
int minPoolSize = 2;
int maxPoolSize = 4;
int queueSize = 3;

ThreadPoolExecutor poolTaskExecutor = new ThreadPoolExecutor(minPoolSize, maxPoolSize, keepAliveTime=1, TimeUnit.Hours, new ArrayBlockingQueue<>(queueSize));
```

### Explanation
When the `ThreadPoolExecutor` is configured with a minimum pool size of 2, a maximum pool size of 4, and a queue size of 3, handling 8 incoming tasks proceeds as follows:

Initially, the executor uses the core 2 threads to start processing the first 2 tasks. As these threads are busy, the next 3 tasks are placed into the `ArrayBlockingQueue`, which has a capacity of 3. This fills up the queue completely. With 5 tasks either being processed or waiting in the queue, there are still 3 tasks left to handle. Since the queue is full, the executor will create additional threads, up to the maximum pool size of 4. Thus, 2 more threads are spawned to handle the remaining tasks. When 8th task comes, since the queue is full and it exceed the maxPoolSize, it will remain alive for up to 1 hour before being terminated if no new tasks arrive, thus optimizing resource use and ensuring efficient task processing.

## @Async Annotation
The `@Async` annotation is used to mark methods that should be executed asynchronously. When a method annotated with `@Async` is called, it will be executed in a separate thread, without blocking the main thread.

`@EnableAsync` - Once Spring boot sees this annotation, it will create a bean for `async` classes which is required for processing the task. There are so many things like `async interceptor` and other classes which get involved but they will only get their objects/beans or get created when you will put this annotation. Otherwise spring unnecessarily will not create a bean for those classes.

### Example
```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class EmailService {

    @Async
    public void sendEmail(String email) {
        // Simulate a long-running task
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Email sent to " + email);
    }
}
```

### Calling the Async Method
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EmailController {

    @Autowired
    private EmailService emailService;

    @GetMapping("/sendEmail")
    public String sendEmail(@RequestParam String email) {
        emailService.sendEmail(email);
        return "Email request received";
    }
}
```

In this example, the `sendEmail` method will be executed asynchronously, allowing the `sendEmail` endpoint to return immediately.

## Internal Code
Internally, Spring uses a proxy to intercept calls to methods annotated with `@Async`. The proxy delegates the method execution to a task executor, which manages the thread pool.

### Example of Internal Code
```java
@Aspect
@Component
public class AsyncAspect {

    @Around("@annotation(org.springframework.scheduling.annotation.Async)")
    public Object manageAsync(ProceedingJoinPoint joinPoint) throws Throwable {
        // Get the task executor
        Executor executor = ...;

        // Submit the task to the executor
        executor.execute(() -> {
            try {
                joinPoint.proceed();
            } catch (Throwable throwable) {
                throwable.printStackTrace();
            }
        });

        return null;
    }
}
```

## Use Cases
The `@Async` annotation is useful in various scenarios, including:
- **Sending Emails**: Sending emails can be time-consuming and should be done asynchronously to avoid blocking the main thread.
- **File Processing**: Large file processing tasks can be performed in the background.
- **Web Scraping**: Scraping data from websites can be done asynchronously to improve performance.
- **Database Operations**: Long-running database operations can be executed in a separate thread.

## Another Way
Another way to achieve asynchronous execution is by using `CompletableFuture`. This allows you to run tasks asynchronously and handle the results when they are available.

### Example with CompletableFuture
```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.concurrent.CompletableFuture;

@Service
public class DataService {

    @Async
    public CompletableFuture<String> fetchData() {
        // Simulate a long-running task
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return CompletableFuture.completedFuture("Data fetched");
    }
}
```

### Calling the CompletableFuture Method
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.CompletableFuture;

@RestController
public class DataController {

    @Autowired
    private DataService dataService;

    @GetMapping("/fetchData")
    public CompletableFuture<String> fetchData() {
        return dataService.fetchData();
    }
}
```

In this example, the `fetchData` method returns a `CompletableFuture`, allowing the caller to handle the result asynchronously.

## Conclusion
The `@Async` annotation in Spring Boot provides a simple and effective way to execute methods asynchronously. By understanding how to use it and customize the thread pool, you can improve the performance and responsiveness of your application. Additionally, using `CompletableFuture` offers another way to handle asynchronous tasks.
