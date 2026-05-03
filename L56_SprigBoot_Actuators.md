ACTUATORS :

    ---> Helps in monitoring and managing appliactions in production
    ---> Provides built in features to track applications health, metrics and internal state through easy to use endpoints
    ---> Helps track metrics, performance, and system behavior for better management


| Endpoint            | Purpose                       |
| ------------------- | ----------------------------- |
| `/actuator/health`  | Check app health              |
| `/actuator/info`    | App info                      |
| `/actuator/metrics` | JVM, memory, CPU metrics      |
| `/actuator/env`     | Environment properties        |
| `/actuator/beans`   | All Spring beans              |
| `/actuator/loggers` | Change log levels dynamically |



| Category         | Examples                            |
| ---------------- | ----------------------------------- |
| Application Info | App name, version, description      |
| Build Info       | Build time, artifact name           |
| Git Info         | Commit ID, branch, last commit time |
| Custom Info      | Team name, environment, region      |

        
        {
        "app": {
        "name": "order-service",
        "version": "1.0.0",
        "description": "Handles order processing"
        },
        "build": {
        "artifact": "order-service",
        "version": "1.0.0",
        "time": "2026-04-30T10:00:00Z"
        },
        "git": {
        "branch": "main",
        "commit": {
        "id": "a1b2c3d"
        }
        }
        }


METRICS : 

| Category                | Examples                              |
| ----------------------- | ------------------------------------- |
| JVM Memory              | Heap usage, non-heap usage            |
| CPU                     | CPU usage                             |
| Threads                 | Active threads, daemon threads        |
| HTTP Requests           | Count, response time, errors          |
| GC (Garbage Collection) | GC pause time, count                  |
| System                  | Uptime, system load                   |
| DB / Custom             | Connection pool stats (if configured) |




🔍 Why do we need Actuator?

When your app is running in production, you need to know:

    Is the app alive?
    Is the database connected?
    How much memory is being used?
    What endpoints are available?
    Can I safely restart or shut down the app?

Instead of writing all this manually, Actuator gives it out-of-the-box.


Dependency to add :
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>


Enable endpoints in application.properties
        
        management.endpoints.web.exposure.include=*
        management.endpoint.health.show-details=always

We can access

👉 http://localhost:8080/actuator/health

    {
        "status" : "UP"
    }



🔥 Real-world usage

1. Health checks (Very important in microservices)
        
        Tools like Kubernetes use /actuator/health to decide:
        
        Restart app if DOWN
        Route traffic only to healthy instances

2. Monitoring

Integrate with:

    Prometheus
    Grafana

To track:
    
    Memory usage
    CPU usage
    Request count

3. Debugging in production

Check:
    
    /actuator/env → wrong config?
    /actuator/beans → bean loaded?
    /actuator/loggers → increase log level dynamically



🔹 1. To detect failures early

Example:
    
    Your API suddenly starts returning 500 errors
    DB connection pool is exhausted
    
    Monitoring tools (like Prometheus + Grafana) alert you immediately.
    
    👉 Without monitoring: users complain first
    👉 With monitoring: you fix it before users notice

🔹 2. To track system health

Using Spring Boot Actuator:
    
    /actuator/health → is app UP?
    /actuator/metrics → memory, CPU, threads
    
    Used by tools like:
    
    Kubernetes (auto-restarts unhealthy apps)

🔹 3. Performance tracking

Example:
        
        API response time increases from 100ms → 2s
        Memory usage slowly rising (possible leak)
        
        Monitoring helps you catch trends before failure happens.

🔹 4. Capacity planning
    
    Example:
    
    Traffic increasing daily
    CPU consistently at 80%
    
    👉 You decide:
    
    Scale horizontally
    Optimize queries


🔍 1. Check if the problem is system-level or code-level
    
    Endpoint:
    /actuator/health
    Example:
    {
    "status": "DOWN",
    "components": {
    "db": { "status": "DOWN" }
    }
    }

👉 Debug insight:
    
    Not a code bug ❌
    Database connection issue ✅

🔍 2. Debug wrong configurations
    
    Endpoint:
    /actuator/env
    
    👉 Shows all environment properties
    
    Example problem:
    API failing due to wrong DB URL
    Wrong port / timeout config
    
    👉 You can verify actual runtime values, not what you think you configured.

🔍 3. Check if beans are loaded correctly
    
    Endpoint:
    /actuator/beans
    
    👉 Shows all Spring beans
    
    Example:
    @Service not working
    Dependency injection failing
    
    👉 You confirm:
    
    Is the bean created?
    Is it wired correctly?

🔍 4. Analyze performance issues
    
    Endpoint:
    /actuator/metrics
    
    👉 Gives:
    
    Memory usage
    CPU usage
    Thread count
    HTTP request timings
    Example:
    API slow only in production
    
    👉 You check:
    
    High memory?
    Too many threads?

🔍 5. Change logging dynamically (VERY powerful)
    
    Endpoint:
    /actuator/loggers
    
    👉 You can increase log level at runtime:
    
    POST /actuator/loggers/com.myapp
    {
    "configuredLevel": "DEBUG"
    }
    
    👉 No restart needed!
    
    Use case:
    Bug happening in production
    Enable DEBUG logs temporarily
    Capture detailed info
    Turn it back to INFO
🔍 6. Understand request flow issues
    
    Metrics like:
    
    /actuator/metrics/http.server.requests
    
    👉 Help debug:
    
    High response time
    Frequent failures
    Which endpoints are problematic



🚨 Why you should NOT expose all Actuator endpoints

    Using Spring Boot Actuator, some endpoints expose sensitive internal details.

🔥 1. Security risk (biggest reason)
    
    Endpoint:
    /actuator/env

👉 Can expose:
    
    DB URLs
    API keys
    Secrets (if not masked properly)
    
    💥 If public:
    
    Anyone can see your configuration → huge security vulnerability

🔥 2. Internal structure leakage
    
    Endpoint:
    /actuator/beans
    
    👉 Shows:
    
    All classes
    Internal architecture
    Dependencies
    
    💥 Attackers can:
    
    Understand your system design
    Find weak points

🔥 3. Ability to modify behavior (VERY dangerous)
    
    Endpoint:
    /actuator/loggers
    
    👉 Allows:
    
    Changing log levels at runtime
    
    💥 Misuse:
    
    Turn on DEBUG everywhere → performance drop
    Fill logs → disk crash
    Even worse (if enabled):

/actuator/shutdown
    
    💥 Anyone can:
    
    Shut down your application remotely ❌

🔥 4. Performance impact
    
    Some endpoints:
    
    /actuator/metrics
    /actuator/threaddump
    
    👉 Heavy operations
    
    💥 If publicly accessible:
    
    Can be spammed
    Causes CPU/memory overhead
🔥 5. Helps attackers plan attacks
    
    Example:
    
    /health → shows DB type
    /metrics → shows load patterns
    
    👉 Attackers learn:
    
    When system is weakest
    What tech stack you use
    ⚠️ Real-world rule
    
    👉 Never expose this in production:
    
    management.endpoints.web.exposure.include=*

❌ Very unsafe

✅ What should you do instead?

    1. Expose only safe endpoints
       management.endpoints.web.exposure.include=health,info
2. Secure endpoints

Use:
    
    Authentication (Spring Security)
    IP restrictions
    Internal network only
    3. Use monitoring tools instead
    
    Instead of public access:
    
    Prometheus
    Grafana
    
    👉 These securely fetch metrics



To cretae custom actuator endpoint

    
    import org.springframework.boot.actuate.endpoint.annotation.Endpoint;
    import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;
    import org.springframework.stereotype.Component;
    
    @Component
    @Endpoint(id = "custom")
    public class CustomEndpoint {
    
        @ReadOperation
        public String message() {
            return "Application is running fine!";
        }
    }


    
    import org.springframework.boot.actuate.health.Health;
    import org.springframework.boot.actuate.health.HealthIndicator;
    import org.springframework.stereotype.Component;
    
    @Component
    public class MyHealthCheck implements HealthIndicator {
    
        @Override
        public Health health() {
            if (checkSomething()) {
                return Health.up().withDetail("service", "Working").build();
            } else {
                return Health.down().withDetail("service", "Not reachable").build();
            }
        }
    
        private boolean checkSomething() {
            return true;
        }
    }



Different ports for actuator (VERY COMMON QUESTION)

    
    management:
    server:
    port: 9090
    
    👉 Now actuator runs on:
    
    http://localhost:9090/actuator/health
    
    💡 Interview answer:
    
    “We can run actuator on a separate port for security and isolation.”


🔥 3. Liveness vs Readiness (VERY IMPORTANT in microservices)

This is commonly asked with Kubernetes.

👉 You should know:
    
    Liveness
    → Is app alive? (restart if dead)
    Readiness
    → Is app ready to serve traffic?
    
    👉 Spring Boot:
    
    /actuator/health/liveness
    /actuator/health/readiness

💡 This is a BIG differentiator in interviews.
