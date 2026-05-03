1. What happens when a request hits a Spring Boot application?

Flow:

Request → Embedded server (Tomcat)
Goes to DispatcherServlet
HandlerMapping finds controller
Controller method executes
Response returned via MessageConverters

👉 This is the core MVC flow — very important.

2. What is DispatcherServlet?

Front controller of Spring MVC.

👉 Responsibilities:

Route requests
Call correct controller
Handle response

👉 Interview line:

“All HTTP requests pass through DispatcherServlet.”

3. What is HandlerMapping?

Maps URL → Controller method

Example:

@GetMapping("/users")

👉 Internally:
Uses mappings created during startup.

4. What is HandlerAdapter?

Executes the controller method.

👉 Why needed?
Because different handler types exist.

5. What is @RestController?

Combination of:

@Controller
@ResponseBody

👉 Means:
Return JSON directly (no view rendering)

6. @Controller vs @RestController?
   Feature	@Controller	@RestController
   Output	View	JSON
   Use case	MVC UI	APIs
7. What is @RequestMapping?

Maps HTTP request to method.

👉 Variants:

@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
8. What is @PathVariable?

Extracts value from URL.

@GetMapping("/user/{id}")
9. What is @RequestParam?

Extracts query params.

/user?id=10
10. @RequestBody vs @ResponseBody?
    @RequestBody → JSON → Java object
    @ResponseBody → Java → JSON
11. What is HttpMessageConverter?

Converts:

JSON ↔ Java object

👉 Default:
Uses Jackson

12. What is Jackson?

Jackson

👉 Used for:

serialization
deserialization
13. What is @Valid?

Triggers validation on request body.

14. What is Bean Validation?

Uses:

@NotNull
@Size
@Email

👉 Backed by Hibernate Validator.

15. What happens if validation fails?

Spring throws:

MethodArgumentNotValidException
16. How to handle exceptions globally?

Using:

@ControllerAdvice
17. What is @ExceptionHandler?

Handles specific exceptions.

18. @ControllerAdvice vs @RestControllerAdvice?
    @ControllerAdvice → for MVC
    @RestControllerAdvice → for REST APIs (returns JSON)
19. What is ResponseEntity?

Wraps response:

body
status
headers
20. What is HTTP Status Code handling?

Example:

return ResponseEntity.status(404).body("Not found");
21. What is Interceptor?

Runs before/after controller.

👉 Use cases:

logging
auth
22. Filter vs Interceptor?
    Feature	Filter	Interceptor
    Level	Servlet	Spring
    Use	low-level	business logic
23. What is CORS?

Cross-Origin Resource Sharing.

👉 Needed when:
Frontend & backend on different domains.

24. How to enable CORS?
    @CrossOrigin
25. What is @ResponseStatus?

Set HTTP status manually.

26. What is Content Negotiation?

Client decides response format:

JSON
XML
27. What is @RequestHeader?

Reads HTTP headers.

28. What is @CookieValue?

Reads cookies.

29. What is Pagination in Spring Boot?

Using:

Pageable
PageRequest
30. What is HATEOAS?

Hypermedia-driven APIs.

👉 Response includes links:

{
"user": {...},
"links": [...]
}