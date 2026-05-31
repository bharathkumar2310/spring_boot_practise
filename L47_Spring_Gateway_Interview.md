1. What is Spring Cloud Gateway?
   Answer

Spring Cloud Gateway is an API Gateway built on top of:

Spring WebFlux
Project Reactor

It provides:

routing
filtering
authentication
rate limiting
resiliency
load balancing

for microservices.

It acts as:

Client → Gateway → Microservices
2. Why do we need API Gateway?

Without gateway:

clients call every microservice directly
authentication duplicated
routing logic duplicated
CORS issues
hard to manage APIs

Gateway provides:

single entry point
centralized security
centralized logging
centralized routing

3. Real-Time Example of API Gateway

Example:

User Service
Order Service
Payment Service

Instead of:

Frontend → all services directly

Use:

Frontend → API Gateway → services

Benefits:

hides internal services
easier security
centralized monitoring
4. How does Spring Cloud Gateway work internally?

Internally:

receives request
matches route predicates
applies filters
forwards request to downstream service

Built on:

non-blocking reactive Netty server


5. What are Route Predicates?

Predicates decide:

where request should go

Example:

routes:
- id: order-service
  uri: lb://ORDER-SERVICE
  predicates:
    - Path=/orders/**

If path matches:

/orders/**

request routed to:

ORDER-SERVICE
6. Common Predicate Types
   Predicate	Purpose
   Path	Match URL path
   Method	Match HTTP method
   Header	Match header
   Query	Match query param
   Host	Match host
   Cookie	Match cookies
   After/Before	Time-based routing
7. What are Filters in Gateway?

Filters modify:

requests
responses

Examples:

authentication
logging
adding headers
rate limiting

8. Types of Filters
   Global Filters

Applied to all routes.

Route-specific Filters

Applied to specific route only.

9. Real Interview Question
   Difference Between Predicate and Filter
   Predicate	Filter
   Decides routing	Modifies request/response
   Route matching	Request processing
   Boolean condition	Processing logic
10. What is Global Filter?

Runs for every request.

Used for:

logging
authentication
tracing
11. What is GatewayFilter?

Applied only to specific routes.

Example:

filters:
- AddRequestHeader=X-App, gateway
12. How does service discovery work with Gateway?

Gateway integrates with:

Netflix Eureka
Consul
Kubernetes discovery

Instead of fixed URL:

uri: lb://ORDER-SERVICE

Gateway asks discovery server:

Where is ORDER-SERVICE running?

Then forwards request.

13. Why is Gateway useful if Eureka already gives URL?

Very common interview question.

Eureka:

only provides service location

Gateway:

handles routing
security
filters
rate limiting
monitoring
authentication

Eureka solves:

service lookup

Gateway solves:

traffic management
14. What does lb:// mean?
    uri: lb://USER-SERVICE

Means:

use load balancer
resolve instances from discovery server

Gateway automatically load balances requests.

15. What Load Balancer is used?

Usually:

Spring Cloud LoadBalancer

Older systems used Ribbon.

16. Difference Between Zuul and Spring Cloud Gateway
    Zuul	Spring Cloud Gateway
    Servlet based	Reactive
    Blocking	Non-blocking
    Older	Modern
    Less performant	Better scalability
17. Why is Spring Cloud Gateway faster?

Because:

non-blocking I/O
reactive Netty server
fewer threads needed

Can handle many concurrent requests efficiently.

18. What is Reactive Programming in Gateway?

Gateway uses:

Reactor
Mono
Flux

Threads are not blocked while waiting.

Better scalability.

19. What is Netty?

Netty is an asynchronous event-driven networking framework.

Gateway uses Netty instead of Tomcat by default.

20. Real Interview Question
    Why is Gateway built on WebFlux and not Spring MVC?

Because API gateways:

handle massive concurrent traffic
mostly I/O operations

Reactive model scales better.

21. Can Spring Cloud Gateway work with MVC applications?

Yes.

Gateway itself is reactive.

Downstream services can still be:

Spring MVC
Spring Boot servlet apps
22. What is Rate Limiting in Gateway?

Controls request rate.

Example:

100 requests/minute

Used to:

prevent abuse
protect services
prevent DDOS
23. How is Rate Limiting implemented?

Usually using:

Redis token bucket algorithm
24. What is Token Bucket Algorithm?

Tokens generated at fixed rate.

Each request consumes token.

No token:
→ request rejected.

25. Real Interview Scenario
    “Public API is getting spammed.”

Use:

Gateway rate limiting
26. How is Authentication handled in Gateway?

Gateway can:

validate JWT
authenticate user
forward user details downstream

Centralized authentication.

27. Why centralize authentication in Gateway?

Without gateway:
every service must:

validate JWT
implement security

Duplication happens.

Gateway simplifies this.

28. What headers are commonly added by Gateway?

Examples:

Authorization
Correlation ID
Trace ID
Custom headers
29. What is Correlation ID?

Unique request identifier.

Used for:

distributed tracing
debugging logs

Across microservices.

30. Real Interview Scenario
    “How do you trace one request across 10 microservices?”

Use:

correlation IDs
distributed tracing

Gateway often generates trace ID.

31. What is Circuit Breaker in Gateway?

Gateway integrates with:
Resilience4j

Example:

filters:
- name: CircuitBreaker

If downstream service fails:

fallback response returned
32. Why use Circuit Breaker at Gateway layer?

Prevents:

failing services from overwhelming clients
cascading failures
33. What is Fallback URI?

Alternative endpoint during failure.

Example:

fallbackUri: forward:/fallback
34. What is Request Rewrite?

Gateway can modify URL path.

Example:

RewritePath=/api/(?<segment>.*), /$\{segment}

Useful for:

versioning
backward compatibility
35. What is CORS in Gateway?

Gateway can centrally handle:

Cross-Origin Resource Sharing

Prevents configuring CORS in every service.

36. What is Header Manipulation?

Gateway can:

add headers
remove headers
modify headers
37. Common Built-in Filters
    Filter	Purpose
    AddRequestHeader	Add header
    AddResponseHeader	Add response header
    RewritePath	Modify path
    Retry	Retry failed requests
    CircuitBreaker	Fault tolerance
    RequestRateLimiter	Rate limiting
38. What is Retry Filter?

Retries failed downstream requests.

Useful for:

temporary network failures

Danger:

retry storms
39. Real Interview Question
    When should Retry NOT be used?

Avoid:

payment APIs
non-idempotent operations

Can create duplicate actions.

40. What is Forward Routing?

Internal forwarding inside gateway.

Example:

forward:/fallback

No external HTTP call made.

41. What is Difference Between Redirect and Forward?
    Redirect	Forward
    Client makes new request	Internal forwarding
    URL changes	URL unchanged
    Extra network call	No extra call
42. What is Gateway Timeout?

Downstream service takes too long.

Gateway should configure:

connect timeout
response timeout

To avoid hanging requests.

43. Real Interview Question
    Why are timeouts critical in API Gateway?

Without timeout:

threads/connections remain occupied
system becomes slow
resource exhaustion occurs
44. How does Gateway support load balancing?

Using:

lb://SERVICE-NAME

Gateway distributes traffic across instances.

45. What load balancing algorithms are used?

Usually:

round robin

Can customize.

46. Can Gateway do SSL termination?

Yes.

HTTPS handled at gateway.

Internal services may use HTTP.

47. Why terminate SSL at Gateway?

Benefits:

centralized certificate management
reduced service complexity
performance improvement
48. What is API Versioning in Gateway?

Gateway can route:

/v1/**
/v2/**

to different services.

49. Common Production Problems in Gateway
    Timeout misconfiguration
    Retry storms
    Memory leaks due to large payloads
    Missing rate limiting
    Blocking calls in reactive flow
    Large request buffering
50. Biggest Interview Trap
    “Can I use blocking RestTemplate inside Gateway filter?”

Correct answer:
NO.

Because Gateway is reactive.

Blocking calls:

block Netty event loop
destroy scalability

Use:

WebClient

instead.

51. Difference Between WebClient and RestTemplate in Gateway
    WebClient	RestTemplate
    Non-blocking	Blocking
    Reactive	Servlet based
    Recommended	Deprecated for reactive use
52. What happens if blocking calls are used in Gateway?

Very important question.

Netty event loop threads get blocked.

Results:

poor throughput
request delays
scalability collapse
53. Real Production Architecture

Typical flow:

Client
→ API Gateway
→ Auth Filter
→ Rate Limiter
→ Routing
→ Microservices



What is Netty?

Netty is a low-level asynchronous networking framework for Java.

It helps build:

HTTP servers
WebSockets
TCP servers
API gateways
high-performance network applications

It is built around:

non-blocking I/O + event loops
Why was Netty created?

Traditional Java servers:

create many threads
each request blocks a thread

Problem:

huge memory usage
context switching overhead
poor scalability

Netty solves this using:

few threads handling many connections
Traditional Blocking Model

Example:

1 request = 1 thread

If request waits:

thread also waits

So:

10,000 requests
→ may need thousands of threads

This is expensive.

Netty Model

Netty uses:

event loop architecture

Small number of threads:

monitor many connections

When I/O ready:

event triggered
processing happens

No thread sits idle waiting.

Real Analogy
Blocking Model

Like:

1 waiter per customer

Very expensive at scale.

Netty/Event Loop

Like:

few waiters handling many tables

Efficient.

What is Event Loop?

Core Netty concept.

Event loop:

continuously checks events
processes ready tasks

Example:

request received
response ready
socket writable
Why is Netty Fast?

Because:

fewer threads
non-blocking I/O
reduced context switching
efficient memory usage
Who uses Netty?

Many popular systems:

Spring Cloud Gateway
Apache Kafka
Discord backend
gRPC
Elasticsearch
What is WebFlux?

Spring WebFlux is Spring’s reactive web framework.

Introduced as alternative to:

Spring MVC

Built on:

Reactive Streams
Reactor
Netty
Why WebFlux?

Spring MVC uses:

thread-per-request model

Good for moderate traffic.

But:

high concurrency
many slow I/O calls

can exhaust threads.

WebFlux solves scalability issues.