# Spring Boot: Bean and its Lifecycle | Inversion of Control (IOC)

## What is a Bean?
A bean is an object that is managed by the Spring IoC container. It can be a simple object or a complex object that has dependencies on other beans.

A bean in Spring Boot is a managed object that is instantiated, assembled, and managed by the Spring framework. It is the basic building block of a Spring application.

IOC Container contains all the beans which get created and also manage them.


## How to Create a Bean (Component Annotation)
To create a bean using the `@Component` annotation, follow these steps:
1. Annotate the class with `@Component`.
2. Enable component scanning in your Spring Boot application.
3. Spring will automatically detect and instantiate the bean.

- @Component annotation follows "convention over configuration" approach.
- Means Springboot will try to auto configure based on conventions reducing the need for explicit configuration.
- @Controller, @Service etc. all are internally tells Spring to create bean and manage it.

Here, Spring boot will internally call new User() to create an instance of this class.

Example code snippet:
```java
@Component
public class User {
    // Bean implementation
    String username;
    String email;

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

But what will happen to this now:

```java
@Component
public class User {
    // Bean implementation
    String username;
    String email;

    public User(String username, String email){
        this.username = username;
        this.email = email;
    }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

Adding @Component tells Spring boot to create its object. Now Spring boot has its autoconfigured that it has to call default constructor.

But Spring is not able to create its object, because there is no default constructor and Spring doesn't know what to pass in for the username and email.

The application will fail to start as a consequence. @Bean annotation comes into the picture, where we provide the configuration details and tells Spring boot to use it while creating a Bean.

## How to Create a Bean (Bean and Configuration Annotation)
To create a bean using the `@Bean` and `@Configuration` annotations, follow these steps:
1. Create a configuration class and annotate it with `@Configuration`.
2. Define a method in the configuration class and annotate it with `@Bean`.
3. The method should return an instance of the bean.
4. Spring will automatically instantiate and manage the bean.

Example code snippet:
```java
@Configuration
public class AppConfig {
    @Bean
    public User createUserBean() {
        // This gets more priority over default configuration. Because this is what we are providing, and what we want is first priority to spring boot, not autoconfiguration.
        return new User("deafultusername", "defaultemail");
    }

    @Bean
    public User createAnotherUserBean() {
        
        return new User("anotherusername", "anotheremail");
    }
}
```

- Let's say we have 2 `@Bean` methods, where both returning user. What would happen here?
- In this case, Spring boot goes one by one and create a bean for all. In this case, 2 objects would get created.

## Use Cases for @Bean Annotation

There are certain use cases where we can only use the `@Bean` annotation and the `@Component` annotation might not be suitable. Let's explore these use cases in detail:

1. Third-Party Libraries:
    When working with third-party libraries that provide their own configuration classes, we cannot directly annotate them with `@Component`. In such cases, we can use the `@Bean` annotation to create and configure instances of these third-party classes.

    Example:
    ```java
    @Configuration
    public class ThirdPartyConfig {
         @Bean
         public ThirdPartyLibrary thirdPartyLibrary() {
              return new ThirdPartyLibrary();
         }
    }
    ```

2. Custom Configuration:
    Sometimes, we need to create beans with custom configurations that cannot be achieved using the `@Component` annotation alone. In such cases, we can define a configuration class and use the `@Bean` annotation to create and customize the beans.

    Example:
    ```java
    @Configuration
    public class CustomConfig {
         @Bean
         public CustomBean customBean() {
              CustomBean bean = new CustomBean();
              // Custom configuration logic
              return bean;
         }
    }
    ```

3. Conditional Bean Creation:
    There might be scenarios where we need to conditionally create beans based on certain conditions or properties. The `@Bean` annotation allows us to define conditional logic using the `@Conditional` annotation, which is not possible with the `@Component` annotation alone.

    Example:
    ```java
    @Configuration
    public class ConditionalConfig {
         @Bean
         @Conditional(DevEnvironmentCondition.class)
         public MyBean devBean() {
              return new MyBean("Dev Bean");
         }
    
         @Bean
         @Conditional(ProdEnvironmentCondition.class)
         public MyBean prodBean() {
              return new MyBean("Prod Bean");
         }
    }
    ```

In summary, the `@Bean` annotation provides more flexibility and control over bean creation and configuration compared to the `@Component` annotation. It allows us to work with third-party libraries, create custom configurations, and conditionally create beans based on specific requirements.

Now, we know what is bean and how its getting created. But few questions comes to our minds:

- How Spring boot find these beans?
- At what time, these beans get created?

## ComponentScan Annotation
The `@ComponentScan` annotation is used to enable component scanning in Spring Boot. It tells Spring where to look for components, such as beans, to be managed.

Example code snippet:
```java
@SpringBootApplication // SpringBootApplication also has @Component annotation and its auto configuration is that start from the package where this class is present and goes further sub classes.
@ComponentScan(basePackages = "com.example.learningspringboot")
public class SpringbootApplication {
    public static void main(String[] args){
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

Through Explicit defining of bean via @Bean annotation in @Configuration class.
```java
@Configuration
public class AppConfig {
    @Bean
    public User createUserBean(){
        return new User("defaultusername", "defaultemail");
    }
}
```

## Eagerly Initialized and Lazy-Initialized Bean (Lazy Annotation)
By default, beans are eagerly initialized, meaning they are instantiated when the application starts. For example, Beans with singleton scope are eagerly initialized.

However, you can use the `@Lazy` annotation to make a bean lazy-initialized, meaning it will be instantiated only when it is first requested. For example, Beans with scope like prototype etc. are lazily initialized. or Beans with @Lazy annotation.

Example code snippet:
```java
@Component
@Lazy
public class Order {
    // Bean implementation
    public Order(){
        System.out.println("Lazy: initializing Order");
    }
}
```

## Lifecycle of Bean (PostConstruct and PreDestroy Annotation)
The lifecycle of a bean refers to its creation, initialization, and destruction. Spring provides two annotations, `@PostConstruct` and `@PreDestroy`, to define methods that should be executed after bean creation and before bean destruction, respectively.


1. Application Start: When you start an application, Springboot internally invoke the IOC. (ApplicationContext provides the implementation of IOC Container)
2. IOC Container Started: IOC will scan for the Bean through `@Bean` and `@Configuration`.
3. Construct Bean: Initialize the beans either eagerly initialized or lazy initialized. `@Autowired`, first look for a bean of the required type. If Bean found, it will inject in different ways i.e Constructor injection, Setter injection, or Field Injection. If Bean not found, Spring will create one and then inject it.
4. Inject Dependency into Constructed Bean: If one `@Bean`, say Bean1, has dependency on another `@Bean`, say Bean2. So Springboot will create a Bean2 object and inject in Bean1.
5. PostConstruct: After the bean is constructed, Spring will invoke the `@PostConstruct`.
6. Bean Usage: After the bean is constructed and dependencies are injected, the bean is ready to use.
7. PreDestroy: When the application is shutting down, Spring will invoke the `@PreDestroy
8. Bean Destruction: After the `@PreDestroy` is invoked, the bean is destroyed.
9. Application Stop: After the bean is destroyed, the application is stopped.

![Bean Lifecycle Diagram](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/BeanLifecycle.png)

Example code snippet:
```java
@Component
public class User {

    @Autowired
    Order order;    // Step 4

    @PostConstruct  // Step 5
    public void init() {
        System.out.println("Bean has been constructed and dependencies have been injected");
    }
    
    @PreDestroy     // Step 7
    public void cleanup() {
        // Before it is actually being destroyed, we can write logic to release all the resources here
        // like we can release DB connection
        System.out.println("Bean is being destroyed");
    }

    public User() {
        System.out.println("Constructing User");    // Step 3
    }
}
```

```java
@Component
@Lazy
public class Order {
    // Bean implementation
    public Order(){
        System.out.println("Lazy: initializing Order");
    }
}
```

```java
@SpringBootApplication
public class SpringBootApplication {

    public static void main(String[] args){
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootApplication.class, args);
        context.close();    // By default it will close, showing for learning purpose.
    }
}
```

Please note that the above code snippets are simplified examples to illustrate the concepts. In a real-world application, you would typically have more complex implementations.
