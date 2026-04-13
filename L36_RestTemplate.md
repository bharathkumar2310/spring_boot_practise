

## 🔹 What is `RestTemplate`?

`RestTemplate` is a **Spring-provided HTTP client** that makes it easy to call **RESTful APIs** from your Java code.

It acts as a **wrapper** over lower-level HTTP libraries like:

- `HttpURLConnection` (Java built-in)

- `Apache HttpClient`

- `OkHttp` (optional)


It handles:
- HTTP methods (GET, POST, PUT, DELETE, etc.)

- Serializing Java objects to JSON/XML (for request)

- Deserializing JSON/XML responses into Java objects

- Setting headers, request bodies, and parameters

> An **HTTP client** is a component (library/class/tool) that **creates and sends HTTP requests and receives HTTP responses**.

---

## 🧠 In simple terms

> It is the **thing that acts as the “caller”** in client → server communication.

---

## 🔹 Example in real life

- Browser (Chrome) → HTTP client
- Your Java code → HTTP client
- Postman → HTTP client

---

## 🔥 In Java specifically

Examples of HTTP clients:

- Java built-in:
    - `HttpURLConnection`
    - `HttpClient` (Java 11+) ✅
- Libraries:
    - Apache HttpClient
    - OkHttp
- Spring:
    - `RestTemplate`
    - `WebClient`

---

## 🧠 What does an HTTP client actually do internally?

When you write:

client.send(request);

👉 It does:

1. Build HTTP request (GET/POST, headers, body)
2. Create socket
3. Establish TCP connection (handshake)
4. Send request over socket
5. Receive response
6. Convert response into usable format



| Advantage                                | Description                                                      |
| ---------------------------------------- | ---------------------------------------------------------------- |
| ✅ **Simple and declarative**             | Very easy to use for quick API calls                             |
| ✅ **Integrated with Spring Boot**        | Automatically configured with message converters (Jackson, Gson) |
| ✅ **Auto serialization/deserialization** | Converts Java ↔ JSON automatically                               |
| ✅ **Exception handling**                 | Built-in `RestClientException` for handling errors               |
| ✅ **Customizable**                       | Supports interceptors, timeouts, error handlers                  |
| ✅ **Supports all HTTP methods**          | GET, POST, PUT, PATCH, DELETE                                    |
|                                          |                                                                  |



| Advantage                                | Description                                                      |
| ---------------------------------------- | ---------------------------------------------------------------- |
| ✅ **Simple and declarative**             | Very easy to use for quick API calls                             |
| ✅ **Integrated with Spring Boot**        | Automatically configured with message converters (Jackson, Gson) |
| ✅ **Auto serialization/deserialization** | Converts Java ↔ JSON automatically                               |
| ✅ **Exception handling**                 | Built-in `RestClientException` for handling errors               |
| ✅ **Customizable**                       | Supports interceptors, timeouts, error handlers                  |
| ✅ **Supports all HTTP methods**          | GET, POST, PUT, PATCH, DELETE                                    |
|                                          |                                                                  |


![img.png](Images/HC1.png)

![img_1.png](Images/HC2.png)

![img_2.png](Images/HC3.png)

![img_3.png](Images/HC4.png)

![img_4.png](Images/HC5.png)

![img_5.png](Images/HC6.png)

![img_6.png](Images/HC7.png)

![img_7.png](Images/HC8.png)

![img_8.png](Images/HC9.png)

![img_9.png](Images/HC10.png)

![img_10.png](Images/HC11.png)

![img_11.png](Images/HC12.png)

![img_12.png](Images/HC13.png)

![img_13.png](Images/HC14.png)




![img.png](Images/RT1.png)

![img_1.png](Images/RT2.png)

![img_2.png](Images/RT3.png)

![img_3.png](Images/RT4.png)

![img_4.png](Images/RT5.png)

![img_5.png](Images/RT6.png)

![img_6.png](Images/RT7.png)

![img_7.png](Images/RT8.png)

![img_8.png](Images/RT9.png)

![img_9.png](Images/RT10.png)

![img_10.png](Images/RT11.png)


Overview: `getForObject()` in `RestTemplate`

The `RestTemplate` class provides **four overloaded versions** of `getForObject()`.

All of them perform an **HTTP GET** request and convert the **response body** into the given `responseType`.  
The main difference lies in **how you pass URI variables** (path parameters or query params).

---

## ⚙️ 1️⃣ `getForObject(String url, Class<T> responseType, Object... uriVariables)`

**Signature:**

`public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables)`

### 📘 Explanation:

- You provide the **URL** with placeholders like `{id}`.

- You pass values for those placeholders as **varargs** (`Object...`).

- Spring replaces `{}` placeholders in order.


**Example:**

```
`String url = "https://api.example.com/users/{id}";
User user = restTemplate.getForObject(url, User.class, 101);

```

```
`String url = "https://api.example.com/{id1}/users/{id2}";
User user = restTemplate.getForObject(url, User.class, 101, 102);
```


✅ **What happens internally:**

1. Spring replaces `{id}` with `101`.

2. Sends GET request to `https://api.example.com/users/101`.

3. Converts JSON response to `User`.


**When to use:**  
When you have a few ordered URI variables.

---

## ⚙️ 2️⃣ `getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables)`

**Signature:**

`public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables)`

### 📘 Explanation:

- You use a `Map` to bind named placeholders to values.

- This makes it **clearer** when there are multiple variables.


**Example:**

```
String url = "https://api.example.com/users/{userId}/posts/{postId}";
Map<String, Object> params = new HashMap<>();
params.put("userId", 101);
params.put("postId", 55);

Post post = restTemplate.getForObject(url, Post.class, params);

```
✅ Internally:

1. `{userId}` → 101

2. `{postId}` → 55

3. URL → `https://api.example.com/users/101/posts/55`


**When to use:**  
When you have **multiple** or **named path variables** (preferred for clarity).

---

## ⚙️ 3️⃣ `getForObject(URI url, Class<T> responseType)`

**Signature:**

`public <T> T getForObject(URI url, Class<T> responseType)`

### 📘 Explanation:

- You pass a **fully constructed URI** (no placeholders).

- Spring doesn’t perform any variable replacement.

- Useful when you already have a `URI` built via `UriComponentsBuilder`.


**Example:**

```
URI uri = UriComponentsBuilder
             .fromUriString("https://api.example.com/users")
             .queryParam("page", 2)
             .queryParam("limit", 10)
             .build()
             .toUri();

UserList users = restTemplate.getForObject(uri, UserList.class);

```

✅ Internally:

- No variable expansion — it just calls the URL directly.


**When to use:**  
When you build dynamic URLs programmatically with `UriComponentsBuilder`.

---

## ⚙️ 4️⃣ (Implicit variant via generic method)

There’s technically also a **generic variant** when using parameterized types:

`<T> T getForObject(String url, Class<T> responseType)`

But note — this **does not actually exist** separately; it’s just a shorthand for:

`public <T> T getForObject(URI url, Class<T> responseType)`

So, effectively, **three main variants** exist (String+varargs, String+Map, URI).

---

## 🧩 Summary Table

|Overload|Parameters|URI Handling|Typical Usage|
|---|---|---|---|
|`getForObject(String, Class<T>, Object...)`|Positional placeholders|Order-based replacement|Simple single or few vars|
|`getForObject(String, Class<T>, Map<String, ?>)`|Named placeholders|Key-based replacement|Multiple vars (clearer)|
|`getForObject(URI, Class<T>)`|Direct URI|Already built URI|Dynamic URLs or query params|


## 🧩 What is `UriComponentsBuilder`?

`UriComponentsBuilder` is a **Spring utility class** used to **build URIs safely and dynamically** — especially when you need to add **path variables** or **query parameters** programmatically.

Instead of manually concatenating strings like:

`"https://api.example.com/users?id=" + userId + "&sort=" + sortOrder`

—you use `UriComponentsBuilder` to handle encoding, slashes, and placeholders correctly.

---

## 🧠 Why we need it

### Problem with manual string building:

- Hard to manage when URLs have **multiple parameters**.

- Easy to miss `?`, `&`, or slashes.

- No automatic **URL encoding** (e.g., spaces, special chars).

- Unreadable when dynamic.


✅ `UriComponentsBuilder` solves all that:

- Builds valid, encoded, and clean URLs.

- Handles **path variables**, **query params**, and **URI encoding** automatically.


---

## ⚙️ Common Usage Patterns

### 🔹 1️⃣ Building a simple URI

```

```

✅ Produces:

`https://api.example.com/users`

---

### 🔹 2️⃣ Adding Query Parameters

```
URI uri = UriComponentsBuilder
            .fromUriString("https://api.example.com/users")
            .queryParam("page", 2)
            .queryParam("limit", 10)
            .queryParam("sort", "name")
            .build()
            .toUri();

```

✅ Produces:

`https://api.example.com/users?page=2&limit=10&sort=name`

---

### 🔹 3️⃣ Adding Path Variables

```
URI uri = UriComponentsBuilder
            .fromUriString("https://api.example.com/users/{id}/posts/{postId}")
            .buildAndExpand(101, 55)
            .toUri();

```

✅ Produces:

`https://api.example.com/users/101/posts/55`

(Using **`buildAndExpand()`** replaces `{}` placeholders.)

---

### 🔹 4️⃣ Handling Special Characters (Auto-Encoding)

```
URI uri = UriComponentsBuilder
            .fromUriString("https://api.example.com/search")
            .queryParam("q", "java rest template")
            .build()
            .encode()
            .toUri();

```

✅ Produces:

`https://api.example.com/search?q=java%20rest%20template`

(`encode()` ensures URL-safe encoding of spaces and symbols.)

---

## 🧩 Combined with RestTemplate

Once the URI is built, you can directly pass it to `RestTemplate`:

```
URI uri = UriComponentsBuilder
            .fromUriString("https://api.example.com/users")
            .queryParam("page", 1)
            .queryParam("size", 5)
            .build()
            .toUri();

UserList users = restTemplate.getForObject(uri, UserList.class);

```


```
// URL template with placeholders
String baseUrl = "https://api.example.com/users/{userId}/posts/{postId}";

// Path variable map
Map<String, Object> pathVars = Map.of(
    "userId", 101,
    "postId", 55
);

// Build query string
URI uri = UriComponentsBuilder
            .fromUriString(baseUrl)
            .queryParam("sort", "date")
            .queryParam("limit", 10)
            .build(pathVars); // uses the map to fill placeholders

// Call RestTemplate
Post post = restTemplate.getForObject(uri, Post.class);

```


✅ Clean, type-safe, and readable.



--------------------------------------------------------------------------------------------



## What is `getForEntity()`?

`getForEntity()` is a **Spring RestTemplate method** used to send an HTTP **GET** request to a URL and receive the **full HTTP response** (not just the body).

It gives you a `ResponseEntity<T>` — which contains:

- the **body**

- the **HTTP status code**

- and the **response headers**



Has same methods as getForBody()
-

-------------------------------------------------------------------------------------------



## 🧩 What is `postForObject()`?

`postForObject()` is a **RestTemplate** method used to send an HTTP **POST** request to a URL and directly get the **response body** as a Java object.

It’s like:

> “Send data to the server (POST) and give me just the response body.”

---

## ⚙️ Method Signatures (Overloads)

There are **three overloaded versions**, similar to `getForObject()`:

### 1️⃣

`<T> T postForObject(String url, Object request, Class<T> responseType, Object... uriVariables)`

→ For **positional path variables**.

---

### 2️⃣

`<T> T postForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)`

→ For **named path variables** using a map.

---

### 3️⃣

`<T> T postForObject(URI url, Object request, Class<T> responseType)`

→ For a **prebuilt URI** (often from `UriComponentsBuilder`).

---

## 🧠 Parameters Breakdown

|Parameter|Description|
|---|---|
|`url`|Target endpoint URL|
|`request`|Object to send in POST body (can be POJO, Map, or JSON String)|
|`responseType`|Type of object expected in the response|
|`uriVariables`|Optional path variables to fill in URL placeholders|

---

## ✅ Example 1 — Simple POST Request

```
String url = "https://api.example.com/users";

User newUser = new User("John", "Doe", "john.doe@example.com");

User createdUser = restTemplate.postForObject(url, newUser, User.class);

System.out.println(createdUser);

```

✅ What happens here:

- `RestTemplate` converts `newUser` → JSON

- Sends it as a **POST body**

- Deserializes response JSON → `User.class`

- Returns **just the body**, not headers or status


---

## ✅ Example 2 — POST with path variables


```
String url = "https://api.example.com/companies/{companyId}/users";

User user = new User("John", "Doe");

User created = restTemplate.postForObject(url, user, User.class, 101);

```


or using a map:

```
Map<String, Object> vars = Map.of("companyId", 101);

User created = restTemplate.postForObject(url, user, User.class, vars);

```

---

## ✅ Example 3 — Using URI with query params

```
URI uri = UriComponentsBuilder
    .fromUriString("https://api.example.com/users")
    .queryParam("notify", true)
    .build()
    .toUri();

User user = new User("Jane", "Smith");

User created = restTemplate.postForObject(uri, user, User.class);

```

---

## ⚠️ Important: No Headers Support

`postForObject()` **cannot** send custom headers (like Authorization or Content-Type manually set).  
However, it automatically sets headers based on your body:

- If body is a POJO → `Content-Type: application/json`

- If body is a MultiValueMap → `Content-Type: application/x-www-form-urlencoded`


If you want to set headers manually (e.g., for **JWT, Basic Auth, or API keys**),  
➡️ you must use `exchange()` instead.




-------------------------------------------------------------------------------


`postForEntity()` is a **RestTemplate** method that sends an HTTP **POST** request to a URL and returns a **`ResponseEntity<T>`** — which includes:

- ✅ **Response body**

- ✅ **Response status code**

- ✅ **Response headers**


So it’s like:

> “POST this object to the server, and give me the **entire HTTP response** — not just the body.”


methods all are same as postForBody()




-------------------------------------------------------------------------------------


## 🧩 What is `put()` in RestTemplate?

`RestTemplate.put()` is used to send an HTTP **PUT** request —  
usually to **update** a resource on the server.

Unlike `postForObject()` or `postForEntity()`,  
👉 **`put()` does not return anything** — it’s a **void** method.

That’s because the HTTP PUT conventionally doesn’t require a response body (the server just updates the resource).


## 🧩 `RestTemplate.put()` Overloads (from Spring 5.x / 6.x)

There are **two main overloaded forms** of `put()` in the `RestTemplate` class:

---

### **1️⃣** `put(String url, @Nullable Object request, Object... uriVariables)`

**Signature:**

`public void put(String url, @Nullable Object request, Object... uriVariables)`

**Usage:**

- Simplest form

- You can provide path variables directly as varargs


**Example:**

`restTemplate.put(     "https://api.example.com/users/{id}",     updatedUser,     10 );`

👉 Here:

- `{id}` → replaced by `10`

- `updatedUser` → serialized to JSON and sent as request body


---

### **2️⃣** `put(String url, @Nullable Object request, Map<String, ?> uriVariables)`

**Signature:**

`public void put(String url, @Nullable Object request, Map<String, ?> uriVariables)`

**Usage:**

- Same as above, but path variables are passed as a `Map`

- Useful when you have dynamic variable names or multiple path variables


**Example:**


```
Map<String, Object> params = new HashMap<>();
params.put("userId", 10);
params.put("deptId", 5);

restTemplate.put(
    "https://api.example.com/departments/{deptId}/users/{userId}",
    updatedUser,
    params
);

```




### 🔹 3️⃣ `put(URI url, @Nullable Object request)`

**Signature:**

`public void put(URI url, @Nullable Object request)`

**Purpose:**  
Used when you already have a fully constructed `URI` (maybe built dynamically with query params, encoded safely, etc.).

---

### 🧠 Example:

```
RestTemplate restTemplate = new RestTemplate();

User updatedUser = new User("Alice", "alice@example.com");

// Build URI safely with query parameters or encoding
URI uri = UriComponentsBuilder
        .fromHttpUrl("https://api.example.com/users/42")
        .queryParam("notify", true)
        .build()
        .toUri();

restTemplate.put(uri, updatedUser);

```

✅ This version avoids Spring doing its own string expansion (`{id}`, `{name}`, etc.)  
✅ It’s safer for **complex or pre-encoded URLs**  
✅ Often used when you’re composing URIs programmatically





--------------------------------------------------------------------------------



---------------------------------------------------------------------------------

## 🧩 What is `delete()` in RestTemplate?

`RestTemplate.delete()` is used to send an HTTP **DELETE** request to remove a resource on the server.

> It’s basically: “Delete this resource from the given URL.”

---

## ⚙️ Method Overloads

Just like `getForObject()` and `put()`, there are **three overloads**:

### 1️⃣

`void delete(String url, Object... uriVariables)`

→ For positional path variables.

---

### 2️⃣

`void delete(String url, Map<String, ?> uriVariables)`

→ For named path variables.

---

### 3️⃣

`void delete(URI url)`

→ When you already have a `URI` (from `UriComponentsBuilder`).

---

## 🧠 Parameters

|Parameter|Description|
|---|---|
|`url`|The target endpoint (e.g., `"https://api.example.com/users/{id}"`)|
|`uriVariables`|Optional values to replace path placeholders|

---

## ✅ Example 1 — Simple DELETE by ID

`String url = "https://api.example.com/users/{id}"; restTemplate.delete(url, 101);`

This sends:

`DELETE https://api.example.com/users/101`

✅ Deletes user with ID 101  
✅ Returns nothing (method is `void`)

---

## ✅ Example 2 — DELETE with Named Variables

```
String url = "https://api.example.com/companies/{companyId}/users/{userId}";

Map<String, Object> vars = Map.of(
    "companyId", 10,
    "userId", 200
);

restTemplate.delete(url, vars);

```

✅ Replaces placeholders before sending request.

---

## ✅ Example 3 — Using URI with Query Parameters

```
URI uri = UriComponentsBuilder
        .fromUriString("https://api.example.com/users")
        .queryParam("inactive", true)
        .build()
        .toUri();

restTemplate.delete(uri);

```

✅ Sends DELETE to `/users?inactive=true`

---

## ⚠️ Important Limitation

Like `put()`, the `delete()` method:

- ❌ **Does not return a response**

- ❌ **Cannot include headers**

- ❌ **Cannot include a request body** (by HTTP spec, DELETE rarely has one)


If you need to send headers (like JWT tokens, API keys, etc.),  
you **must use** the `exchange()` method.



-------------------------------------------------------------------------------

## 🧩 Does `RestTemplate` have a `patch()` method?

✅ **Yes — but not always.**  
It depends on the **Spring version** you’re using.

---

### 🧱 1️⃣ In older Spring versions (before 4.3)

There was **no direct `patchForObject()` or `patchForEntity()`** method.  
You had to use:

`restTemplate.exchange(url, HttpMethod.PATCH, entity, responseType);`

That’s the only way.

---

### 🧱 2️⃣ In Spring 4.3 and later

Spring **added** built-in methods for PATCH:

```
<T> T patchForObject(String url, Object request, Class<T> responseType, Object... uriVariables)
<T> T patchForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)
<T> T patchForObject(URI url, Object request, Class<T> responseType)

```

So yes — **`patchForObject()` exists** now.

---

## 🔍 `patchForObject()` Signature

Let’s go through the overloads 👇

### 1️⃣

```
<T> T patchForObject(String url, Object request, Class<T> responseType, Object... uriVariables)

```

→ Uses positional path variables.

---

### 2️⃣

```
<T> T patchForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)

```

→ Uses named path variables.

---

### 3️⃣

`<T> T patchForObject(URI url, Object request, Class<T> responseType)`

→ Uses a full `URI`.

---

## 🧠 Parameters

|Parameter|Meaning|
|---|---|
|`url`|API endpoint|
|`request`|The request body (fields to partially update)|
|`responseType`|Type of the returned response|
|`uriVariables`|Optional path parameters|

---

## ✅ Example — Basic PATCH

```
String url = "https://api.example.com/users/{id}";

Map<String, Object> updates = new HashMap<>();
updates.put("email", "newemail@example.com");
updates.put("phone", "1234567890");

User updatedUser = restTemplate.patchForObject(url, updates, User.class, 101);

```

This sends a `PATCH` request to `/users/101` with a JSON body:

`{   "email": "newemail@example.com",   "phone": "1234567890" }`





--------------------------------------------------------------------------------------




-----------------------------------------------------------------------------------



## 🧩 What is `exchange()`?

`exchange()` is the **universal method** in `RestTemplate` that allows you to:

- Send any HTTP method (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, etc.)

- Include **headers**, **body**, and **path/query parameters**

- Receive a **ResponseEntity** (body + headers + status code)


It’s like a “manual mode” version of all other shortcut methods (`getForObject`, `postForEntity`, etc.) combined.

---

## ⚙️ Method Signatures

There are **four overloads**:

### 1️⃣

```
<T> ResponseEntity<T> exchange(
    String url,
    HttpMethod method,
    HttpEntity<?> requestEntity,
    Class<T> responseType,
    Object... uriVariables)

```

👉 Use this when you have **positional path variables**.

---

### 2️⃣

```
<T> ResponseEntity<T> exchange(
    String url,
    HttpMethod method,
    HttpEntity<?> requestEntity,
    Class<T> responseType,
    Map<String, ?> uriVariables)

```

👉 Use this when you have **named path variables**.

---

### 3️⃣

```
<T> ResponseEntity<T> exchange(
    URI url,
    HttpMethod method,
    HttpEntity<?> requestEntity,
    Class<T> responseType)

```

👉 Use this when you already built a URI with query parameters.

---

### 4️⃣

```
<T> ResponseEntity<T> exchange(
    RequestEntity<?> requestEntity,
    Class<T> responseType)

```

👉 Most flexible form — `RequestEntity` includes everything (URL, method, headers, body).

---

## 🧠 Parameters Explained

|Parameter|Description|
|---|---|
|`url`|Endpoint of the API|
|`method`|HTTP method (`GET`, `POST`, etc.)|
|`requestEntity`|Includes request body and headers (wrapped inside `HttpEntity`)|
|`responseType`|Expected response class (e.g., `User.class`)|
|`uriVariables`|Optional path variables|

---

## ✅ Example 1 — Basic GET with Headers

```
String url = "https://api.example.com/users/{id}";

HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth("your-jwt-token");

HttpEntity<Void> entity = new HttpEntity<>(headers);

ResponseEntity<User> response = restTemplate.exchange(
    url,
    HttpMethod.GET,
    entity,
    User.class,
    101
);

User user = response.getBody();
HttpStatus status = response.getStatusCode();

```

✅ You get body, status, and headers.

---

## ✅ Example 2 — POST with Request Body

```
String url = "https://api.example.com/users";

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);

User user = new User("John", "john@example.com");
HttpEntity<User> entity = new HttpEntity<>(user, headers);

ResponseEntity<User> response = restTemplate.exchange(
    url,
    HttpMethod.POST,
    entity,
    User.class
);

```

✅ Works like `postForEntity()` but gives more control.

---

## ✅ Example 3 — PUT or PATCH

```
String url = "https://api.example.com/users/{id}";

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
headers.setBearerAuth("your-jwt-token");

User updated = new User("John Updated", "john.updated@example.com");
HttpEntity<User> entity = new HttpEntity<>(updated, headers);

ResponseEntity<Void> response = restTemplate.exchange(
    url,
    HttpMethod.PUT,
    entity,
    Void.class,
    101
);

```

✅ Works for both PUT and PATCH.

---

## ✅ Example 4 — DELETE with Headers

```
String url = "https://api.example.com/users/{id}";

HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth("your-jwt-token");

HttpEntity<Void> entity = new HttpEntity<>(headers);

ResponseEntity<Void> response = restTemplate.exchange(
    url,
    HttpMethod.DELETE,
    entity,
    Void.class,
    101
);

```

✅ Works for secure DELETE (JWT, API key, etc.)

---

## ✅ Example 5 — Using `RequestEntity` (most flexible)

```
RequestEntity<User> request = RequestEntity
    .post(URI.create("https://api.example.com/users"))
    .header("Authorization", "Bearer token")
    .contentType(MediaType.APPLICATION_JSON)
    .body(new User("John", "john@example.com"));

ResponseEntity<User> response = restTemplate.exchange(request, User.class);

```

✅ Cleaner syntax — you specify everything in one chain.

---

## 🧩 `exchange()` Return Type

Always returns a `ResponseEntity<T>`, which gives you:

|Method|Description|
|---|---|
|`getBody()`|Response body|
|`getStatusCode()`|HTTP status|
|`getHeaders()`|Response headers|
|`getStatusCodeValue()`|Status code (as int)|

---

## 🧩 Common Use Cases

|Operation|Use exchange with|Why|
|---|---|---|
|Authenticated requests|JWT / Basic Auth|To send headers|
|Handling full responses|Need status or headers|Unlike `getForObject()`|
|Non-GET methods|PATCH, DELETE with headers|Other shortcuts don’t support headers|
|Error handling|Access 4xx / 5xx|To inspect status codes|



## 🧱 1️⃣ Supports **all HTTP methods**

Other methods are tied to one verb:

- `getForObject()` → GET

- `postForEntity()` → POST

- `put()` → PUT

- `delete()` → DELETE


But `exchange()` works with **any** verb — including uncommon ones:

`restTemplate.exchange(url, HttpMethod.OPTIONS, null, Void.class); restTemplate.exchange(url, HttpMethod.PATCH, entity, User.class);`

✅ You can use it for `PATCH`, `OPTIONS`, `HEAD`, or even custom methods.

---

## 🧱 2️⃣ Lets you send **headers** (Authorization, Content-Type, etc.)

Simpler methods don’t let you add headers directly.  
With `exchange()`, you can send JWT tokens, cookies, custom headers, etc.

```
HttpHeaders headers = new HttpHeaders();
headers.setBearerAuth("jwt-token");
headers.setContentType(MediaType.APPLICATION_JSON);

HttpEntity<User> entity = new HttpEntity<>(user, headers);

ResponseEntity<User> response = restTemplate.exchange(
    url,
    HttpMethod.POST,
    entity,
    User.class
);

```

✅ Perfect for secure APIs, OAuth, etc.

---

## 🧱 3️⃣ Gives full **ResponseEntity** — body, headers, and status code

Most other methods only give you the **body** (or just the status).

`exchange()` always returns a full `ResponseEntity<T>`:

```
ResponseEntity<User> response = restTemplate.exchange(...);

System.out.println(response.getStatusCode());
System.out.println(response.getHeaders());
System.out.println(response.getBody());

```

✅ You can inspect everything in the HTTP response.

---

## 🧱 4️⃣ Works with **generic response types** (`List<User>`, `Map<String, User>`, etc.)

Unlike `getForEntity()` and `getForObject()`,  
`exchange()` supports `ParameterizedTypeReference` — crucial for complex JSON responses.

```
ResponseEntity<List<User>> response = restTemplate.exchange(
    url,
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<List<User>>() {}
);

```

✅ You get proper `List<User>` instead of `List<LinkedHashMap>`.

---

## 🧱 5️⃣ Unified API for all CRUD operations

You can use **one consistent method** for all CRUD operations:

|CRUD|HTTP Method|Example with `exchange()`|
|---|---|---|
|Create|POST|`exchange(url, POST, entity, ...)`|
|Read|GET|`exchange(url, GET, null, ...)`|
|Update|PUT/PATCH|`exchange(url, PUT, entity, ...)`|
|Delete|DELETE|`exchange(url, DELETE, entity, ...)`|

✅ No need to memorize `postForEntity`, `getForObject`, etc.

---

## 🧱 6️⃣ Handles **secure APIs and authentication**

If you need to send headers for JWT, Basic Auth, or API keys, you **must** use `exchange()`.

`headers.set("Authorization", "Bearer " + token);`

✅ Other methods don’t provide a way to attach headers.

---

## 🧱 7️⃣ Better **error handling**

When the server returns a non-2xx status (like 404, 401, or 500),  
`exchange()` throws `HttpStatusCodeException`, which contains detailed response info:

```
try {
    restTemplate.exchange(...);
} catch (HttpStatusCodeException ex) {
    System.out.println(ex.getStatusCode());
    System.out.println(ex.getResponseBodyAsString());
}

```

✅ Easier to debug and log API errors.

---

## 🧱 8️⃣ Can use **RequestEntity** for cleaner code

`RequestEntity` is a builder that combines method, URL, headers, and body neatly:

```
RequestEntity<User> request = RequestEntity
    .post(URI.create("https://api.example.com/users"))
    .header("Authorization", "Bearer token")
    .contentType(MediaType.APPLICATION_JSON)
    .body(new User("John"));

ResponseEntity<User> response = restTemplate.exchange(request, User.class);

```

✅ Cleaner and less error-prone for complex requests.

---

## 🧱 9️⃣ Works with empty or void responses

For endpoints that return no content (`204 No Content`), you can use:

```
ResponseEntity<Void> response = restTemplate.exchange(url, HttpMethod.DELETE, entity, Void.class);

```

✅ Prevents null pointer or deserialization errors.

---

## 🧱 10️⃣ One place for testing and mocking

Since all other RestTemplate methods internally delegate to `exchange()`,  
using `exchange()` directly makes your **unit tests easier** (only one method to mock).


------------------------------------------------------------------------------------



------------------------------------------------------------------------------------


## 🧩 1️⃣ `HttpEntity<T>` — represents an **HTTP request or response body + headers**

Think of it as a **simple envelope** that contains:

- 📦 The **body** (`T`)
- 🧾 The **headers** (`HttpHeaders`)


It **does not** have:

- ❌ HTTP status code

- ❌ HTTP method

- ❌ URL


---

### ✅ Used for both request _and_ response (in basic form)

#### Example (as a request body)

```
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
headers.setBearerAuth("jwt-token");

User user = new User("John", "john@example.com");
HttpEntity<User> request = new HttpEntity<>(user, headers);

```

Now you can send it:

`restTemplate.exchange(url, HttpMethod.POST, request, User.class);`

#### Example (as a response)

When Spring MVC controller returns `HttpEntity`, it sends headers + body:

```
@GetMapping("/user")
public HttpEntity<User> getUser() {
    User user = new User("Alice");
    HttpHeaders headers = new HttpHeaders();
    headers.add("Custom-Header", "Value");
    return new HttpEntity<>(user, headers);
}

```
---

### 🧠 Summary

|Feature|Description|
|---|---|
|Class|`org.springframework.http.HttpEntity<T>`|
|Holds|Headers + Body|
|Has HTTP Status?|❌ No|
|Has HTTP Method or URL?|❌ No|
|Typical Use|To send headers + body to server|

---

## 🧩 2️⃣ `RequestEntity<T>` — a **subclass of `HttpEntity`** that adds HTTP method and URL

It’s basically:

> `HttpEntity` + **HTTP Method** + **URI**

So it’s a **complete HTTP request** representation.

---

### ✅ Example: Building with builder methods

```
RequestEntity<User> request = RequestEntity
    .post(URI.create("https://api.example.com/users"))
    .header("Authorization", "Bearer token")
    .contentType(MediaType.APPLICATION_JSON)
    .body(new User("John", "john@example.com"));

```

Now you can send it directly:

`ResponseEntity<User> response = restTemplate.exchange(request, User.class);`

This is the **cleanest, safest, and most modern** way to call APIs with RestTemplate.

---

### 🧠 Summary

|Feature|Description|
|---|---|
|Class|`org.springframework.http.RequestEntity<T>`|
|Extends|`HttpEntity<T>`|
|Holds|Headers + Body + HTTP Method + URL|
|Has HTTP Status?|❌ No|
|Typical Use|When sending complex requests (POST/PUT/PATCH/DELETE with headers, body, and URL)|

---

## 🧩 3️⃣ `ResponseEntity<T>` — represents an **HTTP response (from server)**

It’s also a subclass of `HttpEntity<T>` but adds:

> ✅ **HTTP Status Code**

So it represents the **entire response** from a REST call:

- 📦 Body (`T`)

- 🧾 Headers (`HttpHeaders`)

- 🟩 Status code (`HttpStatus`)


---

### ✅ Example (RestTemplate response)

```
ResponseEntity<User> response = restTemplate.exchange(
    url,
    HttpMethod.GET,
    null,
    User.class
);

System.out.println(response.getStatusCode());  // 200 OK
System.out.println(response.getHeaders());     // Response headers
System.out.println(response.getBody());        // User object

```

### ✅ Example (Spring Controller returning it)

```
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable int id) {
    User user = service.findUser(id);
    if (user == null)
        return ResponseEntity.status(HttpStatus.NOT_FOUND).build();

    return ResponseEntity.ok(user);
}

```

---

### 🧠 Summary

| Feature            | Description                                  |
| ------------------ | -------------------------------------------- |
| Class              | `org.springframework.http.ResponseEntity<T>` |
| Extends            | `HttpEntity<T>`                              |
| Holds              | Headers + Body + HTTP Status                 |
| Has Method or URL? | ❌ No                                         |
| Typical Use        | To receive or return full HTTP response      |

---

## 🧩 🔁 Relationship between them

|Class|Extends|Used For|Contains|
|---|---|---|---|
|**HttpEntity<T>**|—|Request or response (basic)|Headers + Body|
|**RequestEntity<T>**|HttpEntity|Request only|Headers + Body + Method + URL|
|**ResponseEntity<T>**|HttpEntity|Response only|Headers + Body + Status Code|



## 🧩 1️⃣ Why do we pass a `Class<T>` to RestTemplate?

When you call something like:

`User user = restTemplate.getForObject(url, User.class);`

You’re telling Spring:

> “Hey, when the HTTP response body arrives, convert (deserialize) it into a `User` object.”

So `User.class` is **type metadata** — a _hint_ for Spring’s message converters on **how to read/write JSON or XML**.

---

### 🧠 Under the hood:

1. The response body (from the server) is typically JSON or XML:

   `{   "id": 10,   "name": "John" }`

2. `RestTemplate` reads this raw stream of bytes using a **`HttpMessageConverter`** (like `MappingJackson2HttpMessageConverter`).

3. That converter uses the type you passed (`User.class`) to know **what Java object to create**.


So `User.class` helps Jackson (or any converter) do something like:

`User u = objectMapper.readValue(json, User.class);`

Without `User.class`, it wouldn’t know what type to deserialize into.

---

## 🧩 2️⃣ Serialization vs Deserialization

|Direction|Happens when|Example|Type Used|
|---|---|---|---|
|**Serialization**|Sending data (Java → JSON)|In `postForObject`, `put`, etc.|Uses request body object|
|**Deserialization**|Receiving data (JSON → Java)|In `getForObject`, `exchange`|Uses `Class<T>` or `ParameterizedTypeReference<T>`|

### Example:

`// Serialization: Java → JSON restTemplate.postForObject(url, new User("Alice"), User.class);  // Deserialization: JSON → Java User user = restTemplate.getForObject(url, User.class);`

---

## 🧩 3️⃣ Does it use reflection?

✅ **Yes — heavily.**

### Here’s how:

- Spring’s `MappingJackson2HttpMessageConverter` (default for JSON) uses Jackson’s `ObjectMapper`.

- Jackson internally uses **Java reflection** to:

  - Inspect your class (`User`) at runtime

  - Find its fields, constructors, and getter/setter methods

  - Match them with JSON property names

  - Create new instances and populate their values


Example:

`class User {     private int id;     private String name;      // getters + setters }`

When it sees `{ "id": 10, "name": "John" }`, Jackson:

- Uses reflection to find fields `id` and `name`

- Calls setters or sets fields directly

- Returns a new `User` object


No explicit code from you — all done through **reflection + annotations** (`@JsonProperty`, etc).

---

## 🧩 4️⃣ What if it’s a generic type? (Why `ParameterizedTypeReference`?)

Reflection can’t always keep track of **generic types** due to **type erasure** in Java.

Example:

```
ResponseEntity<List<User>> response = restTemplate.exchange(
    url,
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<List<User>>() {}
);

```

Here, `ParameterizedTypeReference` captures the _actual_ generic type info (`List<User>`) at runtime — something `User.class` alone can’t do.

That’s why it’s necessary when your response is a collection or map type.

---

## 🧠 Summary

|Concept|Purpose|Uses Reflection?|Example|
|---|---|---|---|
|`Class<T>`|Tells what type to deserialize into|✅ Yes|`User.class`|
|`ParameterizedTypeReference<T>`|Captures generic type info|✅ Yes|`List<User>`|
|`Object (request body)`|Serialized into JSON using Jackson|✅ Yes|`new User("John")`|

---

## ⚙️ Flow Inside `RestTemplate` (simplified)

`GET request →    HTTP response stream →    HttpMessageConverter →    Jackson ObjectMapper →    Reflection on User.class →    New User object →    Return to you`

and for POST/PUT:

`User object →    Reflection on fields →    Jackson ObjectMapper →    JSON string →    Request body`




--------------------------------------------------------------------------------------------


You’ve reached one of the deepest parts of Java’s type system — how **`ParameterizedTypeReference`** captures **generic type information** even though Java normally _erases_ it at runtime.

Let’s unpack exactly **how** that line:

`new ParameterizedTypeReference<List<User>>() {}`

manages to “store” the generic type metadata internally 👇

---

## 🧩 1️⃣ The core problem — **type erasure**

In Java, **generics are erased at runtime**.

So something like:

`List<User> users = new ArrayList<>();`

at runtime just becomes:

`List users = new ArrayList();`

Meaning:  
Java doesn’t remember that it was a `List<User>` — only that it’s a `List`.

So if you pass just `List.class`, you lose the `User` part.  
That’s why this won’t deserialize properly:

`restTemplate.exchange(url, HttpMethod.GET, null, List.class);`

➡️ It becomes a raw `List<LinkedHashMap>` because `User` type info is gone.