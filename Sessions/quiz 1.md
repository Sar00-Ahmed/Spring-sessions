**1. What is the primary characteristic of a web application?**  
a) It requires installation from an app store.  
b) It runs on a user's local device.  
c) It runs on a remote server and is accessed via a web browser.  
d) It is always faster than a traditional desktop application.

**2. Which component in the architecture is responsible for processing business logic and generating dynamic responses?**  
A) Browser  
B) Web Server  
C) Application Server  
D) Internet

**3. A WAR (Web Application Archive) file is specifically designed to be deployed on what?** 
A) A user's mobile device.  
B) A version control system like Git.  
C) An application server.  
D) A standalone Java runtime environment (JRE).

**4. According to the notes, what is the sequence of a request's path to the Application Server?**  
A) Browser -> Internet -> Application Server  
B) Browser -> Web Server -> Application Server  
C) Browser -> Internet -> Web Server -> Application Server  
D) Browser -> Application Server -> Web Server

**5. Based on the architecture described, if a request is for a simple CSS file, which component is most likely to handle it directly without forwarding?**  
A) The Application Server  
B) The Database  
C) The Web Server  
D) The Browser's cache

**6. What does it mean that HTTP is "stateless"?**  
A) The server maintains a continuous connection with the client.  
B) The server retains information about all past client interactions.  
C) Each request is treated independently, with no retained memory of past interactions.  
D) The protocol does not use status codes.

**7. Which part of an HTTP request specifies the action the client wants to perform (e.g., GET, POST)?**  
A) Request Headers  
B) Request Body  
C) Query Parameters  
D) HTTP Method in the Start Line

**8. In the URI `/api/products?category=books&sort=price`, what are `category` and `sort`?**  
A) Request Headers  
B) Path Variables  
C) Query Parameters  
D) Body Parameters

**10. The Request Body in an HTTP request is typically used with which methods?**  
A) GET and DELETE  
B) Only GET  
C) POST and PUT  
D) Only HEAD

**11. What is the central servlet in Spring MVC that orchestrates the request lifecycle?**  
A) `HandlerInterceptor`  
B) `WebMvcConfigurer`  
C) `@RestController`  
D) `DispatcherServlet`

**1. What is the primary role of a Controller in a Spring application?**  
A) To execute the core business logic of the application.  
B) To act as the entry point for handling incoming HTTP requests and coordinating responses.  
C) To manage database connections and transactions.  
D) To serve static content like HTML and CSS files.

**2. Which annotation is a combination of `@Controller` and `@ResponseBody`, making it ideal for APIs that return data directly?**  
A) `@Service`  
B) `@Component`  
C) `@RestController`  
D) `@RequestMapping`

**4. Which annotation is used to extract a value from the URI path, such as the `123` in `/users/123`?**  
A) `@RequestParam`  
B) `@RequestBody`  
C) `@PathVariable`  
D) `@RequestHeader`

**6. The `@RequestBody` annotation is used to:**  
A) Read values from the HTTP request headers.  
B) Deserialize the body of the HTTP request into a Java object.  
C) Extract data from the URI path.  
D) Access query parameters from the URL.

**7. What is a primary security benefit of using a DTO (Data Transfer Object)?**  
A) It encrypts all data before sending it over the network.  
B) It prevents SQL injection attacks automatically.  
C) It allows you to expose only specific, safe fields from your domain model.  
D) It validates user credentials against a database.

**9. Which validation annotation would you use to ensure a string field is not null and contains at least one non-whitespace character?**  
A) `@NotNull`  
B) `@NotEmpty`  
C) `@NotBlank`  
D) `@Size`

**10. To trigger validation on a DTO object when it's received as a `@RequestBody` in a controller method, which annotation must you add to the parameter?**  
A) `@Validated`  
B) `@Valid`  
C) `@NotNull`  
D) `@Constraint`

**11. The `ResponseEntity` class in Spring is used to represent:**  
A) Only the body of an HTTP response.  
B) The entire HTTP response, including status code, headers, and body.  
C) Just the headers of an HTTP request.  
D) A template for generating HTML views.

**1. What is the core idea behind the Inversion of Control (IoC) principle?**  
A) To give a module complete control over how its dependencies are created.  
B) To invert the flow of control to achieve decoupling between modules.  
C) To make all class dependencies final and immutable.  
D) To increase the execution speed of an application.

**3. What is Dependency Injection (DI)?**  
A) A design principle for creating tight coupling between classes.  
B) The process of manually instantiating dependencies within a class.  
C) The practical implementation of IoC, supplying dependencies at runtime.  
D) A method for hiding sensitive configuration data.

**4. Which type of dependency injection is considered the most recommended by the Spring team due to its support for immutability and testability?**  
A) Field Injection  
B) Setter Injection  
C) Constructor Injection  
D) Interface Injection

**6. When would Setter Injection be a preferable choice?**  
A) For mandatory dependencies that the class cannot function without.  
B) When you need the ability to change the dependency at runtime.  
C) When you want to make the dependency `final`.  
D) For all dependencies in a production application.

**7. In Spring, what is the primary role of an IoC Container?**  
A) To compile Java source code into bytecode.  
B) To manage the HTTP request/response cycle.  
C) To manage the lifecycle of objects (beans) and implement Dependency Injection.  
D) To provide a user interface for application configuration.

**9. How does the `BeanFactory` typically initialize beans?**  
A) It uses eager initialization, creating all beans at startup.  
B) It uses lazy initialization, creating beans only when requested.  
C) It initializes beans when the application is shut down.  
D) It never initializes beans automatically.

**11. What problem does the `@Qualifier` annotation solve?**  
A) It marks a bean as the default choice for injection.  
B) It specifies the exact bean name to inject when multiple implementations of an interface exist.  
C) It defines the scope of a bean (e.g., singleton, prototype).  
D) It initializes a bean after its dependencies are set.

**18. What is a significant advantage of making dependencies mandatory via Constructor Injection?**  
A) It makes the class more flexible for future changes.  
B) It ensures the object is never in an invalid state due to missing dependencies.  
C) It allows dependencies to be changed after object creation.  
D) It reduces the amount of configuration code needed.

**1. What is the primary purpose of a class annotated with `@Configuration`?**  
A) To define REST API endpoints.  
B) To contain `@Bean` definition methods for the Spring container to process.  
C) To map HTTP request parameters to method arguments.  
D) To validate the structure of incoming JSON payloads.

**2. Which annotation is used on a method inside a `@Configuration` class to indicate that the method's return value should be a Spring bean?**  
A) `@Component`  
B) `@Service`  
C) `@Bean`  
D) `@Autowired`

**10. Which of the following best describes the conceptual difference between a global variable and an externalized property?**  
A) Global variables are for configuration, while properties are for application logic.  
B) Global variables create tight coupling and are unpredictable, while properties promote loose coupling and are read-only at runtime.  
C) Global variables are stored in files, while properties are stored in memory.  
D) There is no significant difference; they are interchangeable terms.

---
# Advanced
**7. Which of the following is NOT listed as a function of Nginx?**  
A) Load Balancer  
B) HTTP Cache  
C) Web Server  
D) Executing Java business logic

**10. In which method of a `HandlerInterceptor` would you place code to log the start time of a request?**  
A) `postHandle()`  
B) `afterCompletion()`  
C) `preHandle()`  
D) `addInterceptors()`

**11. If a `preHandle()` method in an interceptor returns `false`, what happens?**  
A) The request proceeds to the controller but the response is cancelled.  
B) The request processing is stopped, and the controller is never executed.  
C) The `afterCompletion()` method is called immediately.  
D) An automatic `500` error is sent to the client.

**13. Which Spring interface must you implement to register a custom interceptor?**  
A) `HandlerInterceptor`  
B) `DispatcherServlet`  
C) `WebMvcConfigurer`  
D) `@Configuration`

**15. Which annotation is used on the interceptor class so Spring can manage it as a bean?**  
A) `@Interceptor`  
B) `@Service`  
C) `@Component`  
D) `@Bean`

**13. Which annotation would you use to validate that a number is at least 18?**  
A) `@Size(min=18)`  
B) `@Min(18)`  
C) `@NotNull`  
D) `@Positive`

**4. If you annotate a bean with `@Scope(value = WebApplicationContext.SCOPE_PROTOTYPE)`, what happens?**  
A) A single instance is created and shared for the entire application.  
B) A new instance is created every time the bean is requested.  
C) A new instance is created for each HTTP request.  
D) A new instance is created for each HTTP session.

**5. Which scope would be most appropriate for a shopping cart service in a web application?**  
A) Singleton  
B) Prototype  
C) Request  
D) Session

**6. The `@ConfigurationProperties` annotation is primarily used for what purpose?**  
A) Injecting a single property value into a field.  
B) Binding multiple related properties from a configuration file to a type-safe Java object.  
C) Defining the configuration of a Spring Security filter chain.  
D) Activating specific Spring profiles

- - -
**When would Setter Injection be a preferable choice?** - needs revision