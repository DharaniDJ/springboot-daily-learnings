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
### Example
![custom-interceptor-1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-interceptor-1.png)

![custom-interceptor-2](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-interceptor-2.png)

### Output
![custom-interceptor-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/custom-interceptor-output.png)

### Explanation
we are creating a `UserController` class, we have `getUser` method with `/getUser` path. The `getUser` is invoking `user.getUser()` which is annotated with `@Component` which means bean is created by spring. Here `getUser` in `User` class is annotated with `@MyCustomAnnotation(name = "user")`.

Now to write an interceptor, we are creating a class `MyCustomInterceptor`. `@Aspect` means that this class will be considered as an interceptor where we will have a point cut and an advice. We have a method `invoke` where we are providing one point cut `@annotation(com.conceptandcoding.learningspringboot.CustomInterceptor.MyCustomAnnotation)` (which will look for the provided annotation). `@annotation` point cut means, run this advice/method whenever you see this annotation (i.e MyCustomAnnotation)

## Conclusion
Custom interceptors in Spring Boot provide a powerful way to process HTTP requests before they reach the controller and after they leave the controller. By creating custom annotations and interceptors, you can handle specific requirements such as logging, authentication, and modifying request/response objects. This guide has covered the creation and registration of custom interceptors with detailed code examples and explanations.
