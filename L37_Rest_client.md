REST CLIENT :

    RestClient is a modern HTTP client introduced in Spring Framework 6 as a replacement/upgrade for RestTemplate.

🔹 What is RestClient?

    RestClient is a synchronous HTTP client (just like RestTemplate) used to call REST APIs, but designed with a fluent builder-style API, similar to WebClient (without being reactive).

A Fluent API is a style of writing code where methods are chained together in a readable, flowing way, almost like a sentence.

🔹 Simple Definition

    👉 Fluent API = method chaining + readability

It lets you do:

    obj.method1()
    .method2()
    .method3();

Instead of breaking everything into separate steps.

👉 RestClient keeps things:

        Cleaner
        More readable
        More customizable
        Less boilerplate

🔹 Why was RestClient introduced?

Spring team realized:

❌ RestTemplate problems:

        Too many overloaded methods (getForObject, exchange, etc.)
        Hard to customize per request
        Boilerplate-heavy
        Not aligned with modern APIs


1. Fluent API (Not just syntax — reduces complexity)

❌ RestTemplate (mentally fragmented)

    HttpHeaders headers = new HttpHeaders();
    headers.set("Authorization", "Bearer token");
    HttpEntity<User> entity = new HttpEntity<>(user, headers);
    ResponseEntity<User> response = restTemplate.exchange(
        "/users",
        HttpMethod.POST,
        entity,
        User.class
    );

👉 Problem:

    You must create multiple objects (HttpHeaders, HttpEntity)
    Hard to read flow
    Everything is detached

✅ RestClient (single flow)

    User response = client.post()
    .uri("/users")
    .header("Authorization", "Bearer token")
    .body(user)
    .retrieve()
    .body(User.class);

👉 Advantage:

    Everything in one chain
    No extra wrapper objects
    Easy to read & debug


🔥 2. No “exchange() confusion” (huge in interviews)
❌ RestTemplate confusion

    exchange(url, method, entity, responseType)

👉 Problems:

    Order matters (easy to mess up)
    Too generic → not expressive
    Hard for beginners

✅ RestClient clarity

    client.method(HttpMethod.PUT)
    .uri("/users/1")
    .body(user)
    .retrieve()

👉 Advantage:
    
    Step-by-step intention
    Self-documenting code

🔥 3. Cleaner Error Handling (THIS is subtle but powerful)

You said earlier: “I don’t see much difference” — here’s the real difference.

❌ RestTemplate (global or try-catch)

    try {
    User user = restTemplate.getForObject("/users/1", User.class);
    } catch (HttpClientErrorException e) {
    if (e.getStatusCode() == HttpStatus.NOT_FOUND) {
    // handle
    }
    }

👉 Problems:
    
    Error handling is separate from request
    Repetitive
    Hard to customize per API call

✅ RestClient (inline, per request)

    User user = client.get()
    .uri("/users/1")
    .retrieve()
    .onStatus(status -> status.value() == 404, (req, res) -> {
    throw new RuntimeException("User not found");
    })
    .body(User.class);

👉 Advantage:
    
    Error handling is attached to the request
    Per-call customization
    No need for global handler hacks
🔥 4. Interceptors → More Flexible Pipeline

You said: “I don’t see much difference” — here’s the real one 👇

❌ RestTemplate interceptor

    restTemplate.setInterceptors(List.of((req, body, exec) -> {
    req.getHeaders().add("X-Req", "123");
    return exec.execute(req, body);
    }));

👉 Limitation:
    
    Global only
    Hard to compose multiple behaviors cleanly

✅ RestClient interceptor (builder-based, composable)
    
    RestClient client = RestClient.builder()
    .requestInterceptor((req, body, exec) -> {
    req.getHeaders().add("Auth", "token");
    return exec.execute(req, body);
    })
    .requestInterceptor((req, body, exec) -> {
    System.out.println("Logging request");
    return exec.execute(req, body);
    })
    .build();

👉 Advantage:

    Chain multiple interceptors cleanly
    Order is predictable
    Feels like a pipeline

💡 Think:
👉 RestTemplate = “list of interceptors”
👉 RestClient = “execution pipeline”

🔥 5. Better Per-Request Customization

❌ RestTemplate (hard to tweak per request)

    restTemplate.setErrorHandler(customHandler); // affects all calls
✅ RestClient (per request control)
    
    client.get()
    .uri("/users")
    .retrieve()
    .onStatus(HttpStatus::is5xxServerError, (req, res) -> {
    throw new RuntimeException("Server down");
    });

👉 Advantage:
    
    Different APIs → different handling
    No global side effects

🔥 6. Less Boilerplate (this is bigger than it looks)
❌ RestTemplate
    
    HttpHeaders
    HttpEntity
    ResponseEntity
    exchange()

✅ RestClient

    👉 Direct flow — no wrappers


![img.png](Images/RC1.png)

![img_1.png](Images/RC2.png)

![img_2.png](Images/RC3.png)

![img_3.png](Images/RC4.png)

![img_4.png](Images/RC5.png)

![img_5.png](Images/RC6.png)

![img_6.png](Images/RC7.png)

![img_7.png](Images/RC8.png)

![img_8.png](Images/RC9.png)

![img_9.png](Images/RC10.png)

![img_10.png](Images/RC11.png)

![img_11.png](Images/RC12.png)

![img_12.png](Images/RC13.png)

![img_13.png](Images/RC14.png)

![img_14.png](Images/RC15.png)

![img_15.png](Images/RC16.png)

![img_16.png](Images/RC17.png)

![img_17.png](Images/RC18.png)

![img_18.png](Images/RC19.png)

![img_19.png](Images/RC20.png)

![img_20.png](Images/RC21.png)

![img_21.png](Images/RC22.png)

3. When NOT to use RestClient

This is where many fail interviews.

👉 You should say:

High concurrency systems → use WebClient
Streaming APIs → WebClient
Reactive pipelines → WebClient


--------------------------------------------------------------------------------------------------------------------------

We’ll use the same example:
    
    User user = client.get()
    .accept(MediaType.APPLICATION_JSON)
    .uri("/users/1")
    .retrieve()
    .body(User.class);

🔥 1. What object is actually created?

Internally, something like this (simplified):

So here DefaultRequest class means DefualtRequestBodyUriSpec which is the internal implementation of RequestHeadersUriSpec interface

DefaultRequest obj = new DefaultRequest();

RequestHeadersUriSpec ref = obj;

Here default request object is created and then it is referred by the interface reference variable. So when we call method on the interface reference variable it will modify the same default request object and return the same object but with different interface reference type. So here only one object is created and modified throughout the flow.
and since we return the interface only the flow is controlled and we can’t call methods which are not in that interface. So it forces us to move forward in the flow and prevents us from going back and calling methods which are not in that interface.


class DefaultRequest {

    String method;
    String uri;
    Map<String, String> headers = new HashMap<>();
    Object body;
}

👉 Only ONE object like this is created.

🔥 2. Step-by-Step Visual Evolution
🔹 Step 1: 

    client.get()
    var req = client.get();
🧠 Internal object:

    DefaultRequest {
        method = "GET"
        uri = null
        headers = {}
        body = null
    }
🎯 Interface view:

    RequestHeadersUriSpec

👉 You can call:

    uri()
    accept()
    header()

🔹 Step 2: .accept(JSON)

    req = req.accept("application/json");
🧠 Object becomes:

    DefaultRequest {
        method = "GET"
        uri = null
        headers = {
        "Accept": "application/json"
    }
    body = null
    }

👉 SAME object — just modified

👉 Interface still:

    RequestHeadersUriSpec

🔹 Step 3: .uri("/users/1")

    req = req.uri("/users/1");
    🧠 Object now:
    DefaultRequest {
        method = "GET"
        uri = "/users/1"
    headers = {
    "Accept": "application/json"
    }
    body = null
    }
🎯 Interface changes:

    RequestHeadersSpec

👉 Now you can’t call uri() again easily
👉 Flow moves forward

🔹 Step 4: .retrieve()

    var response = req.retrieve();
🧠 What happens:

        Request is prepared
        HTTP call is triggered
        📡 Actual HTTP request sent:
        GET /users/1 HTTP/1.1
        Accept: application/json
🎯 Interface:

    ResponseSpec
🔹 Step 5: .body(User.class)

    User user = response.body(User.class);
🧠 What happens:

    Response received (JSON)
    Converted into:
    User {
    id = 1,
    name = "Bharath"
    }



| Method               | Returns Interface       | Meaning                       |
| -------------------- | ----------------------- | ----------------------------- |
| `get()`              | `RequestHeadersUriSpec` | GET → no body allowed         |
| `delete()`           | `RequestHeadersUriSpec` | DELETE → usually no body      |
| `head()`             | `RequestHeadersUriSpec` | HEAD → headers only           |
| `options()`          | `RequestHeadersUriSpec` | OPTIONS request               |
| `post()`             | `RequestBodyUriSpec`    | POST → body allowed           |
| `put()`              | `RequestBodyUriSpec`    | PUT → body allowed            |
| `patch()`            | `RequestBodyUriSpec`    | PATCH → body allowed          |
| `method(HttpMethod)` | depends                 | Dynamic (body allowed or not) |

| Method         | Returns                 |
| -------------- | ----------------------- |
| `uri(...)`     | `RequestHeadersSpec`    |
| `header(...)`  | `RequestHeadersUriSpec` |
| `headers(...)` | `RequestHeadersUriSpec` |
| `accept(...)`  | `RequestHeadersUriSpec` |
| `retrieve()`   | `ResponseSpec`          |


🔥 3. Body-Capable Stage (POST/PUT)

| Method        | Returns              |
| ------------- | -------------------- |
| `uri(...)`    | `RequestBodySpec`    |
| `header(...)` | `RequestBodyUriSpec` |
| `accept(...)` | `RequestBodyUriSpec` |


| Method       | Returns              |
| ------------ | -------------------- |
| `body(...)`  | `RequestHeadersSpec` |
| `retrieve()` | `ResponseSpec`       |


GET:

    RequestHeadersUriSpec
    ↓ uri()
    RequestHeadersSpec
    ↓ retrieve()
    ResponseSpec
    ↓ body()


POST:
    
    RequestBodyUriSpec
    ↓ uri()
    RequestBodySpec
    ↓ body()
    RequestHeadersSpec
    ↓ retrieve()
    ResponseSpec
    ↓ body()


-----------------------------------------------------------------------------------------------------------------------------


✅ 1. RestClient (built once)

    RestClient client = RestClient.builder()
    .baseUrl("http://user-service")
    .build();

👉 This is:

    Shared
    Reusable
    Thread-safe
    Acts like a factory

❌ 2. Request objects (created per call)

    client.get().uri("/users/1")...
    client.get().uri("/users/2")...

👉 Each call creates:

    NEW request object
    NOT shared
    NOT reused
🔥 Visual Separation
RestClient (ONE INSTANCE)
┌─────────────────────────┐
│ baseUrl, interceptors   │
│ configuration           │
└──────────┬──────────────┘
│
┌─────────────┴─────────────┐
│                           │
Request 1                    Request 2
(GET /users/1)              (GET /users/2)

New Object A                New Object B
🔥 What build() actually does

👉 When you call .build():

It creates a configured client, which contains:

    Base URL
    Default headers
    Interceptors
    HTTP infrastructure

👉 It does NOT create request objects

🔥 What happens when you call get()
client.get()

👉 Now:
    
    A new request builder object is created
    That object implements:
    RequestHeadersUriSpec
    etc.

