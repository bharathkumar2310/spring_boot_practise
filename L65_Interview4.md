1. What is Resilience4j?

Answer:

Resilience4j is a lightweight fault tolerance library for Java microservices.

It helps applications:

handle failures gracefully
prevent cascading failures
improve system stability

It provides:

Circuit Breaker
Retry
Rate Limiter
Bulkhead
Time Limiter
Cache

It is commonly used with:

Spring Boot
RestTemplate
WebClient
Feign Client
2. Why do we need Resilience4j?
   Problem

In microservices:

one service depends on another
network failures happen
slow services can block threads
downstream failures can crash the entire system

Example:

Order Service calls Payment Service
Payment Service becomes slow
all Order threads wait
thread pool exhausts
complete outage happens

Resilience4j prevents this.

3. What is Circuit Breaker?

Answer:

Circuit Breaker prevents continuously calling a failing service.

Instead of:

repeatedly making failing calls

it:

stops calls temporarily
gives failing service time to recover
4. Circuit Breaker States
   CLOSED

Normal state.
Calls are allowed.

If failures cross threshold:
→ OPEN

OPEN

Requests fail immediately.

No actual downstream call happens.

After wait duration:
→ HALF_OPEN

HALF_OPEN

Allows limited test requests.

If successful:
→ CLOSED

If failures continue:
→ OPEN

Real Interview Follow-up
Why not simply retry forever?

Because:

retries increase load
already failing service becomes worse
thread exhaustion may happen

Circuit breaker prevents cascading failure.

5. Real-Time Example of Circuit Breaker
   Scenario

E-commerce app:

Product Service
Payment Service

Payment DB goes down.

Without CB:

thousands of requests hit payment service
thread pools exhaust
whole system slows down

With CB:

calls stop temporarily
fallback response returned
6. Difference Between Retry and Circuit Breaker
   Retry	Circuit Breaker
   Tries again	Stops calls
   Handles temporary failures	Handles persistent failures
   Increases requests	Reduces requests
   Good for transient network issues	Good for service outages
   Real Interview Question
   When should Retry NOT be used?

Answer:

when downstream service is already overloaded
retries can amplify traffic
may create retry storms

Example:
DB already slow → retries make it worse.

7. What is Bulkhead?

Bulkhead isolates resources.

Inspired from ships:

if one compartment floods
entire ship should not sink

In microservices:

one failing dependency should not consume all threads/resources
8. Types of Bulkhead
   Semaphore Bulkhead

Limits concurrent calls using permits.

Example:
only 10 concurrent calls allowed.

Extra requests:
→ rejected immediately

Lightweight.

ThreadPool Bulkhead

Uses separate thread pool + queue.

Each external service gets isolated thread pool.

Real Interview Question
Difference Between Semaphore and ThreadPool Bulkhead
Semaphore	ThreadPool
Uses current thread	Uses separate thread pool
Lightweight	More isolation
No async boundary	Async execution
Lower overhead	Higher overhead
9. Why do we need both Bulkhead and Circuit Breaker?

Very common interview question.

Circuit Breaker

Protects from:

repeated failures
Bulkhead

Protects from:

resource exhaustion

Even if CB exists:

many slow requests can still occupy threads

Bulkhead limits resource usage.

10. What problem does Bulkhead solve that Circuit Breaker does not?

Circuit breaker:

detects failures

Bulkhead:

isolates resources

Example:
service is slow but not failing.

CB may remain CLOSED.

But:

threads become blocked

Bulkhead prevents thread exhaustion.

11. What is Rate Limiter?

Controls number of requests allowed in a time period.

Example:

only 100 requests/sec

Used to:

prevent abuse
avoid overload
protect APIs
Real Scenario

Public payment API:

prevent DDOS
prevent abusive clients
12. What is TimeLimiter?

Limits maximum execution time.

If service takes too long:

timeout occurs

Prevents:

threads waiting forever


Real Interview Question
Difference Between TimeLimiter and Circuit Breaker
TimeLimiter	Circuit Breaker
Single request timeout	Overall failure monitoring
Per-call protection	System-wide protection
Stops long waits	Stops repeated failing calls
13. What is Fallback?

Alternative response when failure occurs.

Example:

@CircuitBreaker(name="payment", fallbackMethod="fallback")

If payment service fails:

public String fallback(Exception ex) {
return "Payment service unavailable";
}
Real Interview Question
Should fallback always return success?

No.

Bad fallback can hide real problems.

Fallback should:

degrade gracefully
provide meaningful response
14. What metrics does Circuit Breaker monitor?
    Failure rate
    Slow call rate
    Number of calls
    Sliding window
    Success count
15. What is Sliding Window?

Circuit breaker checks failures over recent calls.

Two types:

Count based
Time based

Example:
last 10 calls.

If 5 fail:
50% failure rate.

16. What is Failure Rate Threshold?

Percentage of failures needed to OPEN circuit.

Example:

failureRateThreshold: 50

If >50% fail:
→ OPEN

17. What is Slow Call Rate Threshold?

Even successful calls can be slow.

If too many slow calls happen:
CB may open proactively.

Very important production concept.

18. What is Wait Duration in Open State?

How long CB stays OPEN before HALF_OPEN testing.

Example:

waitDurationInOpenState: 10s
19. What happens in HALF_OPEN state?

Limited requests allowed.

Purpose:

test service recovery

Success:
→ CLOSED

Failure:
→ OPEN again

20. Can Circuit Breaker prevent resource exhaustion?

Partially.

But:

slow calls before CB opens can still consume threads.

That is why Bulkhead is still needed.

21. Why is Retry dangerous?

Common senior-level question.

Problems:

retry storms
duplicate requests
overload amplification
cascading failures

Especially dangerous:

distributed systems
22. What operations should NOT be retried?

Non-idempotent operations.

Example:

payment deduction
money transfer

Retry may:

duplicate transaction
23. What is Idempotency?

Multiple identical requests produce same result.

Examples:

GET → idempotent
DELETE → usually idempotent
POST payment → often NOT idempotent
24. How does Resilience4j integrate with Spring Boot?

Using:

annotations
AOP proxies

Example:

@CircuitBreaker(name="payment")

Spring creates proxy around method.

25. Common Resilience4j Annotations
    @CircuitBreaker
    @Retry
    @RateLimiter
    @Bulkhead
    @TimeLimiter
26. Real Interview Scenario
    “Payment service is slow but not failing. What would you use?”

Best answer:

TimeLimiter
Bulkhead

Circuit breaker alone may not help immediately.

27. Real Interview Scenario
    “Service is temporarily failing due to network glitch.”

Use:

Retry

Because failure is transient.

28. Real Interview Scenario
    “External API is completely down.”

Use:

Circuit Breaker
Fallback
29. Real Interview Scenario
    “One downstream service should not consume all threads.”

Use:

Bulkhead
30. Real Interview Scenario
    “Public API should prevent abuse.”

Use:

Rate Limiter
31. Difference Between Hystrix and Resilience4j
    Hystrix	Resilience4j
    Netflix library	Modern lightweight library
    Maintenance mode	Actively maintained
    Heavy	Lightweight
    Thread isolation default	Modular
    Older architecture	Functional style
32. Why did Netflix Hystrix become less popular?
    maintenance stopped
    heavy thread model
    reactive ecosystem evolved

Resilience4j became preferred.

33. Is Resilience4j synchronous or asynchronous?

Supports both:

sync
async
reactive

Works with:

CompletableFuture
Reactor
WebFlux
34. Can multiple Resilience4j patterns be combined?

Yes.

Very common.

Example:

Retry + CircuitBreaker + Bulkhead + TimeLimiter
35. Correct Order of Patterns

Important interview question.

Usually:

RateLimiter
→ Bulkhead
→ TimeLimiter
→ Retry
→ CircuitBreaker

Reason:

protect resources first
retries inside safe boundaries
CB observes final failures
36. What is Cascading Failure?

Failure spreading across services.

Example:

DB slow
service threads block
upstream services timeout
entire system affected

Resilience patterns prevent this.

37. Why are timeouts critical in microservices?

Without timeout:

threads wait indefinitely
resource exhaustion occurs

Always configure:

connect timeout
read timeout
38. Interview Trap Question
    “Can Circuit Breaker replace Retry?”

No.

They solve different problems.

Retry:

temporary failures

Circuit breaker:

persistent failures
39. Interview Trap Question
    “Can Bulkhead replace Circuit Breaker?”

No.

Bulkhead:

limits resources

Circuit breaker:

stops repeated failing calls
40. Most Important Real-World Combination

Very commonly used:

Feign Client
+ Retry
+ CircuitBreaker
+ Fallback
+ Timeouts

because external service calls are unreliable.


41. How does Circuit Breaker decide to OPEN?

Answer:

Circuit breaker calculates:

failure rate
slow call rate

over a sliding window.

If thresholds exceed configured values:

→ circuit becomes OPEN.

Example:

failureRateThreshold: 50
minimumNumberOfCalls: 10

If:

at least 10 calls happen
more than 50% fail

→ OPEN state triggered.

42. What is minimumNumberOfCalls?

Answer:

Circuit breaker will not calculate failure rate until minimum calls are reached.

Example:

minimumNumberOfCalls: 10

Even if first 2 requests fail:

CB will not OPEN yet.

Purpose:

avoid opening circuit too early.

43. Difference Between Count-Based and Time-Based Sliding Window

Count-Based

Checks last N requests.

Example:

last 20 calls.

Time-Based

Checks requests in recent duration.

Example:

last 1 minute.

Interview Point:

Count-based is predictable.
Time-based works better for variable traffic.

44. What is Automatic Transition from OPEN to HALF_OPEN?

Answer:

Resilience4j can automatically move:

OPEN → HALF_OPEN

after wait duration expires.

Config:

automaticTransitionFromOpenToHalfOpenEnabled: true

Otherwise:

transition happens only when new request arrives.

45. What is permittedNumberOfCallsInHalfOpenState?

Answer:

Controls test requests allowed in HALF_OPEN state.

Example:

permittedNumberOfCallsInHalfOpenState: 3

Only 3 trial requests allowed.

If successful:
→ CLOSED

If failures happen:
→ OPEN again.

46. What is the difference between failure and slow call?

Failure

Exception/error occurred.

Slow Call

Request succeeded but exceeded time threshold.

Example:

Response took 8 seconds.

Even success may indicate unhealthy service.

47. What exceptions should NOT trigger Circuit Breaker?

Answer:

Business exceptions.

Example:

InvalidUserInputException
ValidationException

These are expected failures.

Only technical/system failures should affect CB.

Example:

ignoreExceptions:
- com.example.ValidationException
48. Difference Between Fallback and Retry

Retry

Attempts same operation again.

Fallback

Returns alternative response.

Example:

Retry:
try payment again.

Fallback:
show “Payment temporarily unavailable”.

49. Can fallback itself fail?

Answer:

Yes.

Fallback is normal code.

If fallback throws exception:

request still fails.

Interview Follow-up:

Fallback should be lightweight and reliable.

50. What is Retry Storm?

Answer:

Massive retries from multiple services simultaneously.

Example:

Service A retries 3 times
Service B retries 3 times
Service C retries 3 times

Traffic explosion occurs.

This can crash already overloaded systems.

51. What is Exponential Backoff in Retry?

Answer:

Delay increases after each retry.

Example:

1s → 2s → 4s → 8s

Prevents aggressive retry traffic.

Very important production practice.

52. Why is fixed retry dangerous?

Answer:

All clients retry at same intervals.

This creates synchronized traffic spikes.

Called:

Thundering Herd Problem.

53. What is Randomized Retry/Jitter?

Answer:

Adds randomness to retry delays.

Example:

2.1s
1.7s
2.8s

Prevents retry synchronization.

Common production best practice.

54. What happens if TimeLimiter timeout occurs?

Answer:

Request is interrupted/cancelled.

Timeout exception thrown.

Example:

TimeoutException

Usually combined with:

fallback method.

55. Difference Between Connection Timeout and Read Timeout

Connection Timeout

Time to establish connection.

Read Timeout

Time waiting for response data.

Both are critical.

56. Why should TimeLimiter be used with async calls?

Answer:

TimeLimiter works best with:

CompletableFuture

because synchronous blocking calls cannot always be interrupted properly.

57. How does Resilience4j work internally in Spring Boot?

Answer:

Uses:

AOP proxies
decorators

Spring creates proxy around annotated methods.

Example:

@CircuitBreaker(name="payment")

Actual method call goes through proxy.

58. Why might Resilience4j annotations NOT work?

Very common interview question.

Answer:

Because of self-invocation.

Example:

this.callMethod();

Internal method calls bypass Spring proxy.

So annotations won't trigger.

59. What is self-invocation problem in Spring AOP?

Answer:

Proxy intercepts only external calls.

Internal calls inside same class bypass proxy.

Common issue with:

@Transactional
@Retry
@CircuitBreaker
60. How to monitor Resilience4j in production?

Answer:

Using:

Spring Boot Actuator
Micrometer
Prometheus
Grafana

Endpoints:

/actuator/health
/actuator/metrics
/actuator/circuitbreakers
61. Important Circuit Breaker Metrics

Answer:

successful calls
failed calls
slow calls
rejected calls
state transitions
62. What is Bulkhead Full Exception?

Answer:

Occurs when bulkhead limit exceeded.

Example:

BulkheadFullException

Means:

too many concurrent requests already running.

63. What happens to requests rejected by Bulkhead?

Answer:

They fail immediately.

Usually handled using fallback.

Purpose:

protect application resources.

64. What is Thread Starvation?

Answer:

All worker threads become blocked waiting for slow operations.

New requests cannot execute.

Causes:

slow DB
slow APIs
missing timeouts
65. Why are thread pools important in microservices?

Answer:

Threads are limited resources.

If threads block:

application throughput collapses.

Bulkhead protects thread pools.

66. Can Retry and Circuit Breaker conflict?

Answer:

Yes.

Bad retry configuration may delay CB opening.

Too many retries can:

increase failures
overload service
distort metrics

Careful tuning required.

67. Should Retry happen before or after Circuit Breaker?

Common answer:

Retry inside Circuit Breaker.

Reason:

CB should observe final outcome after retries.

68. Why should retries be limited?

Answer:

Unlimited retries can cause:

infinite load
thread exhaustion
cascading failures

Always configure:

max attempts
backoff
timeout
69. What is Fail Fast?

Answer:

Reject request immediately instead of waiting.

Example:

OPEN circuit breaker.

Purpose:

save resources.

70. What is Graceful Degradation?

Answer:

Application provides reduced functionality instead of complete failure.

Examples:

cached data
default response
partial response
71. Can Circuit Breaker work without fallback?

Answer:

Yes.

Fallback is optional.

Without fallback:

exception returned immediately.

72. What is the biggest mistake when using Retry?

Answer:

Retrying non-idempotent operations.

Example:

payment processing.

Can create duplicate transactions.

73. What HTTP status codes are usually retriable?

Common retriable:

408
429
500
502
503
504

Usually NOT retried:

400
401
403
404
74. Why should 404 generally not be retried?

Answer:

404 usually means:

resource does not exist.

Retrying won't help.

75. What is the difference between synchronous and reactive resilience?

Synchronous

Uses blocking threads.

Reactive

Uses non-blocking async streams.

Resilience4j supports:

Reactor
WebFlux
CompletableFuture
76. How does Resilience4j integrate with Feign Client?

Answer:

Using:

Spring Cloud CircuitBreaker
Feign fallback/fallbackFactory

Common in microservices.

77. Difference Between Fallback and FallbackFactory in Feign

Fallback

Static fallback response.

FallbackFactory

Gets actual exception cause.

More flexible debugging.

78. What is a good production resilience strategy?

Typical answer:

timeout first
retry only transient failures
circuit breaker for persistent failures
bulkhead isolation
rate limiting
monitoring/alerts
79. What are common mistakes in Resilience4j configuration?

Answer:

too many retries
no timeout configured
retrying all exceptions
missing fallback logging
wrong CB thresholds
huge wait durations
ignoring monitoring
80. Senior-Level Architecture Question

“How would you protect a payment microservice?”

Strong answer:

Use:

timeout
retry for network glitches
circuit breaker
bulkhead isolation
idempotency keys
rate limiting
monitoring
fallback for non-critical operations

Avoid retrying actual payment deduction blindly.