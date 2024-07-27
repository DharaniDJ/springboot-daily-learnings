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

- **Feature Toggles**: Enable or disable features based on properties. (i.e for example, we want to use either `MySQLConnection` or `NoSQLConnection`)
- **Environment-Specific Beans**: Create beans only in specific environments (e.g., development, production). (i.e for example, development uses `MySQLConnection` and production uses `NoSQLConnection`)
- **Conditional Configuration**: Apply different configurations based on property values.

## ConditionalOnProperty with JAVA
To use the `@ConditionalOnProperty` annotation, you need to specify the property name and optionally the expected value. If the property is present and matches the expected value, the bean will be created; otherwise, it will be skipped.

### Example: Using Prefix
```java
@Component
public class DBConnection {
    @Autowired(required=false)      
    // when we are creating a bean of DB Connection, if mySQLConnection has required=true, the dependency should get resolved.
    // If required=false, it is telling to spring that the object might not be present (Don't consider as Fast Fail)
    MySQLConnection mySQLConnection;
    
    @Autowired(required=false)
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
@ConditionalOnProperty(prefix="nosqlconnection", value="enable", havingValue="true", matchIfMissing=false)
// matchIfMissing=false - If this key/property is not present in the application.properties, it will consider as false and it will not create the bean
public class NoSQLConnection {
    NoSQLConnection(){
        System.out.println("Initialization of NoSQLConnection Bean");
    }
}
```
```java
@Component
@ConditionalOnProperty(prefix="sqlconnection", value="enable", havingValue="true", matchIfMissing=false)
public class MySQLConnection {
    MySQLConnection(){
        System.out.println("Initialization of NoSQLConnection Bean");
    }
}
```
```
application.properties

// prefix.value=havingValue
sqlconnection.enabled=true (prefix="sqlconnection", value="enable", havingValue="true")
```

### Explanation
When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `MySQLConnection` and it is eagarly initialized as it has `Singleton` scope. It also has `@ConditionalOnProperty`, so it will make a key using its prefix and value and look into the `application.properties` files. It gets the value from the file and compare the string with `havingValue`. If it matches, spring will create its bean.

Now lets say, it goes to `NoSQLConnection` and it is eagarly initialized as it has `Singleton` scope. It also has `@ConditionalOnProperty`, so it will make a key using its prefix and value and look into the `application.properties` files. It gets the value from the file and compare the string with `havingValue`. If it matches, spring will create its bean. If the match is missing, it will create based on `matchIfMissing` attributes from the `@ConditionalOnProperty`.

Now it finds `DBConnection` which is `Singleton`, it will create the bean. Now it will inject the dependencies i.e for `NoSQLConnection` it will be `NULL`, and for  `MySQLConnection` it will `not be NULL`, and goes to `@PostConstruct` lifecycle.

## Advantages and Disadvantages

### Advantages
- **Flexibility**: Allows for flexible configuration of beans based on properties.
- **Environment-Specific Configuration**: Easily manage different configurations for different environments. Avoid cluttering application context with un-necessary beans.
- **Feature Management**: Simplifies the process of enabling or disabling features.
- **Save Memory**: During the application startup, all the beans get created and is stored. When we are controlling the creation, and when we are not creating bean of the unnecessary classe. then we are saving the memory.
- **Reduce application Startup time**: During the application startup, it will create all the beans. This will take a lot of time, so if we are conditionally creating the beans, it reduces the application startup time.

### Disadvantages
- **Misconfiguration**: Misconfiguration can happen if we mistakenly put wrong values in the application.properties file.
- **Complexity**: Can add complexity to the configuration if overused.
- **Hidden Dependencies**: Property-based conditions can sometimes make it harder to understand the dependencies and flow of the application.
- **Debugging**: Troubleshooting issues related to conditional beans can be more challenging.

## Conclusion
The `@ConditionalOnProperty` annotation is a powerful tool in Spring Boot that allows for conditional bean creation based on configuration properties. It provides flexibility and control over the application's configuration, making it easier to manage different environments and feature toggles. However, it should be used judiciously to avoid adding unnecessary complexity to the application.
