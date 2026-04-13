

![img.png](Images/lb1.png)

![img_1.png](Images/lb2.png)

![img_2.png](Images/lb3.png)

![img_3.png](Images/lb4.png)

![img_4.png](Images/lb5.png)

![img_5.png](Images/lb6.png)

![img_6.png](Images/lb7.png)


![img_7.png](Images/lb8.png)


![img_8.png](Images/lb9.png)

![img_9.png](Images/lb10.png)

![img_10.png](Images/lb11.png)


![img_11.png](Images/lb12.png)


![img_12.png](Images/lb13.png)


![img_13.png](Images/lb14.png)

![img_14.png](Images/lb15.png)


![img_15.png](Images/lb16.png)


![img_16.png](Images/lb17.png)

![img_17.png](Images/lb18.png)

# ⭐ FULL DETAILED INTERNAL FLOW (from your code → Eureka → LoadBalancer → Instance)

Assume you call:

`restTemplate.getForObject(     "http://ORDER-SERVICE/orders",     String.class );`

Now let’s break the entire flow from start to finish.

---

# 🔵 **STEP 1 — You call the RestTemplate API**

You call `.getForObject()`.

Normally RestTemplate would directly send an HTTP request.  
But since you annotated your bean with:

`@LoadBalanced`

Spring decorated RestTemplate internally by adding a **Load Balancer Interceptor**.

---

# 🔵 **STEP 2 — LoadBalancerInterceptor intercepts the request**

Before RestTemplate sends the HTTP call, the interceptor gets the chance to inspect & modify the request.

It sees the URL:

`http://ORDER-SERVICE/orders`

and detects that **ORDER-SERVICE** is NOT an IP/host but a **Service ID**.

So it knows:  
👉 "I cannot send this directly. I must resolve this service name."

---

# 🔵 **STEP 3 — Interceptor asks the Load Balancer for a service instance**

The interceptor calls the **LoadBalancer Client** with:

- Service name → `ORDER-SERVICE`

- Request context → URL, headers, body


The load balancer now takes over.

---

# 🔵 **STEP 4 — Load Balancer asks Eureka for all available instances**

It contacts the registered discovery client (Eureka client).

Eureka returns something like:

`ORDER-SERVICE instances: 1. 10.0.0.10:8081 2. 10.0.0.11:8082 3. 10.0.0.12:8083`

These are **healthy, alive** instances based on Eureka's heartbeat mechanism.

---

# 🔵 **STEP 5 — LoadBalancer applies an algorithm to select one instance**

Default Spring Cloud LoadBalancer algorithm:  
**Round Robin**

But depending on config, it may be:

- Random

- Weighted

- Zone preference

- Custom strategy

- Retry-based


The algorithm picks _ONE_ instance from the list.

Example:

It picks:

`10.0.0.11:8082`

---

# 🔵 **STEP 6 — The interceptor rewrites the URL**

Original URL:

`http://ORDER-SERVICE/orders`

Rewritten to:

`http://10.0.0.11:8082/orders`

This is the actual, physical reachable host.

---

# 🔵 **STEP 7 — RestTemplate executes the HTTP request normally**

Now the interceptor is done.

It passes this **resolved real URL** into the actual RestTemplate HTTP execution chain.

RestTemplate performs:

- Create HTTP connection

- Send request

- Receive response

- Apply message converters

- Return response body


The request is now a simple HTTP call — load balancer work is already done.

---

# 🔵 **STEP 8 — Response is returned to your application**

The JSON body or object gets returned exactly like a normal RestTemplate call.


-------------------------------------------------------------------------------

## 1️⃣ RestTemplate owns the execution flow

When you call:

`restTemplate.getForObject(url, String.class);`

Internally, **RestTemplate itself** executes the request.

You are NOT calling:

`client.execute()`

Spring is.

---

## 2️⃣ RestTemplate has an interceptor chain

Inside `RestTemplate`:

`List<ClientHttpRequestInterceptor> interceptors;`

This list is populated **at startup**.

When you add:

`@LoadBalanced @Bean RestTemplate restTemplate() { ... }`

Spring auto-adds:

`LoadBalancerInterceptor`

to this list.

---

## 3️⃣ Execution flow (this is the core)

When `RestTemplate.execute()` runs, it does something like:

`ClientHttpResponse execute(HttpRequest request) {     return new InterceptingRequestExecution(interceptors)              .execute(request); }`

Internally simplified:

`Interceptor 1 → Interceptor 2 → Actual HTTP call`

Each interceptor explicitly calls the next one.

---

## 4️⃣ Interceptor is NOT automatic — it is MANUAL

This is critical:

> The interceptor is not magically invoked by JVM  
> It is explicitly invoked by RestTemplate’s code

Pseudo-code:

`for (Interceptor i : interceptors) {     response = i.intercept(request, body, next); }`

That’s it.

No proxy.  
No reflection.  
No bytecode tricks.
---

# 🔥 COMPLETE FLOW: Startup → First Call → Subsequent Calls

---

## 🟢 PHASE 1: APPLICATION STARTUP

---

### 1️⃣ Spring Boot application starts

`main()  → SpringApplication.run()`

Creates:

- `ApplicationContext`

- Loads auto-configurations


---

### 2️⃣ Spring Cloud auto-configuration kicks in

Loaded automatically via:

`spring.factories / AutoConfiguration.imports`

Key auto-configs involved:

- `EurekaClientAutoConfiguration`

- `LoadBalancerAutoConfiguration`

- `LoadBalancerClientConfiguration`

- `FeignAutoConfiguration` (if Feign)


---

## 🟢 PHASE 2: EUREKA CLIENT INITIALIZATION

---

### 3️⃣ Eureka client beans are created

Important beans:

- `DiscoveryClient`

- `EurekaClient`

- `ApplicationInfoManager`


---

### 4️⃣ Service registration with Eureka Server

On startup:

`EurekaClient  → register()  → HTTP POST /eureka/apps/{appName}`

✔ Registers:

- serviceId

- host

- port

- metadata

- health status


---

### 5️⃣ Eureka client fetches registry

`EurekaClient  → fetchRegistry()`

✔ Gets **full registry**  
✔ Stores it in **local in-memory cache**

---

### 6️⃣ Heartbeat & registry refresh scheduled

Two background tasks start:

|Task|Default|
|---|---|
|Heartbeat|30s|
|Registry refresh (delta)|30s|

---

## 🟢 PHASE 3: LOAD BALANCER INFRASTRUCTURE SETUP

---

### 7️⃣ LoadBalancer core beans created (startup)

Created **once**:

- `LoadBalancerClient`

- `LoadBalancerClientFactory`

- `LoadBalancerProperties`


⚠️ **NO algorithm yet**

---

### 8️⃣ `@LoadBalanced RestTemplate` handling

`@LoadBalanced @Bean RestTemplate restTemplate()`

Spring does:

- Registers BeanPostProcessor

- Adds `LoadBalancerInterceptor`

- Interceptor bean created

- Attached to RestTemplate


✔ Happens **at startup**

---

### 9️⃣ ServiceInstanceListSupplier infrastructure created

Base suppliers registered:

- `DiscoveryClientServiceInstanceListSupplier`

- `CachingServiceInstanceListSupplier`


⚠️ These are **factories**, not per-service yet

---

## 🟢 PHASE 4: APPLICATION READY

At this point:  
✔ Eureka registered  
✔ Registry cached  
✔ LoadBalancer ready  
✔ No HTTP calls yet

---

# 🔥 PHASE 5: FIRST SERVICE-TO-SERVICE CALL (IMPORTANT)

---

### 1️⃣ Controller/service calls RestTemplate

`restTemplate.getForObject(  "http://ORDER-SERVICE/api/orders",  String.class )`

---

### 2️⃣ RestTemplate execution begins

`RestTemplate.execute()  → Interceptor chain`

---

### 3️⃣ LoadBalancerInterceptor intercepts request

Responsibilities:

- Extract serviceId → `ORDER-SERVICE`

- Delegate to `LoadBalancerClient`


---

### 4️⃣ LoadBalancerClient is called

`loadBalancerClient.choose("ORDER-SERVICE")`

---

### 5️⃣ LoadBalancerClientFactory invoked

`getInstance("ORDER-SERVICE", ReactorLoadBalancer.class)`

🚨 **Key moment**

---

### 6️⃣ Child ApplicationContext created (lazy)

If first time for `ORDER-SERVICE`:

- Child context created

- Bound to `ORDER-SERVICE`


---

### 7️⃣ Load balancer algorithm created

Inside child context:

`RoundRobinLoadBalancer(   ServiceInstanceListSupplier,   "ORDER-SERVICE" )`

✔ serviceId injected  
✔ Algorithm instance is **per serviceId**

---

### 8️⃣ ServiceInstanceListSupplier chain created

Order:

`CachingServiceInstanceListSupplier  → DiscoveryClientServiceInstanceListSupplier  → Eureka DiscoveryClient`

---

### 9️⃣ Instances fetched from local Eureka cache

✔ NO Eureka Server call  
✔ Uses local registry

---

### 🔟 Load balancing algorithm executes

Round-robin:

- Maintains AtomicInteger

- Chooses one instance


---

### 1️⃣1️⃣ URL rewritten

From:

`http://ORDER-SERVICE/api/orders`

To:

`http://10.0.0.12:8080/api/orders`

---

### 1️⃣2️⃣ Actual HTTP request executed

- Via HTTP client

- Response returned


---

# 🔁 PHASE 6: SUBSEQUENT REQUESTS

---

### ✔ No new creation happens

Reused:

- Same interceptor

- Same LoadBalancerClient

- Same child context

- Same RoundRobinLoadBalancer


Only:

- Instance selection logic runs again


---

## 🟢 PHASE 7: REGISTRY UPDATES (BACKGROUND)

---

### Eureka client refreshes registry

Every 30 seconds:

- Delta updates fetched

- Local cache updated


Load balancer automatically sees:

- New instances

- Removed instances


---

## 🟢 PHASE 8: FAILURE SCENARIOS

---

### Eureka Server DOWN

✔ Calls continue  
✔ Uses last known registry  
✔ System remains available

---

### Instance DOWN

- Eureka marks instance DOWN

- Delta refresh updates cache

- LoadBalancer stops routing traffic


---

# 🧠 COMPLETE COMPONENT MAP

```
Controller
 → RestTemplate
 → LoadBalancerInterceptor
 → LoadBalancerClient
 → LoadBalancerClientFactory
 → Child Context (serviceId)
 → RoundRobinLoadBalancer
 → ServiceInstanceListSupplier
 → DiscoveryClient (local cache)
 → HTTP Client

```