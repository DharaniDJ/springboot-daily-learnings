# Bean Scopes in Spring Boot

## Introduction
In Spring Boot, a bean is an object that is managed by the Spring IoC (Inversion of Control) container. The scope of a bean defines the lifecycle and visibility of that bean in the contexts we use it. Spring Boot supports several types of bean scopes, each serving different purposes.

## Singleton Bean Scope
- The singleton scope is the default scope in Spring. 
- When a bean is defined with the singleton scope, the container creates a single instance of that bean, and all requests for that bean name will return the same object. (i.e Only 1 instance created per IOC)
- Eagerly initialized by IOC (means at the time of application startup, object get created)

### Example
```java
@RestController
@RequestMapping(value="/api/")
// No Scope is defined, so by default it is Singleton scope
public class TestController1 {

    @Autowired
    User user;

    public TestController1(){
        System.out.println("TestController1 instance initialization");
    }

    @PostContruct
    public void init(){
        System.out.println("TestController1 object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode());
    }

    @GetMapping(path="/fetchUser")
    public ResponseEntity<String> getUserDetails(){
        System.out.println("fetchUser api invoked");
        return ResponseEntity.status(HttpStatus.OK).body("");
    }
}
```
```java
@RestController
@RequestMapping(value="/api/")
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON) // Explicitly mentioned singleton scope using ENUM
public class TestController2 {

    @Autowired
    User user;

    public TestController2(){
        System.out.println("TestController2 instance initialization");
    }

    @PostContruct
    public void init(){
        System.out.println("TestController2 object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode());
    }

    @GetMapping(path="/fetchUser2")
    public ResponseEntity<String> getUserDetails(){
        System.out.println("fetchUser2 api invoked");
        return ResponseEntity.status(HttpStatus.OK).body("");
    }
}
```
```java
@Component
@Scope("singleton")     // Defined the scope using string "singleton"
public class User {
    public User(){
        System.out.println("User instance initialization");
    }

    @PostConstruct
    public void init(){
        System.out.println("User object hashCode: " + this.hashCode());
    }
}
```

```
At the start of Application

TestController1 instance initialization
User instance initialization
User object hashCode: 1140202235
TestController1 object hashCode: 1046302571 User object hashCode: 1140202235
TestController2 instance initialization
TestController1 object hashCode: 1525241607 User object hashCode: 1140202235
```
### Explanation
When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `TestController1` and it is eagarly initialized as it has `Singleton` scope. It will call its constructor, but this object is not fully constructed yet. We need to inject dependencies into the constructed bean. So `User` initialization happens and since `User` does not have any dependencies, it will go to `@PostConstruct` life cycle. At this point `User` object got created. In the `TestController1`, `User` dependency will be injected. Now there are no more dependencies left, so it will go to `@PostConstruct` life cycle of `TestController1`.

Now the IOC will see if there are any class for which it is eagarly initialized. And let say it found `TestController2`. It will call its constructor and now it injects the dependencies i.e `User`. Since `User` is singleton which means only one object would be created and its already created during `TestController1` dependency injection. So same object is injected and after that it will go to `@PostContruct` life cycle as there are no more `@Autowired`.

Now if we are invoking an API (fetchUser or fetchUser1), no object construction happends. It will simply invoke the particular API and all the objects are created before the API call itself.

## Prototype Bean Scope
- Return a new instance every time it is requested from the container.
- It's Lazily initialized, means object is created only when its required.

### Example
```java
@RestController
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@RequestMapping(value="/api/")
public class TestController1 {

    @Autowired
    User user;

    @Autowired
    Student student;

    public TestController1(){
        System.out.println("TestController1 instance initialization");
    }

    @PostContruct
    public void init(){
        System.out.println("TestController1 object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode() + " Student object hashCode: " + student.hashCode());
    }

    @GetMapping(path="/fetchUser")
    public ResponseEntity<String> getUserDetails(){
        System.out.println("fetchUser api invoked");
        return ResponseEntity.status(HttpStatus.OK).body("");
    }
}
```
```java
// Singleton
@Component
public class Student {

    @Autowired
    User user;

    public Student(){
        System.out.println("Student instance initialization");
    }

    @PostConstruct
    public void init(){
        System.out.println("Student object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode());
    }
}
```
```java
@Component
@Scope("prototype")
public class User {

    public User(){
        System.out.println("User instance initialization");
    }

    @PostConstruct
    public void init(){
        System.out.println("User object hashCode: " + this.hashCode());
    }
}
```
```
At the start of Application

Student instance initialization
User instance initialization
User object hashCode: 1510009630
Student object hashCode: 2092450685 User object hashCode: 1510009630

Invoke localhost:8090/api/fetchUser

TestController1 instance initialization
User instance initialization
User object hashCode: 1984730322
TestController1 object hashCode: 1786739287 User object hashCode: 1984730322 Student object hashCode: 2092450685
```

### Explanation
When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it start scanning from `TestController1`, since it is `Prototype` scope, it will be lazily initialized. It won't create an object until its not used. Next lets say it found `User`, since it is `Prototype` scope, it will be lazily initialized. Now it found `Student` which is singleton, it will call its constructor, but this object is not fully constructed yet. We need to inject dependencies into the constructed bean. So `User` initialization happens and since `User` does not have any dependencies, it will go to `@PostConstruct` life cycle. At this point `User` object got created. In the `Student`, `User` dependency will injected. Now there are no more dependencies left, so it will go to `@PostConstruct` life cycle of `Student`.

Now when we invoke the `fetchUser` API, `TestController1` object would be created by IOC. `TestController1` is a Prototype which means IOC has to create a new object of `TestController1`. Now it will call its constructor, next it will look for dependency resolutions. Since `User` is marked as `Prototype`, it creates a new object. And since `Student` is marked as `Singleton`, it was eagerly initialized, it would get injected. Now there are no more dependencies left, so it will go to `@PostConstruct` life cycle of `TestController1`.

## Request Bean Scope
- The request scope is used in web applications. 
- A bean with the request scope will be created and available for the duration of a single HTTP request.
- Lazily initialized

### Example
```java
@RestController
@Scope("request")
@RequestMapping(value="/api/")
public class TestController1 {

    @Autowired
    User user;

    @Autowired
    Student student;

    public TestController1(){
        System.out.println("TestController1 instance initialization");
    }

    @PostContruct
    public void init(){
        System.out.println("TestController1 object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode() + " Student object hashCode: " + student.hashCode());
    }

    @GetMapping(path="/fetchUser")
    public ResponseEntity<String> getUserDetails(){
        System.out.println("fetchUser api invoked");
        return ResponseEntity.status(HttpStatus.OK).body("");
    }
}
```
```java
@Component
@Scope("prototype")
public class Student {

    @Autowired
    User user;

    public Student(){
        System.out.println("Student instance initialization");
    }

    @PostConstruct
    public void init(){
        System.out.println("Student object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode());
    }
}
```
```java
@Component
@Scope("request")
public class User {

    public User(){
        System.out.println("User instance initialization");
    }

    @PostConstruct
    public void init(){
        System.out.println("User object hashCode: " + this.hashCode());
    }
}
```
```
No object is created at the start of Application

Invoke localhost:8090/api/fetchUser

TestController1 instance initialization
User instance initialization
User object hashCode: 39793904
Student instance initialization
Student object hashCode: 275139209 User object hashCode: 39793904
TestController1 object hashCode: 898967761 User object hashCode: 39793904 Student object hashCode: 275139209
fetchUser api invoked

Invoke localhost:8090/api/fetchUser

TestController1 instance initialization
User instance initialization
User object hashCode: 1227388929
Student instance initialization
Student object hashCode: 1206886228 User object hashCode: 1227388929
TestController1 object hashCode: 1137709937 User object hashCode: 1227388929 Student object hashCode: 1206886228
fetchUser api invoked
```

### Explanation
When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it start scanning from `TestController1`, since it is `Request` scope, it will be lazily initialized. It won't create an object until its not used. Next lets say it found `Student`, since it is `Prototype` scope, it will be lazily initialized. Now it found `User` which has `Request` scope, it will also be lazily initialized. So no object is created at the application startup.

Now when we invoke the `fetchUser` API, IOC has to create an  object for `TestController1`. For this Http request, `TestController1` initialization happens and next it will look to resolve dependencies. So it goes to `User` which has `Request` scope, so only one instance or new instance should be created for each request. Since the current request does not have any `User` object, it will create a new `User` object. There are no dependencies for `User` object, so it will go to `@PostConstruct` life cycle of `User`. `User` will get injected into `TestController1`.

For `TestController1`, next it will try to resolve `Student` dependency. No matter whether this is same request or different request, it will create a new `Student` object. It will initialize `Student` object and it will try to resolve `User` dependency in `Student`. We already have a `User` object created for the current request, so student will use the existing object and no new object will be created.

Now when we invoke the `fetchUser` API for the second time, a new object will be created for `TestController1` for this Http request. It will go through the same process similar as the first http request.

## Issue with Request Bean
Consider below, What will happen?

```java
@RestController
@Scope("singleton")
@RequestMapping(value="/api/")
public class TestController1 {

    @Autowired
    User user;

    public TestController1(){
        System.out.println("TestController1 instance initialization");
    }

    @PostContruct
    public void init(){
        System.out.println("TestController1 object hashCode: " + this.hashCode() + " User object hashCode: " + user.hashCode());
    }

    @GetMapping(path="/fetchUser")
    public ResponseEntity<String> getUserDetails(){
        System.out.println("fetchUser api invoked");
        return ResponseEntity.status(HttpStatus.OK).body("");
    }
}
```
```java
@Component
@Scope("request")
public class User {

    public User(){
        System.out.println("User instance initialization");
    }

    @PostConstruct
    public void init(){
        System.out.println("User object hashCode: " + this.hashCode());
    }
}
```
```
Error creating bean with name 'testcontroller1': Unsatisfied dependency
```

## Explanation
We have a class `TestController1` marked as `Singleton` and class `User` marked as `Request`. `TestController1` is dependent on `User`

When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `TestController1` and it is eagarly initialized as it has `Singleton` scope. It will call its constructor, but this object is not fully constructed yet. We need to inject dependencies into the constructed bean. 
It goes to `User` and search for any active HTTP requests during the application startup. But there is no HTTP request present, so spring will not create its object and that's why it will get failed.

## ProxyMode Purpose and Usage
ProxyMode is used to create a proxy for a bean, which can be useful in certain scenarios, such as when using scoped beans in singleton beans.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.ScopedProxyMode;

@Configuration
public class AppConfig {

    @Bean
    @Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public MyBean mySessionBean() {
        return new MyBean();
    }
}
```

### Explanation
In this example, `mySessionBean` is defined with a session scope and a proxy mode. The proxy mode ensures that the correct instance of the bean is injected at runtime.

## Session Bean Scope
The session scope is also used in web applications. A bean with the session scope will be created and available for the duration of an HTTP session.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.annotation.SessionScope;

@Configuration
public class AppConfig {

    @Bean
    @SessionScope
    public MyBean mySessionBean() {
        return new MyBean();
    }
}
```

### Explanation
Here, `mySessionBean` is defined with the session scope. A new instance of the bean will be created for each HTTP session.

## Application Bean Scope
The application scope is used in web applications. A bean with the application scope will be created and available for the duration of a ServletContext.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.annotation.ApplicationScope;

@Configuration
public class AppConfig {

    @Bean
    @ApplicationScope
    public MyBean myApplicationBean() {
        return new MyBean();
    }
}
```

### Explanation
In this example, `myApplicationBean` is defined with the application scope. A single instance of the bean will be created for the entire ServletContext.

## WebSocket Bean Scope
The WebSocket scope is used in web applications that use WebSockets. A bean with the WebSocket scope will be created and available for the duration of a WebSocket session.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.context.annotation.WebSocketScope;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    @WebSocketScope
    public MyBean myWebSocketBean() {
        return new MyBean();
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // Register WebSocket handlers here
    }
}
```

### Explanation
Here, `myWebSocketBean` is defined with the WebSocket scope. A new instance of the bean will be created for each WebSocket session.
