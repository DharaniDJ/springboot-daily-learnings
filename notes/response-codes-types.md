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

## HTTP Response Codes

### 1xx: Informational

1xx status codes indicate that the request has been received and the process is continuing.

- **100 Continue**: The server has received the request headers, and the client should proceed to send the request body.
- **101 Switching Protocols**: The requester has asked the server to switch protocols, and the server has agreed to do so.

### 2xx: Success

2xx status codes indicate that the request was successfully received, understood, and accepted.

- **200 OK**: The request has succeeded. The meaning of success depends on the HTTP method.
- **201 Created**: The request has been fulfilled, resulting in the creation of a new resource.
- **202 Accepted**: The request has been accepted for processing, but the processing has not been completed.
- **204 No Content**: The server successfully processed the request, but is not returning any content.

Example:
```java
@GetMapping("/created")
public ResponseEntity<String> created() {
    return new ResponseEntity<>("Resource created", HttpStatus.CREATED);
}
```

### 3xx: Redirection

3xx status codes indicate that further action needs to be taken by the user agent to fulfill the request.

- **301 Moved Permanently**: The requested resource has been assigned a new permanent URI.
- **302 Found**: The requested resource resides temporarily under a different URI.
- **304 Not Modified**: Indicates that the resource has not been modified since the version specified by the request headers.

Example:
```java
@GetMapping("/redirect")
public ResponseEntity<Void> redirect() {
    return ResponseEntity.status(HttpStatus.MOVED_PERMANENTLY)
                         .header("Location", "https://new-url.com")
                         .build();
}
```

### 4xx: Client Error

4xx status codes indicate that the client seems to have erred.

- **400 Bad Request**: The server could not understand the request due to invalid syntax.
- **401 Unauthorized**: The client must authenticate itself to get the requested response.
- **403 Forbidden**: The client does not have access rights to the content.
- **404 Not Found**: The server can not find the requested resource.

Example:
```java
@GetMapping("/bad-request")
public ResponseEntity<String> badRequest() {
    return new ResponseEntity<>("Bad Request", HttpStatus.BAD_REQUEST);
}
```

### 5xx: Server Error

5xx status codes indicate that the server failed to fulfill a valid request.

- **500 Internal Server Error**: The server has encountered a situation it doesn't know how to handle.
- **502 Bad Gateway**: The server, while acting as a gateway or proxy, received an invalid response from the upstream server.
- **503 Service Unavailable**: The server is not ready to handle the request.

Example:
```java
@GetMapping("/server-error")
public ResponseEntity<String> serverError() {
    return new ResponseEntity<>("Internal Server Error", HttpStatus.INTERNAL_SERVER_ERROR);
}
```

## Conclusion

Understanding and using `ResponseEntity` and HTTP response codes effectively is crucial for building robust and user-friendly RESTful APIs in Spring Boot. By leveraging the different classes of HTTP response codes, you can provide meaningful feedback to clients and ensure that your API behaves predictably and consistently.