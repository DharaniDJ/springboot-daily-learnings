# Spring Boot Exception Handling

This guide covers various aspects of exception handling in Spring Boot, focusing on the `@ExceptionHandler`, `@ControllerAdvice`, `@ResponseStatus` annotations, and the exception resolvers involved. It explains the class hierarchy, flow diagrams, and provides demos of different exception resolvers that Spring Boot uses to manage exceptions within an application.

## Class Hierarchy of Exception Handling

Spring Boot’s exception handling structure revolves around a hierarchy of classes that facilitate handling various exceptions. The primary class, `HandlerExceptionResolver`, orchestrates the flow and management of exceptions through the following classes:

1. **HandlerExceptionResolverComposite**: Acts as an orchestrator and helper class that passes exceptions through a sequence of resolvers.
2. **DefaultErrorAttributes**: Provides default attributes for the error response.
3. **ExceptionHandlerExceptionResolver**: Handles exceptions by leveraging `@ExceptionHandler` and `@ControllerAdvice` annotations.
4. **ResponseStatusExceptionResolver**: Manages exceptions annotated with `@ResponseStatus`.
5. **DefaultHandlerExceptionResolver**: Handles predefined exceptions, such as `NoSuchRequestHandlingMethodException` or `HttpRequestMethodNotSupportedException`.

![HandlerClasses](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/HandlerClasses.png)

Each resolver plays a distinct role in processing exceptions, with `HandlerExceptionResolverComposite` managing the sequence, allowing each resolver to attempt handling the exception until it is resolved.

### Code Example: Exception Hierarchy
```java
@RestController
public class SampleController {

    @GetMapping("/api/get-user")
    public String getUser() {
        // Deliberately throw a NullPointerException to test the exception hierarchy
        throw new NullPointerException("Testing NullPointerException handling");
    }
}
```

In this example, when a `NullPointerException` occurs, the flow will be passed to the exception resolvers in sequence.

---

## Flow Diagram of Exception Resolvers

When an exception is thrown in a Spring Boot application, the following flow is followed:

1. **DispatcherServlet** intercepts the request and attempts to handle it.
2. If an exception occurs, **HandlerExceptionResolverComposite** directs it to:
   - `ExceptionHandlerExceptionResolver` – the first resolver in the sequence.
   - `ResponseStatusExceptionResolver` – the second resolver, handling `@ResponseStatus`-annotated exceptions.
   - `DefaultHandlerExceptionResolver` – the third resolver, managing predefined exceptions.
3. Finally, **DefaultErrorAttributes** sets the response attributes if no resolver successfully handles the exception, ensuring the client receives a structured error response.

The exception handling flow follows this sequence, stopping at the first resolver that can process the exception.

![ExceptionSequence](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ExceptionSequence.png)

---

## Demo of Exception Resolvers

![NPECode](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/NPECode.png)

In the above code `UserController` class and `GetMapping` i.e /api/get-user, we are throwing `Null Pointer Exception`(NPE) with message. When we run the code, we will get `500 Internal Server Error`. Internally, We did not set any information about the status code and error message. Then who is responsible for this?

![CustomException](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/CustomException.png)

In the above code, instead of NPE, we are throwing a `CustomException` which has `status` and `message` fields, constructors and getter methods. When we run the code, we will get `500 Internal Server Error`. But why my Return Code "BAD_REQUEST" i.e 400 and my error message is not shown in output? If None of the `ExceptionResolver` handles your Exception, then `DefaultErrorAttributes` will set to 500. In this Exception scenario, exception passes through each Resolver like 
`ExceptionHandlerExceptionResolver` which handles `@ExceptionHandler` and `@ControllerAdvice` annotations, 
`ResponseStatusExceptionResolver` which handles `@ResponseStatus` annotation and 
`DefaultHandlerExceptionResolver` which handles predefined exceptions, such as `NoSuchRequestHandlingMethodException` or `HttpRequestMethodNotSupportedException`.

Since our custom exception does not have these annotations, it will go to `DefaultErrorAttributes` class.

In both the examples, we are not creating the `ResponseEntity` object, we are just returning the exception, some other class is creating the ResponseEntity Object. If we need full control and don't want to rely on Exception Resolvers, then we have to create the `ResponseEntity` object.

![ErrorResponse](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ErrorResponse.png)

We are throwing our `CustomException` and next we are catching the exception. Here we are creating the `ResponseEntity` by passing `ErrorResponse` object which contains Date, Message and Status.

![DefaultErrorAttribues](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/DefaultErrorAttribues.png)
![DefaultErrorAttribuesOutput](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/DefaultErrorAttribuesOutput.png)
---

## `ExceptionHandlerExceptionResolver`

Responsible for handling `@ExceptionHandler` and `@ControllerAdvice`

### `@ExceptionHandler`

The `@ExceptionHandler` annotation is used at the controller level to handle specific exceptions locally. When an exception is thrown in a controller method, Spring checks if any `@ExceptionHandler` method is present in the same controller to handle it. This allows for fine-grained control over exceptions within each controller.

![ControllerLevelException](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ControllerLevelException.png)

Here, the `@ExceptionHandler` intercepts `CustomException` in `UserController`, allowing a custom response to be returned. So what would happen here is that whenever in this controller, no matter how many methods you have, if any one method throws a `CustomException`, `handleCustomException` would catch it

Use-case just to show returning Error Response object instead of just message:
[ControllerLevelErrorResponse](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ControllerLevelErrorResponse.png)

Use-case just to show multiple `@ExceptionHandler` in single Controller class:
[MultipleExceptionHandlers](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/MultipleExceptionHandlers.png)

Use-case to show one `@ExceptionHandler` handling multiple exceptions:
[SingleExceptionHandler](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/SingleExceptionHandler.png)

Use-case just to show `@ExceptionHandler` not returning ResponseEntity and let **DefaultErrorAttributes** to create the ResponseEntity.
[DefaultErrorAttributesExceptionHandler](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/DefaultErrorAttributesExceptionHandler.png)

Function annotated with `@ExceptionHandler` can take  HTTP servlet response, HTTP request and Exception. If your `@ExceptionHandler` is accepting only one, or taking care of only one exception, you can write exact same exception in the input.

### `@ControllerAdvice`

Problem with Controller level @ExceptionHandler is:
- If multiple controller has the same type of Exceptions then same handling we might to do in multiple controller.
- Which is nothing but a code duplication.

For handling exceptions globally across multiple controllers, use `@ControllerAdvice`. This annotation lets us define exception-handling logic in a single class, preventing duplicate code across controllers.

[ControllerAdvice](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ControllerAdvice.png)

With `@ControllerAdvice`, any `CustomException` thrown across the application will be handled by `GlobalExceptionHandler`, unless a **specific `@ExceptionHandler` is defined in a controller**.

[ControllerLevel-vs-GlobalLevel](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ControllerLevel-vs-GlobalLevel.png)

What if there are two handlers which can handle an exception, which one will be given priority:

It will always follow an hierarch, from bottom to up(First look for exact match if not, check for its parent and so on...)

[MultipleHandlersToHandle](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/MultipleHandlersToHandle.png)
---

## @ResponseStatus Annotation

The `@ResponseStatus` annotation, applied directly to exception classes, allows you to define an HTTP status code and reason for specific exceptions without writing custom handlers. It’s particularly useful for custom exceptions, providing a standardized response across the application.

### Code Example: Defining @ResponseStatus on Custom Exception
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class CustomException extends RuntimeException {
    public CustomException(String message) {
        super(message);
    }
}
```

With `@ResponseStatus`, if a `CustomException` is thrown, Spring will return a `400 BAD REQUEST` status by default, eliminating the need for an `@ExceptionHandler`.

```java
@GetMapping("/bad-request")
public String badRequest() {
    throw new CustomException("Request is missing required fields");
}
```

In this example, calling `/bad-request` throws `CustomException`, which is handled by `ResponseStatusExceptionResolver` due to the `@ResponseStatus` annotation.

---

## Understanding DefaultHandlerExceptionResolver

The `DefaultHandlerExceptionResolver` is the last resolver in the sequence. It handles exceptions predefined by the Spring framework, such as:

- `NoSuchRequestHandlingMethodException`: For invalid URLs.
- `HttpRequestMethodNotSupportedException`: For unsupported HTTP methods.
- `MissingServletRequestParameterException`: For missing request parameters.

If none of the previous resolvers handle the exception, `DefaultHandlerExceptionResolver` attempts to match it to one of the predefined types, returning a standard HTTP error response if applicable.

### Code Example: Triggering DefaultHandlerExceptionResolver
```java
@RestController
public class DefaultHandlerDemoController {

    @GetMapping("/unsupported-method")
    public String unsupportedMethod() {
        throw new HttpRequestMethodNotSupportedException("POST");
    }
}
```

Calling `/unsupported-method` with an incorrect HTTP method would invoke `DefaultHandlerExceptionResolver`, which responds with a `405 METHOD NOT ALLOWED` error.

---

## Conclusion

Spring Boot’s exception handling framework offers a comprehensive way to manage errors through structured exception resolvers. By combining `@ExceptionHandler`, `@ControllerAdvice`, and `@ResponseStatus`, developers can create custom responses and handle errors across the application in a clean, maintainable manner. Using the built-in exception resolver hierarchy, Spring Boot also provides default handling for common exceptions, ensuring applications can respond gracefully to unexpected conditions.

For further details and examples, refer to the [Spring Documentation](https://spring.io/projects/spring-boot).
