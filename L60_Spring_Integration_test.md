
What is Integration Testing?

Integration testing means:

    Testing whether multiple parts of the application work together correctly
    Instead of testing one class in isolation, we test the interaction between components.

Example

Suppose you have:

    Controller → Service → Repository → Database

Integration testing checks:
    
    controller calls service properly
    service calls repository properly
    repository talks to DB properly
    complete flow works


| Feature        | Unit Test             | Integration Test             |
| -------------- | --------------------- | ---------------------------- |
| Scope          | Single class          | Multiple components          |
| Dependencies   | Mocked                | Real                         |
| Spring Context | Usually not loaded    | Loaded                       |
| Database       | No                    | Yes/Test DB                  |
| Speed          | Very fast             | Slower                       |
| Reliability    | Tests logic only      | Tests real behavior          |
| Complexity     | Simple                | More setup                   |
| Main Goal      | Verify business logic | Verify component interaction |


--------------------------------------------------------------------------------------------------------------------------------

1. @SpringBootTest

    This is the heart of integration testing.

        @SpringBootTest
        class UserIntegrationTest {
        }

What it does:

    Starts full Spring Boot application
    Loads all beans
    Creates real dependency graph

1. @SpringBootTest

Default behavior:

    webEnvironment = SpringBootTest.WebEnvironment.MOCK

So this is effectively same as:

    @SpringBootTest(webEnvironment = MOCK)

It:
    
    Loads full Spring application context
    Creates all required beans
    Does NOT start a real server like Tomcat on a port

Used for:
    
    Service layer integration tests
    Repository tests
    Controller tests using MockMvc

2. @SpringBootTest(webEnvironment = MOCK)

   @SpringBootTest(webEnvironment = WebEnvironment.MOCK)

What happens:
    
    Full application context loads
    Embedded servlet environment is mocked
    No actual HTTP server starts
    No real port used

You usually test controllers with:
    
    @Autowired
    MockMvc mockMvc;

Example:
    
    mockMvc.perform(get("/users"))
    .andExpect(status().isOk());
    
    Request never goes through real network.
    
    Think of it like:
    
    “Pretend server exists, but don't actually open one.”

3. @SpringBootTest(webEnvironment = RANDOM_PORT)
   @SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)

What happens:
    
    Full application context loads
    Real embedded server starts
    Tomcat / Jetty / Undertow
    Application runs on random port

Example:

    Port may become 54821

Useful for:
    
    Real HTTP integration testing
    Testing filters
    Security
    Serialization
    Full request flow

2. @MockBean

        @MockBean is a Spring Boot testing annotation used to replace a real bean in the Spring application context with a Mockito mock.

Example:

    @MockBean
    UserRepository userRepository;

This means:
    
    Spring will NOT use the real UserRepository
    Instead it puts a Mockito mock inside the application context
    Why @MockBean is needed

Suppose:
    
    @Service
    class UserService {
    
        @Autowired
        UserRepository repo;
    
        public User getUser(Long id) {
            return repo.findById(id).orElse(null);
        }
    }

During @SpringBootTest,
Spring creates:
    
    UserService
    UserRepository
    all beans

But maybe:
    
    you don't want DB access
    you want controlled fake data
    you want to isolate service behavior

So you replace repository bean:
    
    @SpringBootTest
    class UserServiceTest {
    
        @MockBean
        UserRepository repo;
    
        @Autowired
        UserService service;
    
        @Test
        void test() {
    
            when(repo.findById(1L))
                .thenReturn(Optional.of(new User()));
    
            User user = service.getUser(1L);
        }
    }

What actually happens

Normally Spring creates:

    UserRepository (real bean)

With @MockBean:

    UserRepository (Mockito mock bean)

Then Spring injects this mock into:

UserService

    So: service.repo becomes mocked repository.

Difference from @Mock
    
    @Mock
    @Mock
    UserRepository repo;
    
    Only Mockito knows this mock.
    
    Spring does NOT know.
    
    Used in pure unit tests.
    
    @MockBean
    @MockBean
    UserRepository repo;
    
    Spring ALSO knows this mock.

It replaces bean inside Spring context.
Used in Spring Boot integration tests.


3. MockMvc :

        MockMvc is a testing tool used to test Spring MVC controllers without starting a real server.

It simulates HTTP requests like:

    GET
    POST
    PUT
    DELETE

and lets you test:
    
    status codes
    JSON response
    headers
    validation
    controller behavior

Why MockMvc exists

    Normally browser flow is:
    Browser → HTTP → Tomcat → Controller

With MockMvc:
    
    Test → MockMvc → Controller
    No real server starts.
    No actual network call happens.
    It is fast.

    @SpringBootTest(webEnvironment = WebEnvironment.MOCK)
    @AutoConfigureMockMvc
    class UserControllerTest {
    
        @Autowired
        MockMvc mockMvc;
    }


    @Test
    void testHello() throws Exception {
    
        mockMvc.perform(get("/hello"))
               .andExpect(status().isOk())
               .andExpect(content().string("Hello"));
    }


Usually Used With

    @WebMvcTest

Controller-only testing:

    @WebMvcTest(UserController.class)
    class Test {
    }

Loads:

    controller
    MVC layer

Does NOT load:

    full application
    DB
    services (unless mocked)


@AutoConfigureMockMvc tells Spring Boot:

    “Create and configure a MockMvc object for this test.”
    Without it, MockMvc bean is usually not available in @SpringBootTest.


@WebMvcTest(UserController.class): 

    Loads only UserController




perform()

    Used to send fake HTTP request.

    mockMvc.perform(...)
    Inside perform() you use request builders like:
        
        get()
        post()
        put()
        delete()
HTTP Request Methods

Import:
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
    GET
    mockMvc.perform(get("/users"));
    POST
    mockMvc.perform(post("/users"));
    PUT
    mockMvc.perform(put("/users/1"));
    DELETE
    mockMvc.perform(delete("/users/1"));
    PATCH
    mockMvc.perform(patch("/users/1"));
Adding Request Data

Query Params
    
    mockMvc.perform(
    get("/users")
    .param("page", "1")
    );

URL becomes:
    
    /users?page=1
    Path Variable
    mockMvc.perform(get("/users/{id}", 1));
    Headers
    mockMvc.perform(
    get("/users")
    .header("Authorization", "Bearer token")
    );
    Content Type
    .contentType(MediaType.APPLICATION_JSON)
    JSON Body
    mockMvc.perform(
    post("/users")
    .contentType(MediaType.APPLICATION_JSON)
    .content("""
    {
    "name":"John"
    }
    """)
    );
    Response Verification Methods

Import:

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
Status Assertions
    
    Check 200 OK
    .andExpect(status().isOk())
    Check 201 Created
    .andExpect(status().isCreated())
    Check 404
    .andExpect(status().isNotFound())
    Check 400
    .andExpect(status().isBadRequest())
    Content Assertions
    Exact String
    .andExpect(content().string("Hello"))
    JSON
    .andExpect(content().json("""
    {"name":"John"}
    """))
    JSON Path
    
    Used to check JSON fields.
    
    .andExpect(jsonPath("$.name").value("John"))
    Array Example
    .andExpect(jsonPath("$[0].name").value("John"))
    Header Assertions
    .andExpect(header().exists("Content-Type"))
    Content Type Assertions
    .andExpect(content().contentType(MediaType.APPLICATION_JSON))
    Print Response

Very useful during debugging.
    
    .andDo(print())
    Get Actual Response
    MvcResult result =
    mockMvc.perform(get("/users"))
    .andReturn();

Then:
    
    String response =
    result.getResponse().getContentAsString();
Full Example
    
    @Test
    void testCreateUser() throws Exception {
    
        mockMvc.perform(
                post("/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                      "name":"John"
                    }
                """)
        )
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.name").value("John"))
        .andDo(print());
    }

Commonly Used Methods Summary

| Method           | Purpose                |
| ---------------- | ---------------------- |
| `perform()`      | Send request           |
| `get()`          | GET request            |
| `post()`         | POST request           |
| `put()`          | PUT request            |
| `delete()`       | DELETE request         |
| `param()`        | Query params           |
| `header()`       | Add headers            |
| `content()`      | Request body           |
| `contentType()`  | Set content type       |
| `andExpect()`    | Verify response        |
| `status()`       | Verify HTTP status     |
| `jsonPath()`     | Verify JSON field      |
| `andDo(print())` | Print request/response |
| `andReturn()`    | Get actual response    |



