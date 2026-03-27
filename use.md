

![[Pasted image 20251115084738.png]]


# ✅ **What is Feign Client?**

**Feign Client** is a **Java HTTP client for microservices** (part of Spring Cloud).  
It helps you call another microservice **just by writing an interface**—no RestTemplate, no WebClient, no writing JSON parsing manually.

It is like creating a **Java interface → automatically becomes an HTTP REST client**.

---

# 🎯 **Why Feign?**

Without Feign:

```
RestTemplate restTemplate = new RestTemplate();
User user = restTemplate.getForObject("http://USER-SERVICE/users/1", User.class);

```

You write:

- URL

- HTTP method

- Serialization

- Error handling


With Feign:
```
@FeignClient(name = "USER-SERVICE")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}

```


You just call:

`userClient.getUser(1L);`

Much cleaner!


# ⭐ Benefits

### ✔ 1. Declarative HTTP Client

Just declare interface → Feign generates implementation.

### ✔ 2. Load Balancing (via Eureka + Ribbon / Spring Cloud LoadBalancer)

Feign automatically uses service name instead of URL.

### ✔ 3. Built-in Retry

Supports retries for failed calls.

### ✔ 4. Interceptors

You can add headers (JWT, API keys).

### ✔ 5. Logging

Full request/response logs.

### ✔ 6. Error Decoders

Convert error responses into custom exceptions.



# ❌ **2. RestTemplate DOES NOT integrate automatically**

With Feign:

- load balancing is automatic

- service lookup is automatic

- retries & timeouts are simple to configure

- logging is built-in

- headers are auto-binded

- proxy generation automatically creates the full call


RestTemplate requires:

- custom interceptors

- custom retry templates

- custom error handlers

- manually building URLs

- manually setting headers


----------------------------------------------------------------------------------------




![[Pasted image 20251115085226.png]]



![[Pasted image 20251115085500.png]]


![[Pasted image 20251115085523.png]]


![[Pasted image 20251115085556.png]]


![[Pasted image 20251115085625.png]]




# **1️⃣ FeignClientsRegistrar gets triggered**

When Spring sees:

`@EnableFeignClients`

➡️ It loads **FeignClientsRegistrar**.

This registrar:

- scans your classpath

- finds all interfaces annotated with `@FeignClient`

- registers one **FeignClientFactoryBean** per Feign interface


✔ At this stage, **NO proxy created yet**.  
✔ Only Spring bean definitions.

---

# **2️⃣ For each @FeignClient interface**

FeignClientFactoryBean creates:

## **✔ One `Feign.Builder`**

Contains:

- Encoder

- Decoder

- Contract (annotation parser)

- Retryer

- ErrorDecoder

- Logger

- Client (OkHttp, HttpClient)


---

# **3️⃣ Feign parses methods → creates MethodHandlers**

Correcting your point:

👉 Feign **does NOT create a MethodHandler for each method directly**.

❗ It first creates a **RequestTemplate** for each method:

- URL template

- HTTP method

- headers

- path variables

- query params

- body template


Then Feign creates one **SynchronousMethodHandler** per method.

Each MethodHandler contains:

- the RequestTemplate

- Encoder

- Decoder

- Retryer

- LoadBalancer or base URL

- Exception handling logic


➡️ All MethodHandlers are stored in a `Map<Method, MethodHandler>` called **dispatch**.

✔ **This part you understood correctly.**

---

# **4️⃣ InvocationHandler created**

Feign builds a **FeignInvocationHandler**.

This is the “brain”.

It contains:

- the `Map<Method, MethodHandler>`

- logic to select the correct handler


✔ **It does NOT contain encoder/decoder**  
These are inside each MethodHandler.

---

# **5️⃣ JDK Dynamic Proxy is created**

Finally:

`Proxy.newProxyInstance(...)`

Creates a proxy implementing your Feign interface.

✔ This object is the actual bean injected into your Services.

---

# **6️⃣ Runtime: When you call a method**

Your understanding is correct, but let me refine it:

### **Step-by-step:**

`userClient.getUser(10)`

1️⃣ Proxy intercepts the call  
2️⃣ Proxy calls InvocationHandler.invoke()  
3️⃣ InvocationHandler finds the matching MethodHandler from dispatch map  
4️⃣ MethodHandler builds the HTTP request  
5️⃣ Sends request using the HTTP client (OkHttp, Apache)  
6️⃣ Decoder converts response to the return type  
7️⃣ Result returned to your service

✔ This part is **100% correct in your explanation**.

---

# ✅ **Your Version (Corrected & polished)**

Here is the final, interview-ready, accurate version:

---

### **When Spring sees @EnableFeignClients:**

### **1. FeignClientsRegistrar runs**

- Scans all interfaces with `@FeignClient`.

- Registers a `FeignClientFactoryBean` for each.


### **2. Feign builds for each interface:**

- A Feign Builder

- Parses each method

- Creates a `RequestTemplate`

- Creates one `MethodHandler` per method

- Stores them in a **dispatch map**


### **3. Feign creates an InvocationHandler**

- Bridges method calls from proxy → method handlers


### **4. JDK Dynamic Proxy created**

- Implements your Feign interface

- Uses InvocationHandler to handle all calls


### **5. At runtime:**

- Proxy intercepts call

- Invokes InvocationHandler

- Which finds the correct MethodHandler

- Which builds & sends the HTTP request

- Decodes response

- Returns result





![[Pasted image 20251115114147.png]]
![[Pasted image 20251115114212.png]]



![[Pasted image 20251115114244.png]]





![[Pasted image 20251222094145.png]]



![[Pasted image 20251222094159.png]]



![[Pasted image 20251222094220.png]]


![[Pasted image 20251222094239.png]]




![[Pasted image 20251222094312.png]]


![[Pasted image 20251222094335.png]]


![[Pasted image 20251222094348.png]]





![[Pasted image 20251222094404.png]]


![[Pasted image 20251222094419.png]]




![[Pasted image 20251222094449.png]]




![[Pasted image 20251222094509.png]]



![[Pasted image 20251222094525.png]]



![[Pasted image 20251222094539.png]]



![[Pasted image 20251222094550.png]]



![[Pasted image 20251222094609.png]]



![[Pasted image 20251222094625.png]]



![[Pasted image 20251222094639.png]]


![[Pasted image 20251222094655.png]]




![[Pasted image 20251222094722.png]]


![[Pasted image 20251222094747.png]]


![[Pasted image 20251222094809.png]]

![[Pasted image 20251222094826.png]]



![[Pasted image 20251222094844.png]]



![[Pasted image 20251222094910.png]]



![[Pasted image 20251222094922.png]]



---

# 1️⃣ What is “boilerplate” in RestTemplate?

**Boilerplate = repeated, mechanical code you must write every time**, even though the _intent_ is the same.

### Example: calling `order-service` using **RestTemplate**

```
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public Order getOrder(Long id) {

        String url = "http://order-service/orders/" + id;

        ResponseEntity<Order> response =
            restTemplate.exchange(
                url,
                HttpMethod.GET,
                null,
                Order.class
            );

        return response.getBody();
    }
}

```

### 🔴 What’s boilerplate here?

Every REST call repeats:

1. Construct URL

2. Choose HTTP method

3. Handle `ResponseEntity`

4. Specify response type

5. Handle exceptions manually

6. Add headers manually (auth, trace-id)

7. Integrate load balancing manually


You repeat this **for every endpoint, every service**.

---

# 2️⃣ Same call using Feign Client

```
@FeignClient(name = "order-service")
public interface OrderClient {

    @GetMapping("/orders/{id}")
    Order getOrder(@PathVariable Long id);
}

```
And call it like:

`Order order = orderClient.getOrder(id);`

---

# 3️⃣ So WHAT exactly got removed?

|Concern|RestTemplate|Feign|
|---|---|---|
|URL building|Manual|Automatic|
|HTTP method|Manual|Annotation|
|Serialization|Manual|Automatic|
|ResponseEntity|Manual|Hidden|
|Load Balancing|Manual|Automatic|
|Error handling|Manual|Centralized|
|Headers|Manual|Interceptor|
|Retry|Manual|Built-in|

👉 **You only write business intent**, not HTTP mechanics.