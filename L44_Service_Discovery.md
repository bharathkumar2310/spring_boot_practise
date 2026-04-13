
\

![img.png](Images/SD1.png)

![img_1.png](Images/sd2.png)

3. Why is it Needed?**

Without service discovery and registration:

- **Hardcoding service addresses** is required.

    - Problem: IPs/ports may change.

    - Problem: Scaling services requires updating all clients.

- **Dynamic scaling becomes impossible**.

- **Load balancing is difficult**, as clients won't know multiple instances.

- **Fault tolerance decreases**, since clients cannot switch to healthy instances automatically.


With service discovery and registration:

1. **Dynamic Lookup:** Services find other services at runtime.

2. **Automatic Load Balancing:** Registry can provide multiple instances for load distribution.

3. **High Availability:** Clients can avoid unhealthy services.

4. **Decoupling:** Services don’t need to know the exact network locations of others.



### Problems:

1. **Dynamic Scaling Breaks**

    - Suppose you spin up **3 instances** of `Order Service` for load.

    - Now the client doesn’t know about the new instances, so it keeps calling only the old one.

    - Hardcoding IPs or ports is brittle.

2. **IP Address Changes**

    - In cloud or containerized environments (like Docker or Kubernetes), IPs are **dynamic**.

    - If `Order Service` moves to a new container, hardcoded addresses fail.

3. **Fault Tolerance Issues**

    - If the only known instance fails, the client cannot find a healthy instance automatically.

4. **Tight Coupling**

    - Every service must know **exact network details** of every other service.

    - This makes changes painful and error-prone.



![img_2.png](Images/sd3.png)

![img_3.png](Images/sd4.png)

![img_4.png](Images/sd5.png)

![img_5.png](Images/sd6.png)

![img_6.png](Images/sd7.png)

![img_7.png](Images/sd8.png)

![img_8.png](Images/sd9.png)

![img_9.png](Images/sd10.png)

![img_10.png](Images/sd11.png)

![img_11.png](Images/sd12.png)

![img_12.png](Images/sd13.png)

![img_13.png](Images/sd14.png)

![img_14.png](Images/sd15.png)

![img_15.png](Images/sd16.png)

![img_16.png](Images/sd17.png)

![img_17.png](Images/sd18.png)

![img_18.png](Images/sd19.png)

![img_19.png](Images/sd0.png)

![img_20.png](Images/sd21.png)

![img_21.png](Images/sd22.png)

**4. Example Scenario**

- **Scenario 1: Maintenance**

    - You want to deploy a new version of `Payment Service` on instance B.

    - Mark B as **DOWN** → traffic stops to B → deploy → mark **UP** → traffic resumes.

- **Scenario 2: Auto-Scaling**

    - You scale down old instances that are permanently gone.

    - Deregister them → they disappear from registry → new clients won’t see them.

![img_22.png](Images/sd23.png)

![img_23.png](Images/sd24.png)

![img_24.png](Images/sd25.png)

![img_25.png](Images/sd27.png)

![img_26.png](Images/sd28.png)

1️⃣ Eureka Server – INTERNALS

### What is Eureka Server?

- A **Spring Boot app**

- Maintains an **in-memory registry** of services


`Service Name → [Instance1, Instance2, Instance3]`

---

## 🔹 Eureka Server Startup Flow

1. Application starts

2. Initializes:

    - Registry (ConcurrentHashMap)

    - Lease manager

3. Starts REST endpoints:

    - `/eureka/apps`

    - `/eureka/apps/{serviceName}`


👉 At this point, **server is empty**.

---

## 🔹 Registry Data Structure (important)

Internally:

`Map<String, Map<String, Lease<InstanceInfo>>>`

- Key → Service name

- Value → Instances with leases


---

# 2️⃣ Eureka Client – INTERNALS (MOST IMPORTANT)

Every microservice is a **Eureka Client**.

---

## 🔹 Eureka Client Startup Flow

### Step-by-step:

### 🔹 Step 1: Read Configuration

`spring:   application:     name: order-service eureka:   client:     service-url:       defaultZone: http://localhost:8761/eureka`

---

### 🔹 Step 2: Build InstanceInfo

Client creates an object called:

`InstanceInfo`

Contains:

- service name

- instance ID

- IP / hostname

- port

- health check URL

- metadata


---

### 🔹 Step 3: Register with Eureka Server

Client sends **HTTP POST**:

`POST /eureka/apps/order-service`

Payload = `InstanceInfo`

Server:

- Stores instance

- Creates **lease**

- Marks status as `UP`


---

# 3️⃣ HEARTBEAT (VERY IMPORTANT)

### Why heartbeat?

To tell Eureka:

> “I’m still alive”

---

## 🔹 How it works

- Every **30 seconds**

- Client sends:


`PUT /eureka/apps/order-service/{instanceId}`

- Server:

    - Renews lease

    - Updates timestamp


---

## 🔹 What if heartbeat stops?

- Default eviction time = **90 seconds**

- Server marks instance as **DOWN**

- Removes it from registry


---

# 4️⃣ SERVICE DISCOVERY FLOW

### When a client wants to call another service:

#### Step 1: Fetch Registry

- Client periodically pulls registry

- Default: every **30 seconds**


`GET /eureka/apps`

👉 Stored in **local cache**

---

#### Step 2: Use Local Cache

- No server call at runtime

- Faster + resilient


---

# 5️⃣ LOAD BALANCING (Feign / RestTemplate)

When calling:

`@FeignClient(name = "payment-service")`

Flow:

1. Feign → Spring Cloud LoadBalancer

2. LoadBalancer asks:

   `ServiceInstanceListSupplier`

3. Supplier reads **local Eureka cache**

4. One instance chosen (round-robin)

5. HTTP call made


---

# 6️⃣ WHAT IF EUREKA SERVER GOES DOWN?

🔥 Important interview point

- Clients **do not stop working**

- They use **cached registry**

- Default cache expiry = **3 minutes**


👉 This is why Eureka is **AP** (Availability over Consistency).

---

# 7️⃣ EUREKA SERVER SYNC (CLUSTER MODE)

Multiple Eureka servers:

1. One server receives registration

2. Replicates registry to peers

3. Eventually consistent