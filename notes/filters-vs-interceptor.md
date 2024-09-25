# Filters vs Interceptors in Spring Boot

## What is a Filter?

A filter is an object that performs filtering tasks on either the request to a resource or on the response from a resource, or both. Filters are part of the Servlet API and can be used to perform tasks such as logging, authentication, and authorization before the request reaches the servlet.

## What is an Interceptor?

An interceptor is a component that allows for pre-processing and post-processing of requests in a Spring MVC application. Interceptors are part of the Spring framework and can be used to perform tasks such as logging, authentication, and authorization before the `request reaches the controller.`

## Filter and Interceptor

![filter-interceptor](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/filter-interceptor.png)

### Explanation
when the Tomcat/Servlet Container get this HTTP request, before assigning it to a particular servlet, our filter comes into the picture. we can have multiple filters i.e chain of filters and after the request goes through the filters, one of the specific servlet will get invoked. If we are running a spring boot application, the dispatcher servlet will get invoked. After the dispatcher servlet, we have interceptor. So before this dispatcher servlet handles the request to the controller, our interceptor comes into the picture.

so `filters lies before the request handed over to the servlet` and `interceptor lies before the request handed over to the controller`.

## What is a Servlet?

A servlet is a Java programming language class used to extend the capabilities of servers that host applications accessed by means of a request-response programming model. Servlets can respond to any type of request but are commonly used to extend the applications hosted by web servers.

### Example of a Servlet:
```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "MyServlet", urlPatterns = {"/myServlet"})
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("Hello from MyServlet!");
    }
}
```

## How Can We Create Multiple Servlets?

You can create multiple servlets by defining multiple servlet classes and mapping them to different URL patterns using the `@WebServlet` annotation or by configuring them in the `web.xml` file.

we can create multiple servlets like:
- Servlet 1: can be configured to handle REST APIs
- Servlet 2: can be configured to handle SOAP APIs
- Servlet 1: can be configured to handle file uploads

Similarly like this, `DispatcherServlet` is kind of servlet provided by spring, and by default its configured to handle all APIs `/*`. `DispatcherServlet` and `Interceptor` are concpets specific to `Spring boot`
### Example:
```java
@WebServlet(name = "FirstServlet", urlPatterns = {"/firstServlet"})
public class FirstServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("Hello from FirstServlet!");
    }
}

@WebServlet(name = "SecondServlet", urlPatterns = {"/secondServlet"})
public class SecondServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("Hello from SecondServlet!");
    }
}
```

## When to use FILTER vs INTERCEPTOR:

### Filter:
- Is used when we want to intercept HTTP Request and Response and add logic agnostic of the underlying servlets.
- we can have many filters and have ordering between them too.

### Interceptors:
- Is used when we want to intercept HTTP Request and Response and add logic specific to a particular servlet.
- we can have many interceptors and have ordering between them too.

## Multiple Interceptors and its Ordering:

In `Custom Interceptors.md` file., we already saw, how to add 1 interceptor.
How `preHandle`, `postHandle` and `afterCompletion` comes into the picture.

Now here, will show how to add more than 1.

Also, if `preHandle` returns false, next interceptor and controller will not get invoked itself.

![multiple-interceptors](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/multiple-interceptors.png)

Since we added `myCustomInterceptor1` first and then `myCustomInterceptor2`. `myCustomInterceptor1` will be executed first.
```
Request  -> IC1, IC2
Response -> IC2, IC1
```

![multiple-interceptors-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/multiple-interceptors-output.png)

## How to add filters

![filter1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/filter1.png)

Once the object of `MyFilter1` gets created, `init` method gets invoked only one time. In `doFilter`, it does something before it process the request to the next filter (similar to recurrsion).

![filter2](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/filter2.png)

![filter-registration](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/filter-registration.png)

Here in `AppConfig`, we are creating beans for `MyFilter1` and `MyFilter2`. In `MyFirstFilter` and `MySecondFilter`, we are registering our filters, adding all the urls and setting the ordering.

![multiple-filters-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/multiple-filters-output.png)

![filters-interceptors-output](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/filters-interceptors-output.png)