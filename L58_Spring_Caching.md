CACHING :

    Caching is a technique where we store frequently used data temporarily in a fast-access storage so that future requests can be served quickly instead of recomputing or fetching the data again.

In simple terms:

    “Instead of doing expensive work again and again, save the result and reuse it.”

Benefits of Caching

1. Faster Response Time

        Cache access is extremely fast compared to DB/API calls.
        DB call → maybe 50ms
        Cache call → maybe 1ms

2. Reduced Database Load

       Thousands of repeated queries are avoided.
       This improves scalability.

3. Better User Experience

       Pages load faster.
       APIs respond quickly.

4. Reduced Network Calls

       Useful when calling external services.
    
       Example:
    
       payment exchange rates
       weather APIs
       product catalog

SPRING CACHE :



Step 1: Add Dependency


add:
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>

Step 2: Enable Caching
    
    @SpringBootApplication
    @EnableCaching
    public class DemoApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }

This enables Spring cache support.

Step 3: Create Service
    
    @Service
    public class UserService {
    
        @Cacheable("users")
        public String getUser(Long id) {
    
            System.out.println("Fetching from DB...");
    
            simulateSlowCall();
    
            return "User-" + id;
        }
    
        private void simulateSlowCall() {
            try {
                Thread.sleep(3000);
            } catch (Exception e) {
            }
        }
    }

What @Cacheable("users") Means

    Cache name = users
    Spring checks cache before method execution

Flow:
    
    First call  -> executes method -> stores result
    Second call -> returns from cache

Step 4: Create Controller
    
    @RestController
    @RequestMapping("/users")
    public class UserController {
    
        @Autowired
        private UserService userService;
    
        @GetMapping("/{id}")
        public String getUser(@PathVariable Long id) {
            return userService.getUser(id);
        }
    }

Step 5: Run Application

Call:GET /users/1

First request:

    takes 3 seconds
    prints:
    Fetching from DB...

Second request:
        
        instant response
        no DB print

Because returned from cache.

----------------------------------------------------------------------------------------------------------------------------------

| Annotation    | Purpose         | Method Executes?   | Cache Action   |
| ------------- | --------------- | ------------------ | -------------- |
| `@Cacheable`  | Read from cache | Only on cache miss | Stores result  |
| `@CachePut`   | Update cache    | Always             | Replaces cache |
| `@CacheEvict` | Remove cache    | Usually yes        | Deletes cache  |

1. @Cacheable

        Used to: Read from cache first.
        
        If cache exists:method NOT executed
        
        If cache missing:
        
            execute method
            store result in cache

2. @CachePut

       Used to: Always execute method AND update cache. 
       Unlike @Cacheable:
    
           method ALWAYS runs
           cache ALWAYS updated

3. @CacheEvict

       Used to:Remove data from cache.
       Usually after:
    
           delete
           update
           invalidation




---------------------------------------------------------------------------------------------------------------------------

Custom Cache Key

    Default: method parameters used as key

Custom key:
    
    @Cacheable(value = "users", key = "#id")
    public String getUser(Long id)

Multiple Parameters Example
    
    @Cacheable(value = "users", key = "#id + '-' + #name")
    public String getUser(Long id, String name)

Generated key: 1-Bharath

-------------------------------------------------------------------------------------------------------------------------------------

If you only enable caching and do not configure anything:

    @EnableCaching

    Spring Boot automatically chooses a cache provider using auto-configuration.
    Default Cache in Spring Boot
    Usually the default is: ConcurrentMapCache
    managed by:ConcurrentMapCacheManager
    Internally it uses:ConcurrentHashMap
    
    So by default:
    
    cache is in-memory
    stored inside application JVM
    not distributed
    lost on restart
    Internal Structure
    
    Something like this internally:
    
        Map<String, Map<Object, Object>>
    
    Example:
        
        users
        └── 1 -> User Object
        └── 2 -> User Object

Example
    
    @Cacheable("users")
    public User getUser(Long id) {
    return repository.findById(id).get();
    }

    First call:DB hit
    Second call:ConcurrentHashMap lookup

----------------------------------------------------------------------------------------------------------------------------------

Important Production Configurations
1. TTL

        Without TTL:stale data risk

2. Serialization

       Prefer:JSON serializers
    
       Avoid:
    
       default JDK serialization

3. Max Size

       Prevent memory explosion.

4. Eviction Policies

       Example:
    
       LRU
       LFU

ConcurrentMapCacheManager (default cache) does NOT support:

TTL
max size
eviction policies

So for production-grade in-memory cache, use Caffeine.


-------------------------------------------------------------------------------------------------------------------------------



High Level Architecture

        Client
        ↓
        Proxy Object
        ↓
        CacheInterceptor
        ↓
        CacheAspectSupport
        ↓
        CacheResolver
        ↓
        CacheManager
        ↓
        Cache Implementation
        ↓
        Actual Method


Step-by-Step Internal Flow

Step 1: @EnableCaching

    @EnableCaching

    This creates bean for Caching Interceptor which contains the actual logic for fetching the cache

Step 2: Spring Scans @Cacheable

    Spring detects:@Cacheable

    Internal Object Created
    Something like: CacheableOperation

    containing:

        cache name
        key
        condition
        unless
        sync
        etc.    


Step 3: Proxy Creation

    Spring AOP creates proxy around bean.
    If interface exists:
    
        JDK Dynamic Proxy
    
    Else:
    
        CGLIB Proxy


Step 4: Method Invocation

    When:userService.getUser(1L);
    is called:Proxy intercepts.

    Then delegates to:CacheInterceptor



1. CacheInterceptor


    1. Reads cache metadata
    2. Generates key
    3. Checks cache
    4. Decides hit/miss
    5. Executes method if needed
    6. Stores result

2. CacheManager

        Default manager:ConcurrentMapCacheManager
        
        Or:
        
        RedisCacheManager
        CaffeineCacheManager

        Create and retrieve cache instances



If you dont want defult cache u can use 

Redis, ECache, Caffine etc


    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>



    spring.data.redis.host=localhost
    spring.data.redis.port=6379


    Now Spring automatically uses:RedisCacheManager  instead of: ConcurrentMapCacheManager

Verify Which Cache Is Used
    
    @Autowired
    CacheManager cacheManager;
    
    @PostConstruct
    public void test() {
    System.out.println(cacheManager.getClass());
    }

Output:

    class org.springframework.data.redis.cache.RedisCacheManager


Configure TTL (Very Important)

Without TTL:Cache lives forever

Configure:

    @Configuration
    public class RedisConfig {
    
        @Bean
        public RedisCacheConfiguration cacheConfiguration() {
    
            return RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(10));
        }
    }

Now entries expire after 10 mins.



Configure JSON Serialization

Default Redis serialization is ugly binary.

Better use JSON.

Redis Config
    
    @Configuration
    public class RedisConfig {
    
        @Bean
        public RedisCacheConfiguration cacheConfiguration() {
    
            return RedisCacheConfiguration.defaultCacheConfig()
                    .serializeValuesWith(
                        RedisSerializationContext.SerializationPair
                            .fromSerializer(new GenericJackson2JsonRedisSerializer())
                    )
                    .entryTtl(Duration.ofMinutes(10));
        }
    }



--------------------------------------------------------------------------------------------------------------------------------

MULTIPLE CACHE :

    @Bean
    public CacheManager redisCacheManager() {
    }
    
    @Bean
    public CacheManager caffeineCacheManager() {
    }
    
    
    @Cacheable(value = "users",
    cacheManager = "redisCacheManager")


------------------------------------------------------------------------------------------------------------------------



| Cache                          | Type                         | Best For                     |
| ------------------------------ | ---------------------------- | ---------------------------- |
| Default (`ConcurrentMapCache`) | Simple local JVM cache       | Learning/small apps          |
| Caffeine                       | High-performance local cache | Ultra-fast local reads       |
| Redis                          | Distributed cache            | Scalable distributed systems |


1. Default Cache (ConcurrentMapCache)
    
            Internally uses:ConcurrentHashMap
            inside JVM memory.
        
        When Used
        
        Mostly:
        
        learning
        testing
        very small apps
        prototypes
        Why Used
        
        Because:
        
        zero setup
        built into Spring
        easiest to start

Advantages

| Advantage       | Why               |
| --------------- | ----------------- |
| Very simple     | No external setup |
| Fast            | In-memory         |
| No dependencies | Built-in          |


Problems

| Problem         | Why                    |
| --------------- | ---------------------- |
| No TTL          | stale data             |
| No eviction     | memory growth          |
| Not distributed | each app has own cache |
| Lost on restart | JVM memory only        |



2. Caffeine Cache

        High-performance in-memory cache.
        Still local JVM cache, but much more advanced.

When Used

Used when:
    
    single service instance
    ultra-low latency needed
    local memory cache preferred
    want TTL + eviction
    Why Used

Because:
    
    faster than Redis
    no network calls
    production-grade local caching
    advanced eviction algorithms

Advantages

| Advantage        | Why                  |
| ---------------- | -------------------- |
| Extremely fast   | Local memory         |
| TTL support      | auto expiry          |
| Max size         | prevents OOM         |
| Smart eviction   | efficient memory use |
| Production ready | robust algorithms    |


Problems

| Problem             | Why                  |
| ------------------- | -------------------- |
| Not shared          | each JVM separate    |
| Lost on restart     | memory only          |
| Cache inconsistency | multi-instance issue |


3. Redis Cache

Distributed centralized cache server.

When Used

Used when:
    
    multiple app instances
    microservices
    distributed systems
    shared cache needed

ADVANTAGES:

| Advantage    | Why                     |
| ------------ | ----------------------- |
| Shared cache | all apps use same cache |
| Distributed  | works across servers    |
| TTL support  | built-in                |
| Persistence  | optional                |
| Scalable     | cluster support         |


PROBLEMS:

| Problem                | Why                      |
| ---------------------- | ------------------------ |
| Network call needed    | slower than local memory |
| Extra infrastructure   | Redis server needed      |
| Serialization overhead | object conversion        |
| Operational complexity | monitoring/clustering    |

Caffine efficiency over default

1. Better Concurrency

       ConcurrentHashMap still has contention under heavy load.

Caffeine minimizes:
    
    locking
    contention
    synchronization overhead

using advanced lock-free techniques.

Example

    Thousands of threads reading/writing simultaneously.
    Caffeine scales better under concurrency.

2. Efficient Eviction Algorithms

    Default cache:no eviction
    Caffeine:optimized eviction

Uses:

    Window TinyLFU
    Very advanced caching algorithm.

It keeps:
    
    frequently used data
    evicts less useful entries intelligently
    
    Much better hit rate.

Caffeine

    Caffeine is:
    
    a Java library
    runs inside your application
    local



REDIS over other caches

    IT provides distributed caching i.e if there are 3 instances of a cache redis help in providing 
    a single cache for all 3 instances


Caffeine

RAM belongs to:

    Single JVM
    Redis

RAM belongs to:

    Dedicated Redis Server
    shared by all apps.


--------------------------------------------------------------------------------------------------------------------------------


1. Self Invocation Problem (VERY IMPORTANT)

        This is one of the most asked Spring Cache interview questions.
        
        Problem:
        
        @Service
        public class UserService {
        
            @Cacheable("users")
            public String getUser(Long id) {
                return "USER";
            }
        
            public void test() {
                getUser(1L); // caching WILL NOT work
            }
        }
        
        Why?
        
        Because Spring caching works using proxies.
        
        Internal method calls bypass proxy.
        
        Only external calls go through proxy.
        
        Interview One-Liner:
        
        Spring cache annotations work only when methods are invoked through Spring proxy.
        
        Solution:
        
        Call from another bean
        Use self-injection
        Use AspectJ mode

2. Cache Stampede / Cache Penetration

          Very important in system design interviews.
    
          Problem:
    
          When cache expires:
    
          Thousands of requests hit DB simultaneously.
    
          This is called:
    
          Cache Stampede
          Dogpile Effect
    
          Solution:
    
          @Cacheable(value = "users", sync = true)
    
          sync=true ensures:
    
          only one thread loads cache
          others wait
    
          Good interview point.

       Why Needed?
    
       Suppose cache expires.
    
       1000 requests come simultaneously.
    
       Without sync:
    
       all 1000 hit DB
    
       With sync:
    
       only ONE thread loads data
       others wait
    
       Prevents:
    
       cache stampede
       DB overload
    
       Very important production concept.

3. condition and unless

       Frequently asked.
    
       condition
    
       Cache only if condition is true.
    
       @Cacheable(value = "users", condition = "#id > 10")
       unless
    
       Do NOT cache result if condition true.
    
       @Cacheable(value = "users", unless = "#result == null")
    
       Meaning:
    
       execute method
       but don't cache null result
    
       Very important.

4. Cache Eviction Strategies

       You mentioned LRU/LFU but add examples.
    
       LRU
    
       Least Recently Used removed first.
    
       LFU
    
       Least Frequently Used removed first.
    
       Caffeine uses:
    
       Window TinyLFU
    
       Very good advanced interview point.

5. Write Through vs Write Back vs Cache Aside


    VERY important for senior interviews.
    
    Cache Aside (most common in Spring)
    
    Flow:
    
    App → Cache
    Miss → DB
    Then put into cache
    
    Spring @Cacheable mainly follows this.
    
    Write Through
    
    Write DB + cache together.
    
    Write Back
    
    Write only cache first.
    DB updated asynchronously later.
    
    Used for high performance systems.

6. Distributed Cache Consistency Problem

        Very important.
        
        Problem:
        
        Instance A updates DB
        Instance B still has stale local cache
        
        This happens with:
        
        Caffeine
        local JVM cache
        
        Redis solves this better because centralized cache.

12. allEntries = true

           Very commonly asked.
        
           @CacheEvict(value = "users", allEntries = true)
        
           Clears entire cache.

13. beforeInvocation
    
                  @CacheEvict(value="users", beforeInvocation=true)
    
       Evicts cache before method execution.
    
       Useful when method may fail.

 4. Transaction + Cache Consistency

        Very important senior-level topic.
    
        Example:
    
        DB update succeeds
        Cache update fails
    
        or vice versa.
    
        Leads to:
    
        inconsistent state
    
        Mention:
    
        ordering matters
        usually evict cache after transaction commit