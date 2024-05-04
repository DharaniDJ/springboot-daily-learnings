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
@RequestMapping(value="/api")
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
@RequestMapping(value="/api")
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
TestController1 instance initialization
User initialization
User object hashCode: 1140202235
TestController1 object hashCode: 1046302571 User object hashCode: 1140202235
TestController2 instance initialization
TestController1 object hashCode: 1525241607 User object hashCode: 1140202235
```
### Explanation
When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `TestController1` and it is eagarly initialized as it has `Singleton` scope. It will call the constructor, but this object is not fully constructed yet. We need to inject dependencies into the constructed bean. So `User` initialization happens and since `User` does not have any dependencies, it will go to `@PostConstruct` life cycle. At this point `User` object got created. In the `TestController1`, `User` dependency was injected. Now there are no more dependencies left, so it will go to `@PostConstruct` life cycle of `TestController1`.

Now the IOC will see if there are any class for which it is eagarly initialized. And let say it found `TestController2`. It will call the constructor and now it injects the dependencies i.e `User`. Since `User` is singleton which means only one object would be created and its already created during `TestController1` dependency injection. So same object is injected and after that it will go to `@PostContruct` life cycle as there are no more `@Autowired`.

Now if we are invoking an API (fetchUser or fetchUser1), no object construction happends. It will simply invoke the particular API and all the objects are created before the API call itself.

## Prototype Bean Scope
A bean with the prototype scope will return a new instance every time it is requested from the container.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class AppConfig {

    @Bean
    @Scope("prototype")
    public MyBean myPrototypeBean() {
        return new MyBean();
    }
}
```

### Explanation
In this example, `myPrototypeBean` is defined with the prototype scope. Each time the bean is requested, a new instance will be created.

## Request Bean Scope
The request scope is used in web applications. A bean with the request scope will be created and available for the duration of a single HTTP request.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.annotation.RequestScope;

@Configuration
public class AppConfig {

    @Bean
    @RequestScope
    public MyBean myRequestBean() {
        return new MyBean();
    }
}
```

### Explanation
Here, `myRequestBean` is defined with the request scope. A new instance of the bean will be created for each HTTP request.

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
