# Bean Scopes in Spring Boot

## 1. Introduction
In Spring Boot, a bean is an object that is managed by the Spring IoC (Inversion of Control) container. The scope of a bean defines the lifecycle and visibility of that bean in the contexts we use it. Spring Boot supports several types of bean scopes, each serving different purposes.

## 2. Singleton Bean Scope
The singleton scope is the default scope in Spring. When a bean is defined with the singleton scope, the container creates a single instance of that bean, and all requests for that bean name will return the same object.

### Example
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;

@Configuration
public class AppConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    public MyBean mySingletonBean() {
        return new MyBean();
    }
}
```

### Explanation
In the above example, `mySingletonBean` is defined with the singleton scope. This means that every time the bean is requested, the same instance will be returned.

## 3. Prototype Bean Scope
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

## 4. Request Bean Scope
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

## 5. ProxyMode Purpose and Usage
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

## 6. Session Bean Scope
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

## 7. Application Bean Scope
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

## 8. WebSocket Bean Scope
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
