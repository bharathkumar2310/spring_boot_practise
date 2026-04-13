What is **Spring Cloud Gateway**?

**Spring Cloud Gateway (SCG)** is an **API Gateway** built on **Spring Boot + Spring WebFlux** that sits **in front of your microservices** and acts as a **single entry point** for all client requests.

Think of it as the **traffic controller** for your microservices system.

`Client   |   v Spring Cloud Gateway   |   |----> product-service   |----> order-service   |----> payment-service`

---

## What does Spring Cloud Gateway actually do?

At runtime, it performs **3 core things**:

### 1️⃣ Routing

Decides **which microservice** should receive the request.

`spring:   cloud:     gateway:       routes:         - id: product-route           uri: lb://PRODUCT-SERVICE           predicates:             - Path=/products/**`

➡ `/products/**` → `product-service`

---

### 2️⃣ Filtering (Most Important)

Runs **logic before and after** the request reaches the service.

Examples:

- Authentication / Authorization

- Rate limiting

- Logging

- Header modification

- Request validation


`Request → Pre-filters → Service → Post-filters → Response`

---

### 3️⃣ Integration with Service Discovery

Works seamlessly with:

- **Eureka**

- **Consul**

- **Kubernetes**


So it routes dynamically:

`lb://PRODUCT-SERVICE`

instead of hardcoded IPs.

---

## Why do we need Spring Cloud Gateway?

Let’s compare **WITHOUT Gateway vs WITH Gateway**

---

## ❌ WITHOUT Spring Cloud Gateway (Direct Calls)

`Client   |   |----> product-service (http://ip1:8081)   |----> order-service   (http://ip2:8082)   |----> payment-service (http://ip3:8083)`

### Problems:

### 🔴 1. Client must know all service URLs

- IPs change

- Scaling breaks clients

- Very tight coupling


---

### 🔴 2. Security nightmare

- Each service must implement:

    - Authentication

    - Authorization

    - Token validation

- Code duplication everywhere


---

### 🔴 3. No centralized rate limiting

- Bots can overload services

- Each service must protect itself


---

### 🔴 4. Cross-cutting concerns everywhere

Things repeated in every service:

- Logging

- CORS

- Headers

- Metrics

- Tracing


---

### 🔴 5. Hard to evolve APIs

- Changing one endpoint breaks clients

- No versioning strategy


---

## ✅ WITH Spring Cloud Gateway

`Client   |   v Spring Cloud Gateway   |   |----> product-service   |----> order-service   |----> payment-service`

### Benefits:

### 🟢 1. Single Entry Point

Clients talk to **one URL only**:

`api.mycompany.com`

---

### 🟢 2. Centralized Security

- JWT validation at gateway

- OAuth2 / Keycloak integration

- Services stay clean


---

### 🟢 3. Centralized Rate Limiting

Using **Redis / Resilience4j**:

`filters:   - name: RequestRateLimiter`

---

### 🟢 4. Load Balancing

Automatically distributes traffic:

`lb://ORDER-SERVICE`

No client-side complexity.

---

### 🟢 5. Observability

- Central logging

- Tracing (Zipkin / Sleuth / Micrometer)

- Metrics at one place


---

### 🟢 6. API Versioning & Transformation

Example:

`/v1/products → /products`

Gateway can rewrite paths without changing services.

---

## What exactly happens internally in Spring Cloud Gateway?

High-level flow:

`Request   ↓ Netty Server (WebFlux)   ↓ Route Predicate Matching   ↓ Pre Filters   ↓ Load Balancer   ↓ Target Microservice   ↓ Post Filters   ↓ Response`

Everything is:

- **Non-blocking**

- **Reactive**

- **High throughput**


---

## When should you use Spring Cloud Gateway?

Use it when:  
✔ You have **multiple microservices**  
✔ You need **centralized security**  
✔ You want **clean service code**  
✔ You expect **scaling**

---

## When NOT to use it?

Avoid it if:  
❌ You have **1–2 simple services**  
❌ Monolith application  
❌ No cross-cutting concerns







## 1️⃣ What a common-api JAR + Interceptors actually solves

Using a shared JAR with:

- `HandlerInterceptor`

- `OncePerRequestFilter`

- Common security utils

- Logging

- Correlation IDs


### ✅ What you gain

✔ No code duplication  
✔ Consistent logic  
✔ Faster development  
✔ Works perfectly for **small systems**

This is **GOOD design**, not wrong.

---

## 2️⃣ The fundamental limitation (very important)

A **common JAR works only AFTER the request reaches the service**.

`Client   ↓ Service (Interceptor runs here)`

Gateway works **BEFORE the request reaches any service**.

`Client   ↓ Gateway (decision point)   ↓ Service`

This single difference changes EVERYTHING.

---

## 3️⃣ Things a shared JAR CANNOT do (by design)

### 🔴 1. You cannot block traffic early

With interceptors:

- Request already consumed:

    - Thread

    - Memory

    - DB pool (sometimes)


Example:

`Bot sends 100k requests/sec`

❌ Each service gets hit  
❌ JVMs get exhausted

With Gateway:

`Block at the edge → services untouched`

---

### 🔴 2. You cannot hide internal services

With common JAR:

- All services are **publicly exposed**

- Anyone can hit `/order-service/**`


Gateway:

- Only gateway is public

- Services are private (VPC / cluster only)


This is **huge in real production security**.

---

### 🔴 3. You cannot do smart routing

Interceptors **do not route**.

Gateway can:

- Route by header

- Route by version

- Canary release

- Blue–green deployment


Example:

`10% traffic → v2 service`

Impossible with interceptors.

---

### 🔴 4. No protocol transformation

Gateway can:

- REST → gRPC

- WebSocket routing

- Path rewriting


Shared JAR cannot.

---

### 🔴 5. Client coupling still exists

With common JAR:

`Client → product-service Client → order-service`

Clients must know:

- All service URLs

- Versioning

- Scaling behavior


Gateway:

`Client → api.company.com`

Clients are **decoupled forever**.

---

## 4️⃣ Operational nightmare with shared JAR at scale

This is where real systems break.

### Problem: Updating logic

Change needed in auth logic?

With common JAR:

1. Update JAR

2. Rebuild ALL services

3. Redeploy ALL services


Gateway:

1. Update gateway

2. Done


🔥 This matters massively in large systems.

---

## 5️⃣ Rate limiting & DDoS protection (critical)

### With Interceptors:

- Each service rate-limits independently

- Attack still reaches all services


### With Gateway:

- Centralized rate limiting

- Redis-backed

- One place to tune


This is **edge protection**, not app logic.

---

## 6️⃣ Observability difference

Shared JAR:

- Logs spread across services

- Hard to trace client behavior


Gateway:

- First touchpoint

- Perfect place for:

    - Correlation ID

    - Request metrics

    - Traffic patterns


![img.png](Images/sg1.png)

![img_1.png](Images/sg2.png)

![img_2.png](Images/sg3.png)

![img_3.png](Images/sg4.png)

![img_4.png](Images/sg5.png)

![img_5.png](Images/sg6.png)

![img_6.png](Images/sg7.png)

![img_7.png](Images/sg8.png)

![img_8.png](Images/sg9.png)

![img_9.png](Images/sg10.png)

![img_10.png](Images/sg11.png)

![img_11.png](Images/sg12.png)

![img_12.png](Images/sg13.png)

![img_13.png](Images/sg14.png)

![img_14.png](Images/sg15.png)

![img_15.png](Images/sg16.png)

![img_16.png](Images/sg17.png)

![img_17.png](Images/sg18.png)

![img_18.png](Images/sg19.png)

![img_19.png](Images/sg20.png)

![img_20.png](Images/sg21.png)

![img_21.png](Images/sg22.png)

![img_22.png](Images/sg23.png)

![img.png](Images/sg24.png)

![img_1.png](Images/sg25.png)

![img_2.png](Images/sg26.png)

![img_3.png](Images/sg27.png)

![img_4.png](Images/sg28.png)

![img_5.png](Images/sg29.png)

![img_6.png](Images/sg30.png)

![img_7.png](Images/sg31.png)

![img_8.png](Images/sg32.png)

🌐 Spring Cloud Gateway – Internal Architecture Path

> **Important premise**  
> Spring Cloud Gateway is built on **Spring WebFlux** and **Reactor Netty**  
> ➡️ **Fully non-blocking**, **event-loop based**, **no servlet container**

---

## 1️⃣ Incoming request hits the Netty server

`Client   ↓ TCP Connection   ↓ Reactor Netty HTTP Server`

### Key components

- `reactor.netty.http.server.HttpServer`

- Event-loop threads (not Tomcat threads)


➡️ Request is wrapped into:

`ServerHttpRequest ServerHttpResponse`

---

## 2️⃣ WebFlux creates ServerWebExchange

Spring WebFlux creates:

`ServerWebExchange  ├── ServerHttpRequest  ├── ServerHttpResponse  └── Attributes (Map)`

This object is passed **through the entire gateway pipeline**.

---

## 3️⃣ Gateway WebHandler takes control

`HttpWebHandlerAdapter    ↓ FilteringWebHandler   ← ENTRY POINT OF GATEWAY`

### Important class

`org.springframework.cloud.gateway.handler.FilteringWebHandler`

This is where **Gateway logic starts**.

---

## 4️⃣ Global Filters are applied (PRE phase)

`FilteringWebHandler    ↓ GlobalFilter chain (sorted by order)`

### Examples

- `NettyWriteResponseFilter`

- `RouteToRequestUrlFilter`

- Custom `GlobalFilter`


⚠️ **At this stage:**

- Route is **NOT yet selected**

- Only system-wide logic should run


---

## 5️⃣ Route matching (Predicate evaluation)

Now Gateway must decide **where to send the request**.

### Key components

- `RouteLocator`

- `RoutePredicateHandlerMapping`


Flow:

```
Request Path / Headers
   ↓
RoutePredicateHandlerMapping
   ↓
Evaluate predicates (Path, Method, Header…)
   ↓
Matching Route found

```

➡️ Selected `Route` is stored in:

`exchange.getAttributes().put(GATEWAY_ROUTE_ATTR, route)`

---

## 6️⃣ Route-specific filters are loaded

Once route is selected:

`Global Filters     + Route Filters (for matched route)`

### Combined into:

`GatewayFilterChain`

Execution order:

`Global PRE → Route PRE → Forward request → Route POST → Global POST`

---

## 7️⃣ Request URL is prepared

Gateway rewrites the request for downstream service.

### Important filters

- `RouteToRequestUrlFilter`

- `LoadBalancerClientFilter`


If using:

`lb://PRODUCT-SERVICE`

Flow:

`Service ID   ↓ Spring Cloud LoadBalancer   ↓ Service Instance (host:port)`

Final URL is stored in:

`GATEWAY_REQUEST_URL_ATTR`

---

## 8️⃣ Request is forwarded to downstream service

`Netty Routing Filter    ↓ Reactor Netty HttpClient    ↓ Downstream Service`

### Key class

`NettyRoutingFilter`

➡️ This is where the **actual HTTP call happens**.

---

## 9️⃣ Response comes back (POST phase)

Response flow:

`Downstream Service    ↓ Reactor Netty Client    ↓ Route Filters (POST)    ↓ Global Filters (POST)`

Examples:

- Response header modification

- Logging

- Metrics


---

## 🔟 Response written back to client

`NettyWriteResponseFilter    ↓ ServerHttpResponse    ↓ Client`

No blocking, no servlet threads.

---

# 🔁 Complete Flow Diagram (Memory-friendly)


```
Client
 ↓
Reactor Netty Server
 ↓
ServerWebExchange
 ↓
FilteringWebHandler
 ↓
Global Filters (PRE)
 ↓
Route Predicate Matching
 ↓
Route Filters (PRE)
 ↓
Load Balancer
 ↓
NettyRoutingFilter
 ↓
Downstream Service
 ↓
Route Filters (POST)
 ↓
Global Filters (POST)
 ↓
NettyWriteResponseFilter
 ↓
Client

```