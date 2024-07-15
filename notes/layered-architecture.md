## Spring Boot Project Setup

To set up a Spring Boot project using start.spring.io, follow these steps:

1. Open your web browser and go to [start.spring.io](https://start.spring.io).
2. Fill in the necessary project details such as group, artifact, and dependencies.
3. Select the desired Spring Boot version.
4. Click on the "Generate" button to download the project as a ZIP file.
5. Extract the ZIP file to your desired location.
6. Import the project into your preferred IDE.

## JAR vs WAR: Understanding the Differences

When working with Java applications, you may come across two common packaging formats: JAR (Java Archive) and WAR (Web Application Archive). Here's a brief explanation of each:

- JAR: A JAR file is a package format used to aggregate multiple Java class files, resources, and libraries into a single file. It is primarily used for standalone Java applications or libraries. JAR files can be executed using the `java -jar` command and are commonly used for command-line tools or desktop applications.

- WAR: A WAR file is a package format used specifically for web applications. It contains all the necessary files, such as HTML, CSS, JavaScript, JSP, servlets, and libraries, required to deploy a web application on a Java application server. WAR files are typically deployed on web servers like Apache Tomcat or Jetty.

The main differences between JAR and WAR files are:

1. Purpose: JAR files are used for standalone Java applications or libraries, while WAR files are used for web applications.

2. Structure: JAR files contain Java class files, resources, and libraries, whereas WAR files contain web-related files like HTML, CSS, JavaScript, and servlets.

3. Deployment: JAR files can be executed using the `java -jar` command or included as dependencies in other projects. On the other hand, WAR files need to be deployed on a Java application server or web server.

4. Context: JAR files are not aware of web-specific features like servlets or web containers, whereas WAR files are specifically designed for web applications and can take advantage of web-related functionalities.

In summary, JAR files are used for standalone Java applications or libraries, while WAR files are used for web applications and require a Java application server or web server for deployment.

## Layered Architecture

Layered architecture is a design pattern that organizes the application into separate layers, each with its own responsibility. This separation of concerns makes the application more manageable, scalable, and maintainable. Here's a breakdown of the typical layers in a Spring Boot application, along with relevant annotations:

1. Presentation Layer (or Controller Layer): This layer handles HTTP requests and responses, dealing directly with user inputs and outputs. It acts as the application's interface to the outside world.

    - Annotations:
        - @Controller or @RestController: Used to mark a class as a request handler, defining methods to handle various HTTP requests.
        - @RequestMapping or HTTP method mappings like @GetMapping, @PostMapping, etc.: Used to map web requests to handler methods of MVC and REST controllers.

2. Service Layer: This layer contains the business logic of the application. It acts as a transaction boundary and is responsible for enforcing business rules and ensuring business processes are correctly executed.

    - Annotations:
        - @Service: Marks a class as a service component in the business layer.

3. Repository Layer (or Data Access Layer): This layer is responsible for data access and storage. It communicates with the database and performs CRUD operations.

    - Annotations:
        - @Repository: Marks a class as a Data Access Object, indicating that it's responsible for accessing data from databases, file systems, etc.

Apart from the above 3 layers, there are other packages which we create in Spring

4. DTO (Data Transfer Object): DTOs are simple objects used for transferring data between layers and processes, especially between the presentation and service layers. They help in decoupling the internal workings of an application from what is exposed externally.

    - Usage: There's no specific annotation for DTOs, but they are typically plain Java objects (POJOs) with fields and getters/setters.

5. Utility Classes: These are helper classes that provide common, reusable logic across the application, such as formatting, validation, or calculation functions.

    - Usage: Utility classes don't have specific Spring annotations. They are usually implemented as classes with static methods.

6. Entity (or Domain Model): Entities represent the business data and are typically mapped to database tables. In a Spring Boot application, entities are used by the repository layer to interact with the database.

    - Annotations:
    @Entity: Marks a class as a JPA entity.
    @Table, @Id, @GeneratedValue, etc.: Used for specifying table, primary keys, and other JPA-related configurations.

7. Configuration: Configuration classes allow for setting up application components and configurations. They can be used to define beans, database connections, and other infrastructure settings.

    - Annotations:
    @Configuration: Marks a class as a source of bean definitions.
    @Bean: Marks a method to define a bean to be managed by the Spring container.
    @EnableAutoConfiguration: Tells Spring Boot to automatically configure your application based on the dependencies you have added.

By adhering to a layered architecture, Spring Boot applications can achieve a high degree of modularity, making them easier to develop, test, and maintain. Each layer can be independently developed and tested, reducing dependencies and increasing the application's robustness and flexibility.

![Layered Architecture Diagram](../assets/layeredArchDiagram.png?raw=true)