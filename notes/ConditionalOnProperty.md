# @ConditionalOnProperty Annotation

## Definition
The `@ConditionalOnProperty` annotation in Spring Boot is used to conditionally enable or disable a bean based on the presence and value of a specific property in the application's configuration. This annotation is part of the `org.springframework.boot.autoconfigure.condition` package and is commonly used in auto-configuration classes to control bean creation.

In a large scale application, where there are thousands of beans and many of those beans are initialized during application startup. So it will clutter your application context with the unnecessary beans also, which might not require. That's where this annotation comes into picture.

## Example
```java
@Component
public class DBConnection {
    @Autowired
    MySQLConnection mySQLConnection;
    
    @Autowired
    NoSQLConnection noSQLConnection;

    @PostConstruct
    public void init(){
        System.out.println("DB Connection Bean Created with dependencies below:");
        System.out.println("is MySQLConnection object null: " + Objects.isNull(mySQLConnection));
        System.out.println("is NoSQLConnection object null: " + Objects.isNull(noSQLConnection));
    }
}
```
```java
@Component
public class NoSQLConnection {
    NoSQLConnection(){
        System.out.println("Initialization of NoSQLConnection Bean");
    }
}
```
```java
@Component
public class MySQLConnection {
    MySQLConnection(){
        System.out.println("Initialization of NoSQLConnection Bean");
    }
}
```
```
Initialization of NoSQLConnection Bean
Initialization of MySQLConnection Bean
DB Connection Bean Created with dependencies below:
is MySQLConnection object null: false
is NoSQLConnection object null: false
```
## Explanation
When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `NoSQLConnection` and it is eagarly initialized as it has `Singleton` scope. Since it does not have any dependencies, `NoSQLConnection` object is initialized. Same happens with `MySQLConnection` class.

Now it finds `DBConnection` which is `Singleton`, it will create the bean. Now it will inject the dependencies `NoSQLConnection`, `MySQLConnection` and goes to `@PostConstruct` lifecycle.

## Use Cases Where This Annotation Helps
The `@ConditionalOnProperty` annotation is particularly useful in scenarios where you want to enable or disable certain features or beans based on configuration properties. Some common use cases include:

- **Feature Toggles**: Enable or disable features based on properties.
- **Environment-Specific Beans**: Create beans only in specific environments (e.g., development, production).
- **Conditional Configuration**: Apply different configurations based on property values.

## ConditionalOnProperty with JAVA
To use the `@ConditionalOnProperty` annotation, you need to specify the property name and optionally the expected value. If the property is present and matches the expected value, the bean will be created; otherwise, it will be skipped.

### Example 1: Basic Usage
```java
@Configuration
public class MyServiceConfig {

    @Bean
    @ConditionalOnProperty(name = "my.service.enabled", havingValue = "true", matchIfMissing = true)
    public MyService myService() {
        return new MyService();
    }
}
```
In this example, the `myService` bean will only be created if the property `my.service.enabled` is set to `true` or if the property is missing.

### Example 2: Using Prefix
```java
@Configuration
public class MyServiceConfig {

    @Bean
    @ConditionalOnProperty(prefix = "my.service", name = "enabled", havingValue = "true")
    public MyService myService() {
        return new MyService();
    }
}
```


Here

, the `myService` bean will be created if the property `my.service.enabled` is set to `true`.

## Advantages and Disadvantages

### Advantages
- **Flexibility**: Allows for flexible configuration of beans based on properties.
- **Environment-Specific Configuration**: Easily manage different configurations for different environments.
- **Feature Management**: Simplifies the process of enabling or disabling features.

### Disadvantages
- **Complexity**: Can add complexity to the configuration if overused.
- **Hidden Dependencies**: Property-based conditions can sometimes make it harder to understand the dependencies and flow of the application.
- **Debugging**: Troubleshooting issues related to conditional beans can be more challenging.

## Conclusion
The `@ConditionalOnProperty` annotation is a powerful tool in Spring Boot that allows for conditional bean creation based on configuration properties. It provides flexibility and control over the application's configuration, making it easier to manage different environments and feature toggles. However, it should be used judiciously to avoid adding unnecessary complexity to the application.
