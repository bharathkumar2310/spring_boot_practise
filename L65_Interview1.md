✅ 1. What happens when a request hits a Spring Boot application?

Flow (correct and complete):
        
        Client sends HTTP request
        Request reaches Embedded server (Tomcat/Jetty/Undertow)
        Goes to DispatcherServlet (Front Controller)
        DispatcherServlet consults HandlerMapping
        Finds the appropriate Controller method
        Uses HandlerAdapter to invoke it
        Controller processes request (Service → DAO → DB)
        Returns response (Object / ResponseEntity)
        HttpMessageConverters convert Java → JSON/XML
        Response sent back to client

👉 Interview line:

    “Spring Boot uses a front-controller pattern where DispatcherServlet coordinates request handling using HandlerMapping, HandlerAdapter, and MessageConverters.”

    Read SpringMVC.md

✅ 2. What is DispatcherServlet?

Definition:

    Core component of Spring MVC that acts as the front controller.

Responsibilities:
    
    Receives all incoming requests
    Delegates to HandlerMapping
    Invokes controller via HandlerAdapter
    Handles exceptions
    Resolves views / responses

👉 Interview line:

    “All HTTP requests in Spring MVC pass through DispatcherServlet, which orchestrates the entire request lifecycle.”

✅ 3. What is HandlerMapping?

Definition:

    Maps incoming request URLs to the correct controller method.

How it works:
    
    During startup, Spring scans @RequestMapping, @GetMapping, etc.
    Builds a mapping registry

👉 Example:

@GetMapping("/users")

👉 Interview line:

    “HandlerMapping resolves which controller method should handle a given request URL.”

✅ 4. What is HandlerAdapter?

Definition:

    Executes the handler (controller method).

Why needed?

    Supports different handler types (annotation-based, legacy controllers)

👉 Interview line:

    “HandlerAdapter acts as a bridge between DispatcherServlet and controller execution.”

Old Controller uses something like this
        
        public class MyController implements Controller {
        public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse res) {
        // logic
        }
        }

✅ 5. What is @RestController?

Combination of:

    @Controller + @ResponseBody

👉 Meaning:
    
    Returns JSON/XML directly
    No view rendering

@ResponseBody tells Spring:
    
    “Don’t render a view (like JSP/HTML).
    Take the return value of this method and write it directly into the HTTP response body.”
    “@ResponseBody tells Spring to serialize the return value of a controller method directly into the HTTP response body using HttpMessageConverters, instead of resolving it as a view.”
👉 Interview line:
    
    “@RestController is used to build REST APIs where responses are automatically serialized.”

✅ 6. @Controller vs @RestController
    
    Feature	@Controller	@RestController
    Output	View (JSP/HTML)	JSON/XML

    Use	MVC UI	REST APIs

👉 Interview tip: Mention @ResponseBody difference

✅ 7. What is @RequestMapping?
    
    Definition:
    Maps HTTP requests to handler methods.
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    public String getUsers() {
    return "users";
    }
    
    👉 Variants:
    
    @GetMapping
    @PostMapping
    @PutMapping
    @DeleteMapping
    @GetMapping, @PostMapping, @putMapping,... is a composed (meta) annotation built on top of @RequestMapping.
    
    👉 Interview line:
    “@RequestMapping defines URL patterns and HTTP methods for request handling.”

✅ 8. What is @PathVariable?

    Extracts values from URL path.

    @GetMapping("/user/{id}")
    
    👉 /user/10 → id = 10

✅ 9. What is @RequestParam?
    
    Extracts query parameters.
    
    /user?id=10

👉 Interview difference:
    
    PathVariable → part of URL
    RequestParam → query string

✅ 10. @RequestBody vs @ResponseBody
    
    Annotation	    Purpose
    @RequestBody	JSON → Java object
    @ResponseBody	Java → JSON

👉 Uses Jackson internally

✅ 11. What is HttpMessageConverter?

Definition:

    Converts request/response body between Java objects and formats (JSON/XML).

👉 Default: Uses Jackson

👉 Interview line:

        “HttpMessageConverters handle serialization and deserialization automatically.”

✅ 12. What is Jackson?

Definition:

    A Java library for JSON processing.

👉 Uses:
    
    Serialization (Java → JSON)
    Deserialization (JSON → Java)

✅ 13. What is @Valid?
    
    Triggers validation on request body.
    public ResponseEntity save(@Valid @RequestBody User user)

     READ validation.md

✅ 14. What is Bean Validation?

        Standard validation API (JSR-380)

👉 Annotations:
    
    @NotNull
    @Size
    @Email

👉 Implementation:

        Hibernate Validato
r
✅ 15. What happens if validation fails?
    
    Spring throws:MethodArgumentNotValidException
    👉 Returns 400 Bad Request by default

✅ 16. How to handle exceptions in springBoot?

    
    In springBoot exceptions can be handled by @Exception Handler and @controller Advice
    @ExceptionHandler handles controller level exceptions
    @ControllerAvice handles global level exceptions

 @RestControllerAdvice (Important 🔥)

        👉 Same as @ControllerAdvice + @ResponseBody
        
        @RestControllerAdvice
        
        👉 Used in REST APIs (very common)
        👉 Centralized exception handling

    Read SpringBoot_Exception_Handling.md

✅ 17. What is @ExceptionHandler?
    
    Handles specific exceptions:
    @ExceptionHandler(Exception.class)
    


✅ 18. @ControllerAdvice vs @RestControllerAdvice
    
    Feature	ControllerAdvice	RestControllerAdvice
    Output	View	             JSON
    
    👉 RestControllerAdvice = ControllerAdvice + ResponseBody


✅ 19. What is ResponseEntity?

Wraps HTTP response:
    
    Body
    Status
    Headers
    return ResponseEntity.ok(data);
        
        ResponseEntity is a Spring class used to fully control the HTTP response you send back from a controller.
        Instead of just returning data, it lets you control:
            
            ✅ Body
            ✅ Status code
            ✅ Headers


✅ 20. HTTP Status Code handling

            return ResponseEntity.status(404).body("Not found");
        👉 Gives full control over response

✅ 21. What is Interceptor?

    Runs before/after controller execution.

👉 Use cases:
    
    Logging
    Authentication
    Performance tracking

Read Interceptor.md 

✅ 22. Filter vs Interceptor
What is a Filter?
        
        A Filter is part of the Java Servlet API.
        
        It works at the servlet/container level and intercepts requests before they reach the DispatcherServlet.
        
        Used for:
        
        Authentication
        Logging
        CORS
        Compression
        Encoding
        Request/Response modification

What is an Interceptor?
        
        An Interceptor is part of Spring MVC.
        
        It works at the Spring framework level and intercepts requests after DispatcherServlet but before Controller.
        
        Used for:
        
        Role checks
        JWT validation
        Logging
        Performance monitoring
        Controller-specific preprocessing

| Feature                       | Filter                     | Interceptor               |
| ----------------------------- | -------------------------- | ------------------------- |
| Part of                       | Servlet API                | Spring MVC                |
| Level                         | Servlet Container          | Spring Framework          |
| Runs Before                   | DispatcherServlet          | Controller                |
| Can access Controller?        | No                         | Yes                       |
| Used For                      | Generic request processing | Business logic processing |
| Request/Response modification | Yes                        | Limited                   |
| Framework dependent?          | No                         | Yes                       |


✅ 23. What is CORS?
        
        CORS stands for Cross-Origin Resource Sharing.
        It is a browser security mechanism that controls whether a frontend from one origin can access resources from another origin.
        
        An origin consists of:
        
        Protocol
        Domain
        Port
        
        Example:
        
        Frontend: http://localhost:3000
        Backend:  http://localhost:8080
        
        These are different origins because ports are different.
        By default, browsers block such requests due to the Same-Origin Policy.
        CORS allows the backend to explicitly permit cross-origin requests.

✅ 24. How to enable CORS?
@CrossOrigin

```java
@RestController
@CrossOrigin(origins = "http://localhost:3000")
public class UserController {

    @GetMapping("/users")
    public String getUsers() {
        return "Users Data";
    }
}
```

. Global CORS Configuration

Used when entire application should allow CORS.

```java

@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("*");
    }
}

```

With Spring Security we can also enable it in config


✅ 25. What is @ResponseStatus?

Sets HTTP status manually:

    @ResponseStatus is a Spring annotation used to manually set the HTTP status code returned to the client.
    By default, successful requests return:200 OK
    But sometimes we need custom status codes like:
            
            201 CREATED
            404 NOT FOUND
            400 BAD REQUEST

    That is where @ResponseStatus is used.

```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping("/users")
public String saveUser() {

    return "User Created";
}
```
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {

}
```
✅ 26. What is Content Negotiation?
    
    Content Negotiation is the process by which the server decides in which format the response should be sent to the client.
    The client tells the server what response format it can accept using the:
    
    Accept
    header.
    
    Server then returns data in the appropriate format like:
            
            JSON
            XML
            HTML


Example

Client request:

    Accept: application/json

Server returns:
    
    {
    "id": 1,
    "name": "John"
    }

If client sends:

    Accept: application/xml

Server may return:
    
    <User>
       <id>1</id>
       <name>John</name>
    </User>
In Spring Boot
Spring Boot automatically performs content negotiation using:
    
    HttpMessageConverters
    Accept header

✅ 27. What is @RequestHeader?
    
    @RequestHeader is a Spring annotation used to read HTTP request headers from the incoming client request.
    It is commonly used for:
        
        JWT tokens
        Authorization headers
        Content type
        Custom headers

```java

@GetMapping("/users")
public String getUsers(
    @RequestHeader("Authorization") String token) {

    return token;
}
```

✅ 28. What is @CookieValue?
    
    @CookieValue is used to read cookie values from the incoming HTTP request.
    Cookies are small pieces of data stored in the browser and automatically sent with requests.

Example

    Browser sends:Cookie: token=abc123

Controller:
    
    @GetMapping("/home")
    public String home(
    @CookieValue("token") String token) {
    
        return token;
    }


Step 1: Client sends request
    POST /login

Step 2: Server validates login
    Server creates session/token.
    Then sends response:
        
        HTTP/1.1 200 OK
        Set-Cookie: SESSIONID=abc123

Step 3: Browser stores cookie
    
    Browser automatically stores:
    SESSIONID=abc123
    for that domain.

Step 4: Browser automatically sends cookie later
    
    Future requests to same domain:
    GET /profile
    Cookie: SESSIONID=abc123
    Server recognizes user session.


✅ 29. What is Pagination?

Handled using:

Pageable pageable = PageRequest.of(0, 10);

👉 Returns:

Page content
Total pages
Total elements


✅ 30. What is HATEOAS?

    👉 HATEOAS = Hypermedia As The Engine Of Application State

It’s a principle of REST where:
    
    API responses include links that tell the client what it can do next
    👉 API includes links:

🔹 Without HATEOAS (Normal REST)
    
    {
    "id": 1,
    "name": "Bharath"
    }

👉 Client has to hardcode URLs like:
    
    /users/1
    /users/1/orders

🔹 With HATEOAS
        
        {
        "id": 1,
        "name": "Bharath",
        "_links": {
        "self": { "href": "/users/1" },
        "orders": { "href": "/users/1/orders" },
        "update": { "href": "/users/1" }
        }
        }

👉 Now:
    
    API itself tells:
    where to go next
    what actions are possible
🔹 Why HATEOAS?
        
        ✅ 1. No hardcoding URLs
                Client doesn’t need to know endpoints
        
        ✅ 2. API becomes self-discoverable
                Like browsing a website via links
        
        ✅ 3. Loose coupling
                Backend changes → client still works

👉 Makes APIs self-discoverable