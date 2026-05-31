1. What is JPA?

        Java Persistence API (JPA) is a Java specification used for managing relational data in Java applications.
        
        It provides:
            
            Standard annotations
            Interfaces
            ORM rules
            Persistence APIs
        
        JPA itself does NOT contain actual implementation logic.
        
        It only defines:
            
            what should happen
            not how it should happen
        
        Examples of JPA interfaces:
            
            EntityManager
            EntityTransaction
            Query
            Why JPA was created?
        
        Before JPA:
            
            every ORM framework had its own APIs
            switching frameworks was difficult
            
            JPA standardized ORM behavior.
            
            So applications can switch implementations easily.

2. What is Hibernate?
        
        Hibernate ORM is an ORM framework that implements the JPA specification.
        
        Meaning:
        
        JPA defines rules
        Hibernate provides actual implementation
        
        Hibernate internally handles:
            
            SQL generation
            caching
            dirty checking
            transaction synchronization
            lazy loading
        
        Example
            
            @Entity
            class User {
            @Id
            private Long id;
            }
        
        Annotations come from JPA.
        
        But Hibernate executes the actual SQL.
        
        Important Interview Point
        
        You can use:
        
        JPA with Hibernate
        JPA with EclipseLink
        JPA with OpenJPA
        
        Hibernate is just one implementation.

3. What is ORM?

       Object Relational Mapping is a technique that maps Java objects to relational database tables.

       Mapping Example
       Java	   Database
       Class	Table
       Object	Row
       Field	Column

Example:

        User user = new User();
        
        maps to:
        
        users table
        Why ORM?
        
        Without ORM:
        
        developers manually write SQL
        manually map ResultSet to objects
        
        ORM automates:
        
        INSERT
        UPDATE
        DELETE
        SELECT
        Benefits
        less boilerplate
        database independence
        cleaner code
        easier maintenance

4. What is an Entity?
   Answer

An entity is a Java class mapped to a database table.

Defined using:

@Entity

Example:
    
    @Entity
    @Table(name="users")
    public class User {
    
        @Id
        private Long id;
    
        private String name;
    }
Important Rules

An entity should:
    
    have default constructor
    contain primary key
    be non-final
    usually use getters/setters
    Internally

Hibernate stores metadata for entities and tracks them inside Persistence Context.

5. What is @Id?
   
        @Id marks the primary key of an entity.

Example:
    
    @Id
    private Long id;
Why Needed?

Hibernate needs unique identity to:
        
        track entities
        perform updates
        maintain cache consistency
        Commonly Used With
        @GeneratedValue(strategy = GenerationType.IDENTITY)

Example:
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

6. Difference Between save() and persist()
                   save()	                           persist()
                   Hibernate-specific	            JPA standard
                    Returns generated ID	                Returns void
                   Can execute insert immediately	Managed inside persistence context
                   Older Hibernate API	             Recommended in JPA
           Detailed Explanation
        
           save()
        
               Long id = session.save(user);
               returns generated primary key
               directly related to Hibernate Session API
               persist()
               entityManager.persist(user);
               makes entity managed
               insert may happen later during flush
           Interview Best Practice
        
        In Spring Boot:
        
        prefer persist()/EntityManager
        because it is standard JPA

7. What is Persistence Context?
   Answer

🔥 One of the MOST important JPA concepts.
    
    Persistence Context is a container where Hibernate manages entity objects.
    
    Also called:
    
    First-Level Cache
    Session Cache
    Responsibilities
    
    It tracks:
    
    entity states
    snapshots
    dirty checking
    identity management
Example
    
    User user = entityManager.find(User.class,1);
    
    Now user becomes:
    
    managed entity
    stored in persistence context
    Important Feature
    
    Same entity fetched twice:
    
    find(User.class,1);
    find(User.class,1);
    
    Second call comes from cache.
    
    No extra SQL query.

8. Entity Lifecycle States
    
       4 Entity States
       State	Meaning
       Transient	Normal Java object
       Persistent	Managed by Hibernate
       Detached	Exists but unmanaged
       Removed	Marked for deletion
           1. Transient
              User u = new User();
              not saved
              no DB row
              Hibernate unaware
              2. Persistent
                 entityManager.persist(u);
        
        Now:
        
        managed by Hibernate
        tracked inside persistence context
        3. Detached
           entityManager.close();
        
        Object still exists but:
        
        Hibernate no longer tracks it
        4. Removed
           entityManager.remove(user);
        
        Marked for deletion.
        
        Actual delete happens during flush/commit.

9. What is Detached Entity?
   Answer

Detached entity:
    
    exists in memory
    originally persistent
    but no longer managed
    Causes
    session close
    clear()
    detach()
    serialization
    Problem

Changes are NOT automatically saved.

Example:
    
    user.setName("bharath");
    
    No update query generated.
    
    Reattach
    
    Use:
    
    merge()

10. What is Dirty Checking?
    Answer

🔥 Extremely important interview topic.
        
        Dirty Checking means:
        Hibernate automatically detects changes in managed entities.
        
        Example:
        
        User user = entityManager.find(User.class,1);
        
        user.setName("bharath");
        
        No explicit update query needed.
        
        Hibernate automatically generates:
        
        UPDATE users SET name='bharath'
        
        during flush/commit.

11. How Dirty Checking Works Internally?
    Internal Flow
        
        When entity becomes persistent:
        Hibernate stores:
        
        original snapshot
        current object reference
        
        Example snapshot:
        
        Original:
        name = john
        
        Current object:
        
        name = bharath
        
        During flush:
        Hibernate compares both.
        
        If different:
        
        generates UPDATE query
        Why Important?
        
        Reduces manual SQL writing.

12. What is Flush?
    Answer

        Flush synchronizes Persistence Context with database.

Meaning:

    pending SQL statements are executed.

Important

Flush ≠ Commit

Flush only sends SQL.
Transaction may still rollback.

Database still doesnot commits it wil be in undo or redologs

Example Flow
persist(user);
flush();

INSERT query sent.

But if rollback occurs:
data not permanently saved.

13. Flush vs Commit

    Flush	                          Commit
    Sends SQL to DB	Permanently  saves transaction
    Transaction still active	 Transaction ends
    Can rollback	             Cannot rollback after commit
    Important Interview Line

Flush synchronizes state, commit finalizes transaction.

14. What Triggers Flush?
    
        Automatic Flush Triggers
        1. Transaction Commit
           transaction.commit();
        2. JPQL Query Execution
    
    Hibernate flushes before query execution to maintain consistency.
    
3. Manual Flush
   entityManager.flush();

15. What is First-Level Cache?
    
    
            First-level cache is:
            
            session scoped cache
            enabled by default
        
     Every Persistence Context has its own cache.
        
     Example
     User u1 = find(User.class,1);
     User u2 = find(User.class,1);
        
     Second query:
        
     comes from cache
     no DB hit
     Benefits
     reduces SQL
     improves performance
     ensures object identity
    
16. What is Second-Level Cache?
    Answer

        Second-level cache is shared across sessions.

Unlike first-level cache:
        
        survives beyond single session
        Requires External Provider

Examples:
    
    Ehcache
    Hazelcast
    Redis
    Flow
    Session -> L1 Cache -> L2 Cache -> DB
    Benefit

    Reduces repeated DB access.

17. What is Lazy Loading?
    Answer
        
        Lazy loading means:
        related data loads ONLY when accessed.

Example:
    
    @OneToMany(fetch = FetchType.LAZY)
    private List<Order> orders;
    
    Orders NOT loaded initially.
    
    Only loaded when:
    
    user.getOrders();
    Advantage
    
    Better performance.

18. What is Eager Loading?
    Answer
        
        Eager loading means:
        related entities loaded immediately.
        
        Example:
        
        @OneToMany(fetch = FetchType.EAGER)
        
        When User loads:
        Orders also load automatically.
        
        Disadvantage
        
        Can create:
        
        huge joins
        unnecessary data loading
        performance issues

19. Lazy vs Eager
        
                Lazy	Eager
                Loads on demand	Loads immediately
                Better performance	Can be expensive
                Preferred in most cases	Use carefully
                Best Practice
        
        Default to:
        
        LAZY
        
        Fetch explicitly when needed.

20. What is LazyInitializationException?
    

    🔥 Very common Hibernate exception.
    
    Occurs when:
    
    lazy object accessed
    session already closed
    
    Example:
    
    User user = repo.findById(1);
    
    transaction ends
    
    user.getOrders();
    
    Orders cannot load because:
    Persistence Context already closed.

21. How to Fix LazyInitializationException?
        
            1. JOIN FETCH
               SELECT u FROM User u
               JOIN FETCH u.orders
        
        Best solution.
        
        2. DTO Projection
        
        Load exactly needed data.
        
        3. Transactional Scope
        
        Access lazy objects inside transaction.
        
        Avoid
        
        ❌ Making everything EAGER
        
        ❌ Open Session In View abuse
        
        Because:
        
        creates performance problems
        hides architecture issues
22. What is N+1 Problem?
    Answer

🔥 Top performance question.
    
    Example:
    
    List<User> users = repo.findAll();
    
    1 query loads users.
    
    Then:
    
    user.getOrders()
    
    for each user triggers another query.
    
    Total:
    
    1 + N queries
    
    Huge performance issue.

23. How to Solve N+1 Problem?
    Solutions
        
            1. JOIN FETCH
        
        Best approach.
        
        SELECT u FROM User u
        JOIN FETCH u.orders
        2. EntityGraph
        
        Dynamic fetch planning.
        
        3. Batch Fetching
        
        Hibernate loads collections in batches.

Important Interview Point

    N+1 is NOT caused by lazy loading alone.

It is caused by:

    accessing lazy relations repeatedly
24. What is Cascade?
    Answer
        
        Cascade propagates operations from parent to child.
        
        Example:
        
        @OneToMany(cascade = CascadeType.ALL)
        Example
        
        Saving parent:
        
        entityManager.persist(parent);
        
        Automatically saves children.
        
        Types
        Cascade Type	Meaning
        PERSIST	save child
        REMOVE	delete child
        MERGE	merge child
        ALL	all operations

    25. What is orphanRemoval?
        Answer
        
            Deletes child entity when removed from collection.
        
            Example:
        
            @OneToMany(orphanRemoval = true)
            Example
            parent.getChildren().remove(child);
        
            Hibernate automatically deletes child row.

26. Cascade REMOVE vs orphanRemoval

        Cascade REMOVE	                      orphanRemoval
        Child deleted when parent deleted	Child deleted when removed from collection

        Example
        Cascade REMOVE
        delete parent
        → deletes children
        orphanRemoval
        remove child from list
        → deletes child

 27. What is mappedBy?
        
    
        mappedBy defines:
    
        inverse side
        non-owning side of relationship
    
        Example:
    
        @OneToMany(mappedBy = "user")
        private List<Order> orders;
        Why Needed?
    
        Prevents Hibernate from creating:
    
        unnecessary join tables
        duplicate mappings

        
        mappedBy prevents:
        
        duplicate relationship mappings
        extra join tables
        inconsistent updates

28. What is Owning Side?
    
        
        Owning side controls relationship updates in DB.
        
        Usually side containing:
        
        @JoinColumn
        
        Example:
        
        @ManyToOne
        @JoinColumn(name="user_id")
        private User user;
        
        This side updates foreign key.


29. What is @Transactional?
    
    
    @Transactional defines transaction boundary.
    
    Spring automatically:
    
    starts transaction
    commits on success
    rollbacks on exception
    
    Example:
    
    @Transactional
    public void saveUser() {
    }

    Read ACID.md and transaction.md


30. How Does @Transactional Work Internally?
    Answer
        
        🔥 Extremely important Spring interview topic.
        
        Spring uses:
        
        AOP Proxy
        Dynamic Proxy or CGLIB
        Internal Flow
        Client
        ↓
        Proxy Object
        ↓
        Transaction Manager
        ↓
        Actual Method
        Steps
        1. Proxy intercepts method call
        
        Spring creates proxy around bean.
        
        2. Starts Transaction
        
        Using:
        
        PlatformTransactionManager
        3. Executes Method
        
        Business logic runs.
        
        4. Commit or Rollback
           success → commit
           exception → rollback

    During application startup, Spring detects @Transactional methods and creates proxy objects around those beans. When a transactional method is called, the proxy intercepts the call and delegates it to TransactionInterceptor, which starts the transaction, executes the method, and then commits or rolls back the transaction depending on the outcome.


1. What is a Transaction?

A transaction is:

A group of operations executed as one unit.

Properties:

all succeed OR all fail
2. What are ACID properties?
   Property	Meaning
   Atomicity	all or nothing
   Consistency	valid state maintained
   Isolation	concurrent safety
   Durability	data survives crash
3. What does @Transactional do?
   @Transactional
   public void transfer() {}

👉 Creates transaction boundary.

Spring:

opens transaction
executes method
commit/rollback
4. How does @Transactional work internally?

🔥 MOST ASKED

Uses:

AOP proxy

Flow:

Client calls proxy
Proxy starts transaction
Method executes
Commit or rollback
5. Why does @Transactional fail on private methods?

🔥 CLASSIC INTERVIEW QUESTION

Because:

proxy cannot intercept private methods
@Transactional
private void test() {}

❌ ignored

6. Why does self-invocation break @Transactional?
   this.methodB();

👉 call bypasses proxy

So:

transaction advice not applied
7. What is Propagation?

Defines:

How transaction behaves when another transaction already exists.

8. What is REQUIRED propagation?

🔥 Default propagation

Propagation.REQUIRED

Behavior:

join existing transaction
else create new
9. What is REQUIRES_NEW?

Always creates NEW transaction.

Suspends existing transaction.

10. REQUIRED vs REQUIRES_NEW?
    REQUIRED	REQUIRES_NEW
    joins existing	always new
    single transaction	nested independent transaction
11. Real use case of REQUIRES_NEW?

Logging/audit.

Even if parent fails:

audit transaction commits
12. What is SUPPORTS?
    join if transaction exists
    else run non-transactionally
13. What is NOT_SUPPORTED?

Suspends transaction.

Runs without transaction.

14. What is MANDATORY?

Requires existing transaction.

Else throws exception.

15. What is NEVER propagation?

Fails if transaction exists.

16. What is Isolation Level?

Controls concurrent transaction visibility.

17. What problems isolation solves?
    dirty read
    non-repeatable read
    phantom read
18. What is Dirty Read?

Reading uncommitted data.

Transaction A:

UPDATE balance=0

Transaction B reads before commit.

Dangerous.

19. What is Non-repeatable Read?

Same row gives different values within same transaction.

20. What is Phantom Read?

Rows appear/disappear between queries.

21. Isolation Levels?
    Level	Prevents
    READ_UNCOMMITTED	nothing
    READ_COMMITTED	dirty reads
    REPEATABLE_READ	non-repeatable
    SERIALIZABLE	all
22. Default isolation in most DBs?

Usually:

READ_COMMITTED

MySQL commonly:

REPEATABLE_READ
23. What causes rollback in Spring?

🔥 IMPORTANT

By default:
✅ rollback for:

RuntimeException
Error

❌ NOT rollback for:

checked exceptions
24. How to rollback checked exceptions?
    @Transactional(rollbackFor = Exception.class)
25. What is readOnly transaction?
    @Transactional(readOnly = true)
  skips dirty check

Optimizes read operations.

May:

skip dirty checking
optimize DB interaction
26. Does readOnly prevent updates?

❌ No guarantee.

Mostly optimization hint.

27. What is TransactionManager?

Component managing transactions.

Examples:

JpaTransactionManager
DataSourceTransactionManager
28. What is programmatic transaction management?

Manual transaction handling:

TransactionTemplate

Used for fine-grained control.

29. What happens if exception is caught inside transaction?

🔥 VERY TRICKY

try {
...
} catch(Exception e) {}

👉 Spring thinks method succeeded.

❌ No rollback.

30. Best practices for transactions?

✅ Keep transaction small
✅ Avoid remote calls inside transaction
✅ Use service layer
✅ Avoid long-running transactions



@Modifying is used for both JPQL and native queries whenever the query modifies data.

It is needed for:

UPDATE
DELETE
sometimes INSERT (native query)
JPQL Example
@Transactional
@Modifying
@Query("UPDATE Employee e SET e.salary = :salary WHERE e.id = :id")
int updateSalary(Long id, double salary);

Without @Modifying:

Spring thinks it is a SELECT query and throws exception.

Native Query Example
@Transactional
@Modifying
@Query(
value = "UPDATE employee SET salary = ? WHERE id = ?",
nativeQuery = true
)
int updateSalary(double salary, Long id);

Works here also.

Why Is @Modifying Needed?

Normally @Query is treated as:

SELECT query

@Modifying tells Spring Data:

"This query changes data."

So Spring uses:

executeUpdate()
instead of
executeQuery()

internally.

When @Modifying is NOT Needed
Derived delete methods
deleteById(id);

No need.

Repository save methods
save(entity);

No need.

Hibernate manages automatically.

Important Note About INSERT

JPQL does NOT support normal INSERT queries.

So:

INSERT INTO Employee ...

works only in:

native query

with:

@Modifying
nativeQuery = true
Quick Table
Query Type	SELECT	UPDATE	DELETE	INSERT
JPQL	❌ No @Modifying	✅ Needed	✅ Needed	❌ Not supported
Native SQL	❌ No @Modifying	✅ Needed	✅ Needed	✅ Needed
Interview One-Liner

@Modifying is required for both JPQL and native queries whenever the query performs UPDATE, DELETE, or INSERT operations, because Spring Data must treat it as a data-modifying query instead of a SELECT query.

why cant derived queries use update?

Derived queries are designed around entity retrieval and simple delete/count/exists operations.

Spring Data parses method names like:

findByName()
deleteById()
countByStatus()
existsByEmail()

These map naturally to standard repository operations.

But update queries are fundamentally different.

Main Reason

An UPDATE query needs:

fields to change
new values
conditions

Method names become ambiguous and extremely complex.

Imagine this:

updateSalaryById(double salary, Long id)

Questions:

Which field is being updated?
Is it partial update?
Multiple fields?
Increment or replace?
Null handling?
Versioning?
Dynamic conditions?

Spring Data chose not to support parsing such modifying semantics from method names.

Another Important Reason: JPA Update Semantics

Normally JPA updates entities like this:

Employee e = repo.findById(id).get();
e.setSalary(5000);

Hibernate dirty checking detects change:

managed entity changed
↓
UPDATE generated automatically

So JPA philosophy is:

modify entities
not write manual UPDATE statements frequently
Why Delete Derived Queries Exist Then?

Delete is simpler.

deleteByName(String name)

means clearly:

DELETE FROM employee WHERE name=?

No ambiguity.

What Happens Internally in Normal Entity Update
@Transactional
public void update() {
Employee e = repo.findById(1L).get();
e.setSalary(1000);
}

Hibernate tracks entity snapshot:

old salary = 500
new salary = 1000

At flush/commit:

UPDATE employee SET salary=1000 WHERE id=1

generated automatically.

No explicit update query needed.

Why Explicit JPQL UPDATE Exists Then?

For bulk updates.

Example:

@Modifying
@Query("UPDATE Employee e SET e.salary = e.salary * 1.1")

This updates directly in DB without loading entities.

Much faster for large data.

Important Interview Point

Bulk JPQL updates:

bypass persistence context
bypass dirty checking
may cause stale entities in memory

That is another reason Spring treats them specially using @Modifying.

Quick Table
Operation	Derived Query Support?	Why
findBy	✅	Retrieval semantics easy
countBy	✅	Simple aggregation
existsBy	✅	Boolean existence
deleteBy	✅	Clear delete semantics
updateBy	❌	Ambiguous modifying semantics