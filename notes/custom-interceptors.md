# Custom Interceptor in Spring Boot

## Introduction
In Spring Boot, interceptors are used to intercept and process HTTP requests before they reach the controller and after they leave the controller. This allows you to perform operations such as logging, authentication, and modifying request/response objects. Custom interceptors can be created to handle specific requirements that are not covered by the default interceptors.

## Custom Interceptor for Requests Before Reaching the Controller
A custom interceptor can be used to process requests before they reach the specific controller class. This is useful for tasks such as logging request details, checking authentication, or modifying the request object.

![dispatcher-servlet](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/dispatcher-servlet.png)

- The `Request` first comes to `Servlet Container` where our application is deployed.
- The `Servlet Container` then calls the `Dispatcher Servlet` which is the entry point and it chooses the controller.
- So we want a custom interceptor before reaching to our particular controller, so our code of custom interceptor should go somewhere in between `DispatcherServlet` and `invoking the controller method`.

### Example

![custom-interceptor-before-controller-code](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-interceptor-before-controller-code.png)

- we implemented `MyCustomInterceptor` with `HandlerInterceptor`, we need to Override `preHandle`, `postHandle`, `afterComplete`.
- we have to register this `MyCustomInterceptor` for which API it has to invoke. For that we have to implement `WebMvcConfigurer` for `AppConfig`.
- we have to Override `addInterceptor` method, and inside this method we have to do a registry, so this is the parameter it accepts.
- Since we have `@component` for `MyCustomInterceptor` class, bean is already created and we also have `@Autowired` in `AppConfig`.

![custom-interceptor-before-controller-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-interceptor-before-controller-output.png)

## Custom Interceptor for Requests After Reaching the Controller
A custom interceptor can also be used to process requests after they have been handled by the controller. This is useful for tasks such as logging response details, modifying the response object, or performing cleanup operations. Let's break it into 2 steps.

## Step 1: Creation of Custom Annotation
we can create Custom Annotation using keyword `@interface` java Annotation.

### Example
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyCustomAnnotation {
}
```

```java
public class User {
    @MyCustomAnnotation
    public void updateUser(){
        // some business logic
    }
}
```

### Types of Meta Annotation Properties with Code Examples
- **@Target**: Specifies the kinds of elements an annotation type can be applied to(like method or class or constructor).

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD})
public @interface MyCustomAnnotation {
}
```

- **@Retention**: Specifies how long annotations with the annotated type are to be retained.
![@retention](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/@retention.png)

```
1. RententionPolicy.SOURCE:  Annotation will be discarded by compiler itself and its not even recorded in .class file.
2. RententionPolicy.CLASS:   Annotation will be recorded in .class file but ignored by JVM during run time.
3. RententionPolicy.RUNTIME: Annotation will be recorded in .class file and also available during run time.
```

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD})
public @interface MyCustomAnnotation {
}
```

### How to create Custom Annotation with methods (more like a fields):
- No parameter, no body
- Return type is restricted to:
    - Primitive type (int, boolean, double...etc).
    - String
    - Enum
    - Class<?>
    - Annotation
    - Array of above types

![custom-annotation-return-types](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-annotation-return-types.png)

## Step 2: Creation of Custom Interceptor
To create a custom interceptor, you need to implement the `HandlerInterceptor` interface and override its methods. You can then register the interceptor with Spring Boot.

### Example
```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class CustomInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("Custom Interceptor: Pre Handle method is Calling");
        // Perform operations before the request reaches the controller
        return true; // Continue with the next interceptor or the controller
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("Custom Interceptor: Request and Response is completed");
        // Perform operations after the request has been handled by the controller
    }
}
```

### Registering the Interceptor
You need to register the custom interceptor with Spring Boot by adding it to the `WebMvcConfigurer`.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private CustomInterceptor customInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(customInterceptor);
    }
}
```

## Conclusion
Custom interceptors in Spring Boot provide a powerful way to process HTTP requests before they reach the controller and after they leave the controller. By creating custom annotations and interceptors, you can handle specific requirements such as logging, authentication, and modifying request/response objects. This guide has covered the creation and registration of custom interceptors with detailed code examples and explanations.
