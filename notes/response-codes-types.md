# Spring Boot ResponseEntity and Response Codes

## Introduction

In Spring Boot, `ResponseEntity` is a powerful tool for handling HTTP responses. It allows you to control the HTTP status code, headers, and body of the response. This flexibility is essential for building robust RESTful APIs. HTTP response codes are categorized into five classes: 1xx, 2xx, 3xx, 4xx, and 5xx. Each class represents a different type of response.

Response generally contains 3 parts:
- **Status Code**: HTTP return code like 200 OK, 500 INTERNAL_ERROR etc.
- **Header**: Additional information (Optional).
- **Body**: Data need to be sent in response.

We can use `ResponseEntity<T>` to create Response and in this `T` represents the type of the `Body`.

## ResponseEntity in Spring Boot

`ResponseEntity` is a generic class that extends `HttpEntity` and adds a status code. It is used to represent the entire HTTP response, including the status code, headers, and body.

![response-entity](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/response-entity.png)

Body should be last, actually its kind of usign Builder design pattern, so `status`, `header` all are returning Builder object and `body` method call returns the ResponseEntity object.

![header](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/header.png)

When we don't want to add any body in the response, we should use `build` method. `<T> ResponseEntity <T> build()`

```java
@GetMapping(path="/get-user")
public ResponseEntity<Void> getUser(){
    HttpHeaders headers = new HttpHeaders();
    headers.add("header1", "value1");
    headers.add("header2", "value2");

    return ResponseEntity.status(HttpStatus.OK).headers(headers).build();
}
```

### Example

By default, 200 OK is the status code set

```java
@RestController
@RequestMapping(path="/api")
public class UserController{

    @GetMapping(path="/get-user")
    public User getUser(){
        User responseObj = new User("Dharani", 26);
        return responseObj;
    }
}
```
## @ResponseBody in Spring Boot
When we return plain string or POJO directly from the class, then `@ResponseBody` annotation is required

Why?
It tells to considered value as `Response Body` and not as a `View`

In the above example, we did not use `@ResponseBody`, but `@RestController` automatically puts `@ResponseBody` to all the methods.

![rest-controller](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/rest-controller.png)

But lets say, if we use `@Controller` instead, then below code will throw exception.

![controller](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/controller.png)

Because, Return value is treated as "view" and spring boot will try to look for file with the given name "XYZ" which do not exist.

## HTTP Response Codes

![response-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/response-code.png)

### 1xx: Informational

1xx status codes indicate that the request has been received and the process is continuing.

![1xx status codes](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/1xx.png)

### 2xx: Success

2xx status codes indicate that the request was successfully received, understood, and accepted.

![2xx status codes](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/2xx.png)

### 3xx: Redirection

3xx status codes indicate that further action needs to be taken by the user agent to fulfill the request.

![3xx status codes](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/3xx.png)

### 4xx: Client Error

4xx status codes indicate that the client seems to have erred.

![4xx status codes](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/4xx.png)

### 5xx: Server Error

5xx status codes indicate that the server failed to fulfill a valid request.

![5xx status codes](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/5xx.png)

## Conclusion

Understanding and using `ResponseEntity` and HTTP response codes effectively is crucial for building robust and user-friendly RESTful APIs in Spring Boot. By leveraging the different classes of HTTP response codes, you can provide meaningful feedback to clients and ensure that your API behaves predictably and consistently.