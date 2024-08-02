# Dynamically Initialized Beans in Spring Boot

## Introduction
In Spring Boot, beans are the backbone of any application. They are objects that are instantiated, assembled, and managed by the Spring IoC container. While most beans are statically defined in configuration files or annotated classes, there are scenarios where beans need to be initialized dynamically at runtime.

## Problem Statement
In some applications, the exact nature of the beans required cannot be determined at compile-time. For instance, you might need to create beans based on user input, configuration files, or external services. This poses a challenge as the traditional static bean definitions are insufficient for such dynamic requirements.

```java
@RestController
@RequestMapping(value="/api")
public class User {
    
    @Autowired
    Order order;

    @PostMapping("/createOrder")
    public ResponseEntity<String> createOrder() {
        order.createOrder()
        return ResponseEntity.ok("");
    }
}
```
```java
public interface Order {
    
    public void createOrder() {}
}
```
```java
@Component
public class OnlineOrder implements Order {

    public OnlineOrder(){
        System.out.println("Online Order Initialized")
    }
    
    public void createOrder() {
        System.out.println("Created Online Order");
    }
}
```
```java
@Component
public class OfflineOrder implements Order {

    public OfflineOrder(){
        System.out.println("Offline Order Initialized")
    }
    
    public void createOrder() {
        System.out.println("Created Offline Order");
    }
}
```
In the above code, we have an interface `Order` and it has two children `OnlineOrder` and `OfflineOrder`. We have a `User` class which is a controller, it has a dependency on this `Order` object. 

When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `User` and it is eagarly initialized as it has `Singleton` scope. It will call its constructor, but this object is not fully constructed yet. We need to inject dependencies into the constructed bean. Now Spring doesn't know which object to assign to this `Order`, whether `OnlineOrder` or `OfflineOrder`. 

So the application failed to start - __UnsatisfiedDependencyException: Error creating bean with name `User`__

## Solution
Spring Boot provides several mechanisms to dynamically initialize beans. These include:

- Using `@Qualifier` to specify which bean should be injected.
- Leveraging the `ApplicationContext` to register beans programmatically.
- Utilizing factory beans for more complex initialization logic.
- Dynamically Initialized Beans using `@Bean`

### Using `@Qualifer` Annotation
- Using `@Qualifier` to specify which bean should be injected is particularly useful when dynamically initializing beans. This annotation allows you to differentiate between multiple beans of the same type.

```java
@RestController
@RequestMapping(value="/api")
public class User {
    
    @Qualifier("onlineOrderObject")
    @Autowired
    Order order;

    @PostMapping("/createOrder")
    public ResponseEntity<String> createOrder() {
        order.createOrder()
        return ResponseEntity.ok("");
    }
}
```
```java
public interface Order {
    
    public void createOrder() {}
}
```
```java
@Qualifier("onlineOrderObject")
@Component
public class OnlineOrder implements Order {

    public OnlineOrder(){
        System.out.println("Online Order Initialized")
    }
    
    public void createOrder() {
        System.out.println("Created Online Order");
    }
}
```
```java
@Qualifier("offlineOrderObject")
@Component
public class OfflineOrder implements Order {

    public OfflineOrder(){
        System.out.println("Offline Order Initialized")
    }
    
    public void createOrder() {
        System.out.println("Created Offline Order");
    }
}
```

- Here we have hardcoded the `order` value in `User`. If `User` want to create `offlineOrder` we can't do it because when spring creates a `User` object, it will only assign the `onlineObject`. It breaks `Dependency Injection`, which says that dynamically you can provide any value. Dependency injection is basically providing the objects that an object needs (its dependencies) instead of having it construct them itself.

Look at the below updated code for ensure Dependency Injection.

```java
@RestController
@RequestMapping(value="/api")
public class User {
    
    @Qualifier("onlineOrderObject")
    @Autowired
    Order onlineOrderObj;

    @Qualifier("offlineOrderObject")
    @Autowired
    Order offlineOrderObj;

    @PostMapping("/createOrder")
    public ResponseEntity<String> createOrder(@RequestPara boolean isOnlineOrder) {
        
        if(isOnlineOrder){
            onlineOrderObj.createOrder();
        }else{
            offlineOrderObj.createOrder();
        }

        return ResponseEntity.ok("");
    }
}
```

### Registering Beans Programmatically
You can also register beans programmatically using the `ApplicationContext`. This approach is useful when the bean definitions depend on runtime information.

```java
@Component
public class DynamicBeanRegistrar implements ApplicationRunner {

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ConfigurableApplicationContext configurableApplicationContext = 
            (ConfigurableApplicationContext) applicationContext;

        BeanDefinitionRegistry beanDefinitionRegistry = 
            (BeanDefinitionRegistry) configurableApplicationContext.getBeanFactory();

        BeanDefinitionBuilder beanDefinitionBuilder = 
            BeanDefinitionBuilder.genericBeanDefinition(MyBean.class);
        
        beanDefinitionRegistry.registerBeanDefinition("myDynamicBean", 
            beanDefinitionBuilder.getBeanDefinition());
    }
}
```

### Using Factory Beans
Factory beans provide a flexible way to create complex beans. They can encapsulate the logic required to instantiate and configure beans.

```java
public class MyBeanFactory implements FactoryBean<MyBean> {

    @Override
    public MyBean getObject() throws Exception {
        if (someCondition()) {
            return new MyBeanImpl1();
        } else {
            return new MyBeanImpl2();
        }
    }

    @Override
    public Class<?> getObjectType() {
        return MyBean.class;
    }

    private boolean someCondition() {
        // Your condition logic here
        return true;
    }
}
```

### Dynamically Initialized Beans
Dynamically initialized beans provide the flexibility needed for complex applications where the exact nature of the beans cannot be determined at compile-time. By leveraging Spring Boot's configuration capabilities, you can create and manage beans dynamically, ensuring your application remains robust and adaptable to changing requirements.

```java
@RestController
@RequestMapping(value="/api")
public class User {
    
    @Autowired
    Order order;

    @PostMapping("/createOrder")
    public ResponseEntity<String> createOrder() {
        order.createOrder()
        return ResponseEntity.ok("");
    }
}
```
```java
public interface Order {
    
    public void createOrder() {}
}
```
```java
// No @Component
public class OnlineOrder implements Order {

    public OnlineOrder(){
        System.out.println("Online Order Initialized")
    }
    
    public void createOrder() {
        System.out.println("Created Online Order");
    }
}
```
```java
// No @Component
public class OfflineOrder implements Order {

    public OfflineOrder(){
        System.out.println("Offline Order Initialized")
    }
    
    public void createOrder() {
        System.out.println("Created Offline Order");
    }
}
```
```java
@Configuration
public class AppConfig{

    @Bean
    public Order createOrderBean(@Value("${isOnlineOrder}") boolean isOnlineOrder){
        if(isOnlineOrder){
            return new OnlineOrder();
        }else{
            return new OfflineOrder();
        }
    }
}
```
```
In application.properties, we will set isOnlineOrder to false and @Value takes the value of isOnlineOrder from application.properties file

isOnlineOrder=false
```

When you start the application, IOC gets started. IOC will look the classes which needs to be managed by spring i.e `@Component`,`@RestController`. Lets say it found `User` and it is eagarly initialized as it has `Singleton` scope. It will call its constructor, but this object is not fully constructed yet. We need to inject dependencies into the constructed bean. The `OnlineOrder` and `OfflineOrder` class does not have `@Component`, so IOC will skip these files and go to `@Configuration` which is `AppConfig` to see if there is any configuration provided for `Order`. 

`@Value` is used to inject values from various sources like property file, environment variables or inline literals.

