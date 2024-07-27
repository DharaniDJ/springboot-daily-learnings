## Spring Boot Annotations in Controller Layer

1. **Controller and RestController Annotation:**
    - The `@Controller` annotation is used to mark a class as a Spring MVC controller. It is responsible for handling HTTP requests and returning the appropriate response.
    - The `@RestController` annotation is a specialized version of `@Controller` that is used for RESTful web services. It combines the `@Controller` and `@ResponseBody` annotations, making it easier to write RESTful APIs.

    ```java
    @Controller
    public class UserController {
        // Controller methods...
        @RequestMapping(path = "/fetchUser", method = RequestMethod.GET)
        @ResponseBody // ----> tells that the return type of the method should be considered as HTTP Response
        public User fetchUserDetails() {
            return "fetching and returning user details";
        }
    }

    @RestController
    @RequestMapping(value = "/api/")
    public class RestUserController {
        // RESTful API methods...
        @GetMapping(path = "/fetchUser")
        public User fetchUserDetails() {
            return "fetching and returning user details";
        }
    }
    ```
2. **RequestMapping Annotation:**
    - The `@RequestMapping` annotation is used to map HTTP requests to specific methods in a controller class.
    - It can be used at both class and method levels. At the class level, it defines the base URL for all methods in the controller. At the method level, it further specifies the URL path and HTTP method for the specific method.
    - The `value` attribute of `@RequestMapping` is used to specify the URL path for the method. For example, `@RequestMapping(value = "/users")` maps the method to the "/users" URL path.
    - The `path` attribute is an alias for `value` and can be used interchangeably. For example, `@RequestMapping(path = "/users")` has the same effect as `@RequestMapping(value = "/users")`.
    - The `method` attribute is used to specify the HTTP method for the method. For example, `@RequestMapping(method = RequestMethod.GET)` maps the method to handle GET requests.
    - Multiple `@RequestMapping` annotations can be used on a single method to handle multiple URL paths or HTTP methods. For example, `@RequestMapping(value = {"/users", "/customers"}, method = RequestMethod.GET)` maps the method to handle GET requests for both "/users" and "/customers" URL paths.
    - There are also specialized annotations like `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, etc., which are shortcuts for `@RequestMapping` with specific HTTP methods.
    - @Reflective({ControllerMappingReflectiveProcessor.class}) - does the mapping of all the methods to the controller

    ```java
    @Controller
    @RequestMapping("/users")
    public class UserController {

        @RequestMapping(method = RequestMethod.GET)
        public String getUsers() {
            // Logic to get all users...
        }

        @RequestMapping(value = "/{id}", method = RequestMethod.GET)
        public String getUserById(@PathVariable("id") Long id) {
            // Logic to get user by ID...
        }

        // Other methods...
    }
    ```

3. **RequestParam Annotation:**
    - The `@RequestParam` annotation is used to bind request parameters to method parameters in a controller method.
    - It can be used to extract query parameters, form data, or path variables from the request URL.
    - By default, the `@RequestParam` annotation expects the parameter to be present in the request. However, you can make it optional by specifying the `required` attribute as `false`.
    - The `defaultValue` attribute can be used to specify a default value for the parameter if it
    - http://localhost:8080/api/fetchUser?firstName=Dharani&lastName=DJ&age=26

    ```java
    @RestController
    @RequestMapping(path = "/api/")
    public class UserController {

        @GetMapping(path = "/fetchUser")
        public String fetchUsersDetails(@RequestParam(name = "firstName", defaultValue = "Dharani") String firstName,
                                        @RequestParam(name = "lastName", required = false) String lastName,
                                        @RequestParam(name = "age") int age) {
            return "fetching user details based on first name = " + firstName + " , last name = " + lastName + " and age is = " + age;
        }

        // Other methods...
    }
    ```

4. **InitBinder Annotation:**
    - The `@InitBinder` annotation is used to customize the data binding process in Spring MVC.
    - It is typically used to register a custom `DataBinder` instance, which can be used to convert request parameters to specific types or apply validation rules.
    - The annotated method should have a single parameter of type `DataBinder`.
    - The `@InitBinder` annotation can be used at the controller level or at the method level.

    ```java
    @Controller
    public class UserController {
        
        @InitBinder
        public void initBinder(DataBinder binder) {
            binder.registerCustomEditor(String.class, "firstName", new FirstNamePropertyEditor());
        }
        
        // Other methods...
    }

    public class FirstNamePropertyEditor extends PropertyEditorSupport {
        @Override
        public void setAsText(String text) throws IllegalArgumentException {
            setValue(text.trim().toUpperCase());
        }
    }
    ```

5. **PathVariable Annotation:**
    - The `@PathVariable` annotation is used to bind a path variable from the request URL to a method parameter in a controller method.
    - Path variables are used to capture dynamic parts of the URL and pass them as parameters to the controller method.
    - The annotation can be used to specify the name of the path variable and apply additional constraints, such as regular expressions, to validate the variable's value.

    ```java
    @RestController
    @RequestMapping(path = "/api/")
    public class UserController {

        @GetMapping(path = "/fetchUser/{firstName}")
        public String fetchUsersDetails(@PathVariable(value = "firstName") String firstName) {
            return "fetching user details based on first name = " + firstName;
        }

    }
    ```

6. **RequestBody Annotation:**
    - The `@RequestBody` annotation is used to bind the request body to a method parameter in a controller method.
    - It is commonly used when working with RESTful APIs to extract the request payload and convert it into an object.
    - By default, Spring uses a message converter to automatically convert the request body to the specified parameter type.
    - When data is passed through the request, it can be bound to the `@RequestBody` parameter in a controller method using either GSON or Jackson. GSON and Jackson are popular Java libraries for JSON serialization and deserialization.
    - By default, Spring uses Jackson as the JSON message converter, but you can also configure GSON if needed. To use GSON, you need to add the GSON dependency to your project and configure it as the message converter for JSON data.

    ```java
    @RestController
    @RequestMapping("/api/")
    public class UserController {

        @PostMapping(path = "/saveUser")
        public String createUser(@RequestBody User user) {
            // Logic to create a new user...
            return "User created " + user.username + " : " + user.email;
        }

        // Other methods...
    }
    ```

    ```java
    public class User {

        @JsonProperty("user_name")
        String username;
        String email;

        // Constructors, getters, and setters...

        // Other methods...
    }
    ```

    ```
    curl --location --request POST 'http://localhost:8080/api/saveUser' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "user_name": "John Doe",
        "email": "john.doe@example.com"
    }'
    ```

7. **ResponseEntity Annotation:**
    - The `ResponseEntity` class is not an annotation, but it is often used in conjunction with controller methods.
    - It represents the entire HTTP response, including the status code, headers, and body.
    - By returning a `ResponseEntity` object from a controller method, you have full control over the response, allowing you to set custom status codes, headers, and response bodies.
    - In case of `@RestController`, when we return String for the controller methods, Springboot internally creates ResponseEntity of String.

    ```java
    @RestController
    @RequestMapping("/api")
    public class UserController {

        @GetMapping("/fetchUser")
        public ResponseEntity<String> fetchUserDetails(@RequestParam(value = "firstName") String firstName) {
            String output = "fetched User details of " + firstName;
            return ResponseEntity.status(HttpStatus.OK).body(output);
        }
    }
    ```

8. **ResponseBody Annotation:**
    - The `@ResponseBody` annotation is used in Spring MVC to indicate that the return value of a controller method should be serialized and included in the HTTP response body. 

    - When a method is annotated with `@ResponseBody`, Spring will automatically convert the return value to the appropriate format, such as JSON or XML, based on the configured message converters. This allows the method to directly return the data that needs to be sent back to the client, without the need for additional view resolution or rendering.

    - By using `@ResponseBody`, you can easily build RESTful APIs that return data in a structured format, making it easier for clients to consume the response. It is commonly used in combination with the `@RestController` annotation, which is a specialized version of `@Controller` that includes `@ResponseBody` by default.

    - It is important to note that if `@ResponseBody` is not provided, Spring will consider the return value as the name of a view (.jsp file) and try to resolve and render it, especially when using the `@Controller` annotation. Therefore, when building APIs or returning raw data, it is recommended to use `@ResponseBody` to explicitly indicate that the return value should be serialized to the HTTP response body.

    ```java
    @Controller
    public class UserController {
        // Controller methods...
        @RequestMapping(path = "/fetchUser", method = RequestMethod.GET)
        @ResponseBody // ----> tells that the return type of the method should be considered as HTTP Response
        public User fetchUserDetails() {
            return "fetching and returning user details";
        }
    }
    ```

**Type Conversion**

- Type conversion from the request parameter's string representation to the specified type is handled by Spring MVC.
- When a request parameter is received as a string, Spring MVC automatically converts it to the specified type based on the method parameter's type.
- Spring MVC supports type conversion for common types such as integers, booleans, dates, and enums.
- Custom type conversion can be implemented by registering a custom converter or using the `@InitBinder` annotation.
- To specify the type of a request parameter, you can use the appropriate method parameter type in the controller method signature.
- If the request parameter cannot be converted to the specified type, Spring MVC will throw a `TypeMismatchException`.
- It is important to ensure that the request parameter's string representation is compatible with the specified type to avoid conversion errors.
- Type conversion can also be applied to path variables and request body parameters using the appropriate annotations (`@PathVariable` and `@RequestBody`).
- By default, Spring MVC uses the configured `ConversionService` to perform type conversion, but you can also customize the conversion process by implementing the `Converter` interface.
- Type conversion is a powerful feature of Spring MVC that simplifies handling request parameters of different types in controller methods.

