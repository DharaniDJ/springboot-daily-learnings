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

![async-annotation-example](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/async-annotation-example.png)

![async-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/async-output.png)

## How does this "Async" Annotation, creates a new thread?
Internally, Spring boot first looks for `defaultExecutor`, if no `defaultExecutor` found, only then `SimpleAsyncTaskExecutor` is used.

### AsyncExecutionInterceptor.java
```java
@Nullable
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {

    Executor defaultExecutor = super.getDefaultExecutor(beanFactory);
    return (Executor) (defaultExecutor != null ? defaultExecutor : new SimpleAsyncTaskExecutor());
}
```

## Use Cases
The `@Async` annotation is useful in various scenarios, including:
- **Sending Emails**: Sending emails can be time-consuming and should be done asynchronously to avoid blocking the main thread.
- **File Processing**: Large file processing tasks can be performed in the background.
- **Web Scraping**: Scraping data from websites can be done asynchronously to improve performance.
- **Database Operations**: Long-running database operations can be executed in a separate thread.

### Use case 1

![usecase1-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase1-code.png)

#### Explanation
If you see the `AppConfig` class, it is empty. During the application startup, spring boot sees that, no `ThreadPoolTaskExecutor` Bean present, so it creates its default `ThreadPoolTaskExecutor` with below configurations

![threadPoolTaskExecutor-configurations](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/threadPoolTaskExecutor-configurations.png)

`ThreadPoolTaskExecutor` is nothing but a Spring boot object, which is just a wrapper around Java `ThreadPoolTaskExecutor`.

![threadPoolTaskExecutor-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/threadPoolTaskExecutor-code.png)

And it's not recommended at all, why?
- **Under Utilization of Threads**: With Fixed Min pool size and Unbounded Queue(size is too big), its possible that most of the tasks will sit in the queue rather than creating new thread.
- **High Latency**: Since queue size is too big, tasks will queue up till queue is not fill, high latency might occur during high load.
- **Thread Exhaustion**: Lets say, if Queue also get filled up, then Executor will try to create new threads till Max pools size is not reached, which is Interger.MAX_VALUE. This can lead to thread exhaustion. And server might go down because of overhead of managing so many threads.
- **High Memory Usage**: Each threads need some memory, when we are creating these many threads, which may consume large amount of memory too, which might lead to performance degradation too.

In summary, we should not go with `defaultExecutor`.

### Use case 2: Creating our own custom, ThreadPoolTaskExecutor

![usecase2-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase2-code.png)

#### Explanation
Inside the `AppConfig`, we are providing our own `ThreadPoolTaskExecutor`. During Application startup, Spring boot sees that, `ThreadPoolTaskExecutor` Bean present, so it makes it default only.

And even when we use `@Async` without any name, our `myThreadPoolExecutor` will get picked only. Hence we can either use `@async` or `@async("myThreadPoolExecutor")`

![usecase2-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase2-output.png)

![usecase2-sleep-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase2-sleep-output.png)

To understand the queue and all, we are putting sleep in async method, to simulate load. Initially the `minPoolSize = 2`, `maxPoolSize = 4`, and `queueSize = 3`. So `Task#1` comes and it picks `Thread#1`, same happens with `Task#2` and `Thread#2`. Now no free thread are available, and any incoming task will go into queue, `Task#3`, `Task#4`, and `Task#5` goes to queue.

When `Task#6` comes, another thread got invoked because `maxPoolSize = 4`, same thing happens when `Task#7` comes. Any new requests after this point will get rejected.

It is recommended, as it solves all the issues existing with the previous use case.

### Use case 3: Creating our own custom, ThreadPoolExecutor (Java one)

![usecase3-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase3-code.png)

#### Explanation
Inside the `AppConfig`, we are providing our own `ThreadPoolExecutor`. During Application startup, Spring boot sees that, `ThreadPoolExecutor`(Java one) Bean present, so **`it do not create its own default `ThreadPoolTaskExecutor`(spring wrapper one)`**, instead it set the default executor to `SimpleAsyncTaskExecutor`. In the above `AsyncExecutionInterception` class, `defaultExecutor` will be null and hence it returns `SimpleAsyncTaskExecutor`.

Now, if we run the above code, what we see.

![usecase3-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase3-output.png)

And it is not recommended at all to use `SimpleAsyncTaskExecutor`, why?

It just creates new thread every time. so it may lead to 

- **Thread Exhaustion**: Just blindly creating new thread with every Async request, might lead up to thread exhaustion.
- **Thread Creation Overhead**: Since Threads are not reused, so thread management (creation, destroying) is an additional overhead.
- **High Memory Usage**: Each threads need some memory, when we are creating these many threads, which may consume large amount of memory too, which might lead to performance degration too.

So, whenever we have defined our own `ThreadPoolExecutor`(Java one), always specify the name also with Async annotation.

![usecase3-code-1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase3-code-1.png)

![usecase3-output-1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase3-output-1.png)


## Another Way
During the configuration, for AppConfig implement `AsyncConfigurer`. Once you implement this class, you need to override `getAsyncExecutor` method. We can use `ThreadPoolExecutor`(java one) or `ThreadPoolTaskExecutor`. we do not need to include executor name in `@Async`.

Since we are not creating a bean(which is singleton), we are creating a private ThreadPoolExecutor `poolExecutor` and checking if it is null. If it is null, then only we are creating the object. Else we are using the existing one.

![usecase4-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase4-code.png)

![usecase4-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/usecase4-output.png)


## Conclusion
The `@Async` annotation in Spring Boot provides a simple and effective way to execute methods asynchronously. By understanding how to use it and customize the thread pool, you can improve the performance and responsiveness of your application.