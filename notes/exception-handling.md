# Spring Boot Exception Handling

## Class Hierarchy of Exception Handling

In Spring Boot, exception handling involves several classes. The hierarchy is as follows:

- `HandlerExceptionResolver` (interface)
  - `HandlerExceptionResolverComposite`
  - `ExceptionHandlerExceptionResolver`
  - `ResponseStatusExceptionResolver`
  - `DefaultHandlerExceptionResolver`

### Key Classes

1. **HandlerExceptionResolverComposite**: Orchestrates the other resolvers.
2. **ExceptionHandlerExceptionResolver**: Handles exceptions using `@ExceptionHandler` and `@ControllerAdvice`.
3. **ResponseStatusExceptionResolver**: Handles exceptions annotated with `@ResponseStatus`.
4. **DefaultHandlerExceptionResolver**: Handles predefined Spring framework exceptions.

## Flow Diagram of ExceptionResolvers involved in exception handling

![ExceptionResolvers Flow Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/ExceptionResolversFlow.png)

## Demo of ExceptionResolvers involved in exception handling

### Example Controller

```java
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/get-user")
    public String getUser() {
        throw new NullPointerException("Throwing NullPointerException for testing");
    }

    @GetMapping("/get-custom-user")
    public String getCustomUser() {
        throw new CustomException("Bad Request", "Request is not correct, user ID is missing");
    }
}
```

### Custom Exception

```java
public class CustomException extends RuntimeException {
    private final String status;
    private final String message;

    public CustomException(String status, String message) {
        super(message);
        this.status = status;
        this.message = message;
    }

    public String getStatus() {
        return status;
    }

    @Override
    public String getMessage() {
        return message;
    }
}
```

### Exception Handling Flow

1. **Exception Occurs**: The `DispatcherServlet` catches the exception.
2. **HandlerExceptionResolverComposite**: Passes the exception to the first resolver.
3. **ExceptionHandlerExceptionResolver**: Checks for `@ExceptionHandler` or `@ControllerAdvice`.
4. **ResponseStatusExceptionResolver**: Checks for `@ResponseStatus` on the exception.
5. **DefaultHandlerExceptionResolver**: Handles predefined exceptions.
6. **DefaultErrorAttributes**: Creates the final error response.

## @ExceptionHandler and @ControllerAdvice with Demo

### Controller Level Exception Handling

```java
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/get-user")
    public String getUser() {
        throw new CustomException("Bad Request", "Request is not correct, user ID is missing");
    }

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<String> handleCustomException(CustomException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.BAD_REQUEST);
    }
}
```

### Global Exception Handling

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<ErrorResponse> handleCustomException(CustomException ex) {
        ErrorResponse errorResponse = new ErrorResponse(new Date(), ex.getMessage(), ex.getStatus());
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }
}
```

### ErrorResponse Class

```java
public class ErrorResponse {
    private Date timestamp;
    private String message;
    private String status;

    public ErrorResponse(Date timestamp, String message, String status) {
        this.timestamp = timestamp;
        this.message = message;
        this.status = status;
    }

    // Getters and Setters
}
```

## @ResponseStatus annotation with Demo

### Custom Exception with @ResponseStatus

```java
@ResponseStatus(value = HttpStatus.BAD_REQUEST, reason = "Invalid Request")
public class CustomException extends RuntimeException {
    public CustomException(String message) {
        super(message);
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/get-user")
    public String getUser() {
        throw new CustomException("Request is not correct, user ID is missing");
    }
}
```

## Understand DefaultHandlerExceptionResolver

The `DefaultHandlerExceptionResolver` handles predefined Spring framework exceptions such as:

- `NoSuchRequestHandlingMethodException`
- `HttpRequestMethodNotSupportedException`
- `HttpMediaTypeNotSupportedException`
- `MissingServletRequestParameterException`

### Example

```java
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/get-user")
    public String getUser() {
        throw new NoSuchRequestHandlingMethodException("GET", UserController.class);
    }
}
```

### Response

```json
{
    "timestamp": "2023-10-01T12:34:56.789+00:00",
    "status": 404,
    "error": "Not Found",
    "message": "No handler found for GET /api/get-user",
    "path": "/api/get-user"
}
```

By understanding and utilizing these exception handling mechanisms, you can create robust and user-friendly error handling in your Spring Boot applications.