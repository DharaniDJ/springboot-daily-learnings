## Introduction to Spring Boot
- What problem it solves:
    - Spring Boot simplifies the process of creating and deploying Java applications by providing a convention-over-configuration approach. It eliminates the need for complex XML configurations and boilerplate code, allowing developers to focus on writing business logic.

- Its features and advantages:
    - Spring Boot offers a wide range of features and advantages, including:
        - Embedded server: Spring Boot comes with an embedded server (Tomcat, Jetty, or Undertow) that allows you to run your application as a standalone JAR file.
        - Auto-configuration: Spring Boot automatically configures various components based on the dependencies present in your project, reducing the need for manual configuration.
        - Starter dependencies: Spring Boot provides a set of starter dependencies that include all the necessary libraries for common use cases, making it easy to get started with Spring projects.
        - Actuator: Spring Boot Actuator provides production-ready features like health checks, metrics, and monitoring out of the box.
        - Devtools: Spring Boot Devtools provides automatic restarts, live reload, and other development-time features to enhance productivity.
        - Spring Boot CLI: Spring Boot Command Line Interface allows you to quickly prototype and develop Spring Boot applications using a command-line interface.

- Servlets:
    - Servlets are Java classes that handle HTTP requests and generate HTTP responses. They provide a foundation for building web applications in Java. Servlets are part of the Java Servlet API, which is a standard for building web applications on the Java platform.
    - Servlets Container are the ones which manages the servlets.
    - When an input request comes to a web application that uses servlets, the servlet container receives the request and determines which servlet should handle it based on the URL mapping defined in the web application's configuration.
        - The servlet container creates an instance of the servlet class that is responsible for handling the request.
        - The container calls the init() method of the servlet to initialize it. This method is called only once during the lifecycle of the servlet.
        - The container calls the service() method of the servlet, passing in the request and response objects. The service() method is responsible for processing the request and generating the response.
        - The service() method typically delegates the actual processing of the request to other methods such as doGet(), doPost(), doPut(), etc., based on the HTTP method of the request.
        - The servlet retrieves any necessary data from the request, performs the required processing, and generates the response.
        - The servlet sets the appropriate headers and content in the response object.
        - The container sends the response back to the client that made the request.
        - It's important to note that servlets are multithreaded, meaning that multiple requests can be processed concurrently by different threads. The servlet container manages the lifecycle and threading of servlet instances, ensuring that each request is handled by a separate thread.

    - Servlets provide a powerful and flexible mechanism for handling HTTP requests and generating responses. They can be used to build various types of web applications, from simple static content delivery to complex dynamic web applications.


- Spring MVC and its advantages over servlets:
    - Spring MVC is a web framework built on top of the Servlet API. It provides a higher level of abstraction and simplifies the development of web applications compared to raw servlets.
    
    - When an input request comes to a web application that uses the Spring framework, the request goes through several steps in the request processing pipeline. Here's an overview of how Spring framework works when handling an input request:

        - DispatcherServlet: The entry point for handling requests in a Spring MVC application is the DispatcherServlet, also known as first controller. It acts as a front controller and receives all incoming requests. The DispatcherServlet is responsible for routing the request to the appropriate controller based on the URL mapping defined in the application's configuration.

        - Handler Mapping: The DispatcherServlet consults the configured HandlerMappings to determine which controller should handle the request. HandlerMappings are responsible for mapping the incoming request to a specific controller based on criteria such as URL patterns or request headers.

        - Controller: Once the appropriate controller is determined, the DispatcherServlet invokes the corresponding controller method to handle the request. Controllers in Spring MVC are typically annotated with @Controller and contain methods annotated with @RequestMapping to specify the URL mapping for each method.

        - Request Processing: The controller method processes the request by performing the necessary business logic. It can access the request parameters, headers, and body to extract data and perform operations. The controller method can also interact with services, repositories, or other components to retrieve or manipulate data.

        - Model and View: The controller method prepares the data to be displayed in the response by populating a model object. The model object holds the data that will be passed to the view for rendering. The controller method also returns the logical view name or a ModelAndView object that specifies the view to be rendered.

        - View Resolution: The DispatcherServlet consults the configured ViewResolvers to determine the appropriate view for rendering the response. ViewResolvers are responsible for mapping the logical view name returned by the controller to an actual view template or resource.

        - View Rendering: Once the view is resolved, the DispatcherServlet invokes the view to render the response. The view can be a JSP, Thymeleaf template, or any other view technology supported by Spring MVC. The view combines the model data with the view template to generate the final HTML or other content to be sent back to the client.

        - Response: The rendered view generates the response, including the HTML content, headers, and status code. The DispatcherServlet sends the response back to the client that made the request.

    - Some advantages of Spring MVC over servlets include:
        - Annotations-based configuration: Spring MVC uses annotations to configure web components, eliminating the need for a separate web.xml file. This makes the configuration process more streamlined and easier to understand.- Annotations-based configuration: Spring MVC uses annotations to configure web components, eliminating the need for a separate web.xml file. This makes the configuration process more streamlined and easier to understand.
        - Inversion of Control (IoC): Spring MVC uses dependency injection to manage dependencies between components, making it easier to write modular and testable code.
        - Request mapping: Spring MVC provides a flexible and powerful mechanism for mapping HTTP requests to controller methods, allowing for clean and expressive URL patterns.
        - View resolution: Spring MVC supports various view technologies (JSP, Thymeleaf, etc.) and provides a unified way to render views, making it easier to develop user interfaces.
        - Validation and data binding: Spring MVC includes built-in support for data validation and binding, reducing the amount of boilerplate code required for form handling.
    - There are many other areas where Spring framework makes developer life easy such as :
        - Integration with Unit testing framework like Junit or Mockito.
        - Integration with ORM frameworks like Hibernate, JDBC, or JPA.
        - Integration with Asynchoronous programming.
        - Similar way, it has different integration available for: Caching, Messaging, Security etc...



- Spring Boot and its advantages over Spring MVC:
    - Spring Boot builds on top of Spring MVC and provides additional features and advantages, such as:
        - Simplified configuration: Spring Boot eliminates the need for XML configurations and provides sensible defaults, reducing the amount of configuration required.
        - Embedded server: Spring Boot includes an embedded server, allowing you to run your application as a standalone JAR file without the need for external server installations. In traditional Spring MVC application, we need to build a WAR file, which is a packaged file containing your applicaiton's classes, JSP pages, configuration files, and dependencies. Then we need to deploy this WAR file to a servlet container like Tomcat.
        - Auto-configuration(Opinionated Nature): Spring Boot automatically configures various components based on the dependencies present in your project, reducing the need for manual configuration for DispatcherSevlets, AppConfig, EnableWebMVC, ComponentScan.
        - Production-ready features: Spring Boot Actuator provides production-ready features like health checks, metrics, and monitoring out of the box, making it easier to manage and monitor your application.

    - So, What is Spring boot?
        - It provides a quick way to create a production ready application.
        - It is based on spring framework.
        - It support "Convention over Configuration".
            - use default values for configuration, and if developer don't want to go with convention(the way something is done), they can override it.
        - It also help to run an application as quick as possible.

