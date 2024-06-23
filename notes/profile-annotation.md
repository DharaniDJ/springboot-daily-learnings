# @Profile Annotation | How Profiling Works in Spring Boot

## Context Setup

`@ConditionalOnProperty` - Bean is created Conditionally (mean bean can be created or not).

And based on this `@ConditionalOnProperty`, I asked below questions:

1. You have 2 applications and 1 common code base, how will you make sure that, bean is only created for one application, not for other?

    The answers were mix of `@ConditionalOnProperty` and `@Profile` annotation.

In my view:

Yes, using both we can achieve this requirement. But `@Profile` is technically intended for `environment separation` rather than application specific bean creation.

In Spring Boot, the `@Profile` annotation is used to conditionally enable or disable beans based on the active profiles. Profiles are a way to segregate parts of your application configuration and make it only available in certain environments. This is particularly useful for managing different configurations for development, testing, and production environments.

## Profiling in Spring Boot ("spring.profiles.active")

Consider a situation that you have a DB and you are running certain code in your Dev/Local machine, it would have a different username and password to connect to dev database compared to, let's say, QA/Stage machine. It would also be different in your Prod/Live machine. So, we have different types of environments when we work with the same code.

![Spring Profile Example](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/SpringProfileExample.png)


This `Username` and `Password` is just one example. There are so many other configurations, which are different for different environments, like:
- URL and Port number
- Connection timeout values
- Request Timeout values
- Throttle values
- Retry values etc.

## How to handle for different environment configurations?
we put the configurations in `application.properties` file, but how to handle for different environment configurations in a single file?

That's where `Profiling` comes into picture. Spring Boot allows you to specify which profiles are active using the `spring.profiles.active` property. This can be set in various ways, including:

- **application.properties or application.yml**: You can define the active profiles directly in your configuration files. we can have `application.properties` for each profile/environment.

  ![Spring Profile Default Code Snippet](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/SpringProfileDefaultCodeSnippet.png)
  
  ```properties
  spring.profiles.active=dev
  ```
  ```yaml
  spring:
    profiles:
      active: dev
  ```

  And during application startup, we can tell spring boot to pick specific "application properties" file, using `spring.profile.active` configuration.

  ![Spring Profile Profile Code Snippet](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/SpringProfileProfileCodeSnippet.png)

  We can pass the value to this configuration "spring.profiles.active" during application startup itself

- **Command Line Arguments**: You can pass the active profiles as command-line arguments when starting your application.
  ```sh
  java -jar myapp.jar --spring.profiles.active=dev
  ```

- **Environment Variables**: You can set the active profiles using environment variables.
  ```sh
  export SPRING_PROFILES_ACTIVE=dev
  ```

## Profile Annotation with Java Example
The `@Profile` annotation can be used on `@Configuration` classes or individual `@Bean` methods to conditionally include them based on the active profiles.

### Example 1: Using @Profile on Configuration Class
```java
@Configuration
@Profile("dev")
public class DevConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
In this example, the `DevConfig` configuration class and its beans will only be loaded when the `dev` profile is active.

### Example 2: Using @Profile on Bean Method
```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("prod")
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
Here, the `myService` bean will only be created if the `prod` profile is active.

### Example 3: Multiple Profiles
You can also specify multiple profiles for a single configuration or bean.
```java
@Configuration
@Profile({"dev", "test"})
public class DevTestConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
In this example, the `DevTestConfig` configuration class and its beans will be loaded if either the `dev` or `test` profile is active.

## Conclusion
The `@Profile` annotation in Spring Boot provides a powerful way to manage different configurations for different environments. By using profiles, you can easily switch between configurations without changing the code, making your application more flexible and easier to maintain. Whether you are working in development, testing, or production, profiles help ensure that your application behaves as expected in each environment.
