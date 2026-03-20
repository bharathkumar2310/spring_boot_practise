FIRST LEVEL CACHE (Persistence Context Cache)

    - Each EntityManager has a first-level cache (also called the persistence context cache).
      - When you retrieve an entity, it is stored in the first-level cache.
      - If you retrieve the same entity again within the same EntityManager, it will be returned from the cache, not from the database.
      - This cache is transactional and is cleared when the transaction ends.
      - It is also cleared when you call `entityManager.clear()` or `entityManager.close()`.

1️⃣ What is Persistence Context?

A Persistence Context is a cache (memory area) where JPA stores and manages entity objects.

It keeps track of:

        Entities fetched from the database
        Changes made to those entities
        New entities that must be inserted
        Entities that must be deleted

It is basically a map of entities managed by JPA.

Conceptually:

Persistence Context
|
|---- Entity Object 1 (User id=1)
|---- Entity Object 2 (Order id=10)
|---- Entity Object 3 (Product id=5)

All entities inside it are called Managed Entities.

2️⃣ Who creates the Persistence Context?

    The Persistence Context is created by the
    Hibernate or any Java Persistence API implementation.

    It is associated with the EntityManager.

Example:

    EntityManager em = entityManagerFactory.createEntityManager();

When EntityManager is created → Persistence Context is also created.

3️⃣ Why Persistence Context is Needed

Without it, ORM frameworks would be inefficient and inconsistent.

Main reasons:

1️⃣ Avoid Multiple DB Calls (First Level Cache)

If you fetch the same entity twice, the second time it comes from memory.

Example:

        User u1 = em.find(User.class, 1);
        User u2 = em.find(User.class, 1);
        
        Without persistence context:
        
        SELECT * FROM user WHERE id=1
        SELECT * FROM user WHERE id=1
        
        With persistence context:
        
        SELECT * FROM user WHERE id=1
        (no second query)
        
        Because it already exists in the persistence context.

2️⃣ Automatic Change Detection (Dirty Checking)

JPA automatically detects changes and updates DB.

Example:

        User user = em.find(User.class, 1);
        user.setName("Bharath");
        
        You never wrote:
        
        update user set name='Bharath'
        
        But when transaction commits:
        
        UPDATE user SET name='Bharath' WHERE id=1
        
        This happens because persistence context tracks object changes.

3️⃣ Ensures Only One Object per Row

Persistence Context guarantees:

One DB Row → One Java Object

Example:

    User u1 = em.find(User.class, 1);
    User u2 = em.find(User.class, 1);
    u1 == u2  → true
    
    Both references point to the same object.
    
    Without persistence context you could get multiple objects for the same row.

4️⃣ Batch Write Optimization

Instead of executing SQL immediately, Hibernate stores changes in the persistence context and executes them at transaction commit.

Example:

    user.setName("Bharath");
    order.setAmount(500);
    product.setPrice(100);
    
    Hibernate delays SQL until commit.
    
    At commit:
    
    UPDATE user...
    UPDATE order...
    UPDATE product...
    
    This improves performance.

4️⃣ Lifecycle of Entities in Persistence Context

Entities have 4 states:

1️⃣ Transient

    Object created but not saved.
    User user = new User();
    Not in persistence context.

2️⃣ Managed

    After persist() or find().
    User user = em.find(User.class,1);
    Now entity is inside persistence context.

Changes will be tracked.

3️⃣ Detached

When persistence context closes.

    em.close();

Entity exists but not managed anymore.

4️⃣ Removed

    Marked for deletion.
    
    em.remove(user);
    
    Deleted at commit.

5️⃣ Internal Structure (Important for Interviews)

Persistence context internally acts like:

Map<EntityClass + PrimaryKey, EntityObject>

Example:

Map
---------------------------------
(User,1)     → User Object
(Order,10)   → Order Object
(Product,5)  → Product Object

This ensures no duplicate objects.



USES OF PERSISTENCE CONTEXT :

    1. Caching: It serves as a first-level cache, improving performance by avoiding redundant database queries.
       2. Change Tracking: It tracks changes to entities, enabling automatic updates to the database when transactions commit.
       3. Identity Management: It ensures that each database row corresponds to a single Java object, maintaining consistency.
       4. Batch Operations: It optimizes database interactions by batching SQL statements at transaction commit, reducing overhead.
       5. Transaction Management: It manages the lifecycle of entities within a transaction, ensuring proper handling of entity states (transient, managed, detached, removed).

       Here batching means SQL statements are delayed and executed together during flush/commit instead of immediately.
       The persistence context collects all entity changes first, then sends SQL to the database in a group.


![img.png](Images/FLC1.png)

![img_1.png](Images/FLC2.png)

![img_2.png](Images/FLC3.png)

![img_3.png](Images/FLC4.png)

![img_4.png](Images/FLC5.png)

![img_5.png](Images/FLC6.png)

![img_6.png](Images/FLC7.png)

![img.png](Images/FLC13.png)

![img_1.png](Images/FLC14.png)



1. Persistence Context Scope
   Rule
   1 Transaction → 1 EntityManager → 1 Persistence Context

When a transaction ends:

Persistence Context is destroyed
Entities become DETACHED
2. When @Transactional is NOT Used

Each repository method runs in its own transaction.

Example:

public void process() {

    repo.save(user);

    user.setName("B");

    repo.findAll();
}

Execution:

save() → Transaction 1
findAll() → Transaction 2

Each transaction has a different persistence context.

3. Example 1 — Save Then Modify (Without Second Save)
   Code
   public void process() {

   User user = new User();
   user.setName("A");

   repo.save(user);

   user.setName("B");
   }
   Execution

Transaction 1:

persist(user)
INSERT INTO user (name='A')
commit

After commit:

entity becomes DETACHED

Modification:

user.setName("B")

This only changes the Java object in memory.

Database Result
name = A

NOT "B".

4. Example 2 — Save → Modify → Save Again
   Code
   public void process() {

   User user = new User();
   user.setName("A");

   repo.save(user);

   user.setName("B");

   repo.save(user);
   }
   Execution

Transaction 1

INSERT INTO user (name='A')

Transaction 2

merge(user)
UPDATE user SET name='B'
Database Result
name = B

Reason:

entity has ID → Hibernate performs merge()
5. Example 3 — Save → Modify → FindAll
   Code
   public void process() {

   User user = new User();
   user.setName("A");

   repo.save(user);

   user.setName("B");

   repo.findAll();
   }
   Execution

Transaction 1

INSERT user A
commit

Entity becomes DETACHED.

Modification:

user.setName("B")

Transaction 2

SELECT * FROM user
Result

Database:

name = A

findAll() returns:

User{name='A'}

NOT "B".

Reason:

Persistence Context 2 loads data from database
not from detached object
6. What Happens During merge()

If a detached entity is saved again:

repo.save(user);

Hibernate internally does:

merge(detachedEntity)

Steps:

1. Check entity ID
2. Load current row from DB
3. Create managed entity in new PC
4. Copy state from detached entity
5. Perform dirty checking
6. Generate UPDATE

Example:

Detached entity:

User{id=1, name='B'}

DB row:

User{id=1, name='A'}

After merge:

UPDATE user SET name='B'
7. Entity State Lifecycle
   NEW (Transient)
   ↓
   persist()
   ↓
   MANAGED
   ↓
   transaction commit
   ↓
   DETACHED

If you call merge:

DETACHED
↓
merge()
↓
MANAGED (new persistence context)
8. Key Interview Rules
   Rule 1
   Persistence Context is transaction scoped
   Rule 2
   After transaction commit → entities become DETACHED
   Rule 3

Detached entity changes are not tracked.

Rule 4
save(detached entity) → merge() → UPDATE
Rule 5
findAll() always reads from database

unless inside the same persistence context.

9. Quick Comparison
   Scenario	Result
   save → modify	change lost
   save → modify → save	UPDATE
   save → modify → findAll	returns old DB value
   transactional method	changes auto persisted

----------------------------------------------------------------------------------------------------------------------------------


L2 Cache (Second Level Cache)

    - The second-level cache is an optional cache that can be configured in JPA.
      - It is shared across multiple EntityManager instances and transactions.
      - It can cache entities, collections, and query results.
      - It is typically used to improve performance by reducing database access for frequently accessed data.
      - It is not transactional and must be explicitly configured and managed by the developer.

2️⃣ Second-Level Cache (L2 Cache)

L2 Cache is a cache shared across multiple sessions.

Scope:

    Application Level

Architecture:

Session 1
|
|---- L1 Cache
|
Session 2
|
|---- L1 Cache
|
v
L2 Cache
|
v
DB

Flow:

        Session -> L1 Cache -> L2 Cache -> Database
        Example Flow
        First request
        User user = session1.get(User.class, 1);

Steps:

    1. Check L1 cache → not found
       2. Check L2 cache → not found
       3. Query DB
       4. Store in L1 + L2 cache
          Second request (different session)
          User user = session2.get(User.class, 1);

Steps:

    1. L1 cache → not found
       2. L2 cache → found
       3. No DB query

So DB load reduces drastically.

Why L2 Cache is Needed
1️⃣ Reduce Database Queries

Without L2:

        Session1 -> DB
        Session2 -> DB
        Session3 -> DB
        Session4 -> DB

With L2:

        Session1 -> DB
        Session2 -> L2 Cache
        Session3 -> L2 Cache
        Session4 -> L2 Cache

Huge performance improvement.

2️⃣ Improves Scalability

High traffic systems:

    1000 users -> same product

Without L2:

    1000 DB queries

With L2:

        1 DB query
        999 cache hits
        3️⃣ Useful for Frequently Read Data

Example tables:

        Countries
        States
        Product categories
        Currency
        Configuration tables

These rarely change.

Popular L2 Cache Providers

    Hibernate itself doesn't implement cache storage. It integrates with cache providers like:

    Ehcache
    Redis
    Hazelcast
    Infinispan

Important Interview Point ⚡

L2 cache is NOT enabled by default.

You must:

        1️⃣ Enable cache provider
        2️⃣ Enable cache in Hibernate config
        3️⃣ Mark entities as cacheable




**_When L2 Cache IS Required / Useful**_

Use L2 cache when data is:

1️⃣ Frequently Read but Rarely Updated

Example tables:

        Countries
        States
        Product categories
        Configuration tables

Example:

Countries Table
---------------
1 India
2 USA
3 Germany

    Application flow:
    
    1000 users login
    ↓
    each needs country list
    
    Without L2:
    
    1000 DB queries

With L2:

    1 DB query
    999 cache hits
    
    So DB load drastically reduces.

2️⃣ Reference / Master Data

Example:
    
    Currency
    Tax Types
    Product Types
    Roles
    Permissions

These change very rarely, so caching them is safe.

3️⃣ Expensive Queries

If retrieving an entity involves:

        heavy joins
        complex mapping
        large object graph

Caching avoids repeating the expensive DB work.

Example:

Product

        → Category
        → Supplier
        → PriceHistory

If every request loads this from DB, performance suffers.

4️⃣ High Traffic Applications

Example:

    An e-commerce product page.
    10000 users viewing same product
    Without cache:
    10000 DB hits
    
    With L2 cache:
    1 DB hit
    9999 cache hits



When L2 Cache SHOULD NOT Be Used
1️⃣ Frequently Updated Data

Example:

        Bank account balance
        Order status
        Stock quantity

Why bad?

    Because cache must be updated every time.

Example:

Update → DB
Update → Cache
Invalidate cache across servers

    This creates synchronization overhead.
    Updating cache all the time becomes a problem because the whole purpose of cache is to reduce work. 
    If the data changes frequently, the system must constantly keep cache and database in sync, which creates extra overhead.

2️⃣ Transactional Data

Example:

        payments
        orders
        inventory
        financial transactions

These must always be fresh.

If cached:

    User A updates balance
    User B reads old cached value

This leads to data inconsistency.

3️⃣ Highly Dynamic Data

Example:

    Live stock prices
    Ride availability
    Auction bids

Such data changes every second, so caching becomes useless.

4️⃣ Large Objects Used Rarely

Example:

        Huge audit logs
        Large reports
        Historical data

Caching them wastes memory.

Simple Rule (Best Interview Answer)

Use L2 cache when data is:

    READ FREQUENTLY
    UPDATED RARELY
    SHARED ACROSS SESSIONS

Avoid when data is:

    UPDATED FREQUENTLY
    TRANSACTION CRITICAL
    REAL-TIME


![img.png](Images/L2C1.png)

![img_1.png](Images/L2C2.png)

![img_2.png](Images/L2C3.png)


![img_3.png](Images/L2C4.png)

![img_4.png](Images/L2C5.png)

![img_5.png](Images/L2C6.png)

![img.png](Images/L2C7.png)

![img_1.png](Images/L2C8.png)

![img_2.png](Images/L2C9.png)

![img_3.png](Images/L2C10.png)

![img_4.png](Images/L2C11.png)

![img_5.png](Images/L2C12.png)

![img_6.png](Images/L2C13.png)

![img_7.png](Images/L2C14.png)

![img.png](Images/L2C15.png)

![img_1.png](Images/L2C16.png)

![img_2.png](Images/L2C17.png)

![img_3.png](Images/L2C18.png)

![img_4.png](Images/L2C19.png)


1️⃣ Add Cache Provider Dependency

Hibernate does NOT store cache itself.
It needs a cache provider.

Example: Ehcache.

Maven dependency
<dependency>
<groupId>org.hibernate.orm</groupId>
<artifactId>hibernate-jcache</artifactId>
</dependency>

<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
Why this is needed

hibernate-jcache → Allows Hibernate to talk to cache implementations

ehcache → The actual cache storage

Architecture:

Application
|
Hibernate
|
JCache API
|
Ehcache (actual cache)

Hibernate delegates caching to a provider.

2️⃣ Enable Second Level Cache in Configuration

Add properties in application.properties or hibernate.cfg.xml.

Example:

spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.jpa.properties.hibernate.javax.cache.provider=org.ehcache.jsr107.EhcacheCachingProvider
Why each property is needed
1️⃣ Enable L2 cache
hibernate.cache.use_second_level_cache=true

Why?

By default:

      L1 cache → enabled
      L2 cache → disabled

This line turns on L2 caching globally.

2️⃣ Cache Region Factory
hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory

Why?

      Hibernate organizes cache into regions.

Example regions:

      User
      Product
      Order

Each entity gets its own cache region.

This factory:

      creates
      manages
      organizes

those cache regions.

3️⃣ Cache Provider
hibernate.javax.cache.provider=org.ehcache.jsr107.EhcacheCachingProvider

Why?

Hibernate must know which caching system to use.

Possible providers:

      Ehcache
      Redis
      Hazelcast
      Infinispan

This line tells Hibernate:

Use Ehcache as cache storage
3️⃣ Create Cache Configuration File

File:

ehcache.xml

Example:

      <config xmlns="http://www.ehcache.org/v3">
      
          <cache alias="com.example.User">
      
              <heap unit="entries">1000</heap>
      
          </cache>
      
      </config>
Why this is needed

This defines:

how much memory
how many objects
eviction policy

Example meaning:

User cache → store 1000 objects

If more come:

old entries evicted
4️⃣ Enable Cache on Entity

Example entity:

import jakarta.persistence.Entity;
import jakarta.persistence.Cacheable;
import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {

    @Id
    private Long id;

    private String name;

}

Now let’s explain each annotation.

1️⃣ @Cacheable
@Cacheable

Why?

Tells Hibernate:

this entity can be stored in L2 cache

Without this:

Hibernate will NEVER cache the entity

Even if L2 cache is enabled globally.

2️⃣ @Cache
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)

Why?

Defines how cache handles concurrent access.

Possible strategies:

Strategy	Use Case
READ_ONLY	data never changes
READ_WRITE	data updates occasionally
NONSTRICT_READ_WRITE	slight stale data allowed
TRANSACTIONAL	strict transactional consistency

Example:

Country → READ_ONLY
User → READ_WRITE
5️⃣ How It Works Internally

Let’s say code runs:

User user = entityManager.find(User.class, 1);

Flow:

Step 1 → Check L1 cache
Step 2 → Check L2 cache
Step 3 → Query DB
Step 4 → Store in L2 cache

Next request:

Step 1 → L1 cache
Step 2 → L2 cache HIT
Step 3 → DB not called

Performance improves.



1️⃣ READ_ONLY Strategy
What it means

The cached entity can never be updated.

If an update happens, Hibernate throws an exception.

Example:

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
public class Country {

    @Id
    private Long id;

    private String name;
}

Example table:

Country
-------
1 India
2 USA
3 Germany

Flow:

User1 → read country
User2 → read country
User3 → read country

Cache works perfectly because data never changes.

What happens if update is attempted
update Country set name="Bharat" where id=1

Hibernate will throw an error because the entity is immutable in cache.

When to use

Use for static reference data:

Countries
Currencies
States
Tax types
Why it is fastest

Because:

No locks
No cache invalidation
No synchronization

Just read directly from cache.

2️⃣ READ_WRITE Strategy
What it means

Allows both reads and updates, while ensuring cache consistency.

Example:

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class User {

    @Id
    private Long id;

    private String name;
}

Example table:

User
----
1 Alice
2 Bob
How Hibernate ensures consistency

Hibernate uses soft locks.

Example scenario:

User A updating record
User B reading record

Flow:

1. Cache entry locked
2. DB updated
3. Cache updated
4. Lock released

Example flow:

Thread1 → update user
Thread2 → read user

While updating:

Cache entry marked LOCKED

Other threads:

wait OR read old value
Example

Initial cache:

User(1) = Alice

Update happens:

update user set name="Alicia"

Steps:

1 lock cache entry
2 update database
3 update cache
4 release lock

Now cache contains:

User(1) = Alicia
When to use

For occasionally updated entities:

Users
Products
Profiles
Settings
Performance
Good performance
Moderate locking overhead
3️⃣ NONSTRICT_READ_WRITE Strategy
What it means

Allows updates but does not guarantee strong consistency.

Cache may temporarily contain stale data.

Example:

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
public class Product {

    @Id
    private Long id;

    private String name;
}
Example scenario

Initial cache:

Product(10) = Laptop

Update happens:

update product set name="Gaming Laptop"

What happens:

DB updated
Cache entry invalidated

But another user might still read old cached value briefly.

Example:

User1 reads → Laptop
User2 updates → Gaming Laptop
User3 reads → Laptop (stale)

Eventually cache refreshes.

Why this happens

This strategy does not lock cache entries.

So:

better performance
weaker consistency
When to use

Use when slightly stale data is acceptable.

Examples:

Product descriptions
Blog posts
Catalog metadata
4️⃣ TRANSACTIONAL Strategy
What it means

Provides full transactional consistency between:

cache
database

Used with transactional caches like:

Infinispan

Ehcache (JTA mode)

Example:

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.TRANSACTIONAL)
public class Account {

    @Id
    private Long id;

    private Double balance;
}

Flow:

Transaction begins
Update DB
Update cache
Commit transaction

If transaction rolls back:

cache rollback
database rollback

Everything remains consistent.

Example

Initial:

Account balance = 1000

Transaction:

withdraw 200

Steps:

1 start transaction
2 update DB
3 update cache
4 commit

Both contain:

balance = 800
When to use

Only for distributed transactional caches.

Example systems:

banking systems
distributed enterprise applications