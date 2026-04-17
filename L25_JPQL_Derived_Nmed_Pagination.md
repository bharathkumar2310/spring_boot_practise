DERIVED QUERY:

    ---> A derived query is a query which is automatically derived by JPA from method names in our JPARepo interface
    A derived query is created just by defining a method like:
    
    List<User> findByName(String name);
    
    👉 Spring Data JPA derives the query from the method name.
    
    Equivalent JPQL:
    
        SELECT u FROM User u WHERE u.name = :name

In Spring Data JPA, a derived query (method-name-based query) is first converted into JPQL, not directly into SQL.

Example : findDistinctTop3ByNameOrderByAgeDesc()



1. PartTree (🔥 Most Important)

👉 Entry point for parsing method names

    new PartTree("findDistinctTop3ByNameOrderByAgeDesc", User.class);

PartTree splits it into Subject, Predica

✅ 1. SUBJECT
findDistinctTop3

PartTree extracts:

isDistinct = true
maxResults = 3
queryType = SELECT


| Prefix                | Meaning      |
| --------------------- | ------------ |
| `find`, `read`, `get` | SELECT       |
| `count`               | COUNT query  |
| `exists`              | EXISTS query |
| `delete`, `remove`    | DELETE query |


✅ 2. PREDICATE
Name

👉 Now predicate is further split into Parts

Creates:

Part(property="name", type=SIMPLE_PROPERTY)


| Keyword        | Enum            |
| -------------- | --------------- |
| `Is`, `Equals` | SIMPLE_PROPERTY |
| `Between`      | BETWEEN         |
| `LessThan`     | LESS_THAN       |
| `GreaterThan`  | GREATER_THAN    |
| `Like`         | LIKE            |
| `Containing`   | CONTAINING      |
| `StartingWith` | STARTING_WITH   |
| `EndingWith`   | ENDING_WITH     |
| `In`           | IN              |
| `True`         | TRUE            |
| `False`        | FALSE           |
| `IsNull`       | IS_NULL         |
| `IsNotNull`    | IS_NOT_NULL     |



✅ 3. ORDER BY
OrderByAgeDesc

Stored as:

Sort → age DESC

PartTree
Parses method name
Splits into subject + predicate
Part
Represents each condition (name, age, etc.)






![img.png](Images/DQ1.png)

![img_1.png](Images/DQ2.png)

![img_2.png](Images/DQ3.png)

![img_3.png](Images/DQ4.png)

![img_4.png](Images/DQ5.png)


![img_5.png](Images/DQ6.png)

![img_6.png](Images/DQ7.png)

![img_7.png](Images/DQ8.png)


👉 Spring Data JPA generates JPQL (HQL), NOT SQL directly for derived queries
👉 Then Hibernate converts JPQL → SQL

1. JPA’s core model (this is the real reason)

JPA (with providers like Hibernate) works like this:

Load entity into memory
Modify object
ORM detects changes (dirty checking)
Generates UPDATE SQL automatically

👉 So updates are supposed to happen like:

User user = repo.findById(id).get();
user.setName("New Name");   // change state
// Hibernate auto-updates

Not like:

UPDATE user SET name = 'New Name' WHERE id = 1;
2. Derived queries are just “query builders”

Derived queries:

findByNameAndAge(...)
Parsed into JPQL SELECT
Used to fetch entities

👉 They don’t deal with:

Entity lifecycle
Persistence context
Dirty checking

So they are intentionally limited to read operations

3. Why update via derived query is problematic ❌

If Spring allowed:

updateByName(...)

It would:

Bypass persistence context ❌
Skip dirty checking ❌
Not update already loaded entities ❌

👉 Leads to inconsistent state in memory vs DB

4. Why deleteBy is allowed then? 🤔

Good catch — this is often asked.

deleteByName(String name)

Works because:

Internally:
Either fetches entities → deletes them
OR executes safe delete query

Deletion:

Doesn’t need field-level tracking like updates
Easier to keep consistency
Easier to keep consistency



PAGINATION AND SORTING: 


# 🧩 1️⃣ `Pageable` — The _input_ (what page you want)

`Pageable` is just an **interface** that defines:

    - Which page to fetch
      - How many records per page
      - How to sort them


It does **not** contain data — it’s just a _description_ of the page request.
**Methods in `Pageable`:**

`int getPageNumber();   // which page (0-based)
 int getPageSize();     // how many records per page 
 Sort getSort();        // sorting info`

---

# 🧩 2️⃣ `PageRequest` — The _implementation_ of `Pageable`

    `PageRequest` is a **class** that implements `Pageable`.
    You usually create it using its static factory methods:

### Example:

`Pageable pageable = PageRequest.of(0, 10);`

✅ Meaning:  
Fetch **page 0** (first page), with **10 elements per page**.

You can also add sorting:

`Pageable pageable = PageRequest.of(1, 5, Sort.by("name").descending());`

✅ Meaning:  
Fetch **page 1** (second page), 5 records per page, sorted by `name` descending.



# 🧩 5️⃣ `Page<T>` — The _output result_

`Page<T>` is a **container** for:

- The actual content (`List<T>`)

- Metadata about pagination


### Example usage:

```
Page<User> page = userRepository.findByStatus("ACTIVE", pageable);

List<User> users = page.getContent();          // actual results
int totalPages = page.getTotalPages();         // total number of pages
long totalElements = page.getTotalElements();  // total count
int pageNumber = page.getNumber();             // current page
boolean hasNext = page.hasNext();              // more pages?

```

When you pass:

    PageRequest.of(page, size)

It converts to:

    LIMIT size OFFSET (page * size)

![img.png](Images/pagsort1.png)

![img_1.png](Images/pagsort2.png)

![img_2.png](Images/pagsort3.png)

![img_3.png](Images/pagsort4.png)

![img_4.png](Images/pagsort5.png)

![img_5.png](Images/pagsort6.png)

![img_6.png](Images/pagsort7.png)

![img_7.png](Images/pagsort8.png)

![img_8.png](Images/pagsort9.png)


🧠 What is OFFSET?

👉 OFFSET tells the database how many rows to skip before starting to return results.

🔥 Simple Definition
OFFSET = number of records to SKIP
📊 Example
SELECT * FROM user
LIMIT 5 OFFSET 0;

👉 Means:

Skip 0 rows
Return next 5 rows
SELECT * FROM user
LIMIT 5 OFFSET 5;

👉 Means:

Skip first 5 rows
Return next 5 rows

| Return Type | Meaning                            |
| ----------- | ---------------------------------- |
| `Page<T>`   | full pagination (count + data)     |
| `Slice<T>`  | only next page info (faster)       |
| `List<T>`   | just data (no pagination metadata) |

Slice is a lightweight version of pagination
It gives you:

✔ Current page data
✔ Whether next page exists

❌ But NOT total count



JPQL :


![img.png](Images/JPQL1.png)

![img_1.png](Images/JPQL2.png)

![img_2.png](Images/JPQL3.png)

![img_3.png](Images/JPQL4.png)

![img_4.png](Images/JPQL5.png)

![img_5.png](Images/JPQL6.png)

----------------------------------------------------------------------------------------------------------------------------------------------

For JPL queries also proxy directly calls enity manager.createquery("jpqlquery")



--------------------------------------------------------------------------------------------------------------------------------------



## ⚙️ How Do Derived Query They Work Internally?

Let’s break it down into **4 phases** to understand the internals clearly:

---

### **1️⃣ Repository Interface Parsing**

When the application starts (Spring context initialization), Spring scans your repository interfaces — like `UserRepository`.

Spring Data detects all methods that:

- Aren’t standard CRUD ones (`save`, `findAll`, etc.), and

- Don’t have an explicit `@Query` annotation.


These methods are assumed to be **query derivation methods**.

---

### **2️⃣ Query Name Parsing**

Spring uses a class called **`PartTree`** (from `org.springframework.data.repository.query.parser`) to **parse the method name**.

For example:

`findByEmailAndAgeGreaterThan`

gets broken into parts:

| Segment | Keyword | Property | Operator |  
|----------|----------|-----------|  
| `Email` | `By` | `email` | `=` |  
| `AndAgeGreaterThan` | `And` | `age` | `>` |

The `PartTree` parser builds an **abstract syntax tree (AST)** representing these parts.

---

### **3️⃣ Query Creation**

Once parsed, Spring creates a `QueryMethod` object (from `org.springframework.data.repository.query`).

Then, depending on the repository type (JPA, Mongo, etc.), it delegates to a **query builder** — for JPA, that’s handled by:

- `JpaQueryLookupStrategy`

- `PartTreeJpaQuery`


These internally use **JPA Criteria API** to dynamically construct a `javax.persistence.criteria.CriteriaQuery` matching your derived conditions.

So for example:

`findByEmailAndAgeGreaterThan(String email, int age)`

turns into something equivalent to:

```
CriteriaBuilder cb = entityManager.getCriteriaBuilder();
CriteriaQuery<User> query = cb.createQuery(User.class);
Root<User> root = query.from(User.class);

Predicate emailPredicate = cb.equal(root.get("email"), email);
Predicate agePredicate = cb.greaterThan(root.get("age"), age);
query.where(cb.and(emailPredicate, agePredicate));

return entityManager.createQuery(query).getResultList();

```

That’s what **Spring JPA builds for you internally**.

---

### **4️⃣ Query Execution**

When you call the method, e.g.:

`userRepository.findByEmailAndAgeGreaterThan("a@b.com", 25);`

Spring executes the dynamically created query through the **EntityManager**:

- It binds the parameters in order,

- Executes it on the database,

- Maps the result set back into your entity class.


---

## 🧠 Bonus: Derived Query Keywords

Spring supports a huge set of keywords to derive conditions. Examples:

|Keyword|Operator|
|---|---|
|`Is`, `Equals`|`=`|
|`Between`|`BETWEEN ? AND ?`|
|`LessThan`, `GreaterThan`|`<`, `>`|
|`Like`, `Containing`|`LIKE %?%`|
|`StartingWith`, `EndingWith`|`LIKE ?%`, `LIKE %?`|
|`IsNull`, `IsNotNull`|`IS NULL`, `IS NOT NULL`|
|`In`, `NotIn`|`IN (...)`, `NOT IN (...)`|
|`OrderBy`|Sort results|
|`IgnoreCase`|`LOWER(column)` used for comparison|

---

## 🧩 Summary

|Step|Internal Component|Purpose|
|---|---|---|
|1️⃣|`RepositoryFactoryBean`|Scans and instantiates repositories|
|2️⃣|`PartTree`|Parses method names into parts|
|3️⃣|`PartTreeJpaQuery`|Builds JPQL dynamically|
|4️⃣|`EntityManager`|Executes the query on the DB|

---

## 🪄 Example Flow (Visualization)

Method:

`findByFirstNameAndAgeLessThan("John", 30)`

1. Spring detects repository method.

2. `PartTree` parses → `firstName = ? AND age < ?`

3. `PartTreeJpaQuery` builds Criteria query.

4. `EntityManager` executes:

   `SELECT * FROM user WHERE first_name = 'John' AND age < 30;`

1. Results mapped back to `User`.

------------------------------------------------------------------------------------------


## ⚙️ 2️⃣ What happens at **startup** (Repository creation phase)

When the app starts and Spring scans your repo:

`interface UserRepository extends JpaRepository<User, Long> {     User findByEmail(String email); }`

👉 Spring’s `JpaRepositoryFactory` detects `findByEmail`  
and uses a component called `QueryLookupStrategy`.

---

### 🧠 Step-by-step startup flow:

1. **Spring scans** the interface.

2. It finds methods not implemented in `SimpleJpaRepository` (like `findByEmail`).

3. For each such method, it uses `QueryLookupStrategy` to build a `RepositoryQuery` object.


Depending on your configuration (`create`, `use-declared-query`, or `create-if-not-found`):

- It **tries to find a `@Query` annotation**.

- If not found, it **derives a JPQL query** from the method name.


Example:

`findByEmail(String email) ↓ "select u from User u where u.email = ?1"`

4. This JPQL is wrapped inside a `PartTreeJpaQuery` or `SimpleJpaQuery` instance.

5. These `RepositoryQuery` objects are stored in a **map** inside the proxy’s metadata.


So yes — the query is **parsed and prepared once at startup**, not on every call.

---

## ⚡ 3️⃣ What happens at **runtime** (When you call the derived method)

Now, when you actually do:

`userRepository.findByEmail("john@gmail.com");`

Here’s the exact flow 👇

---

### 🔹 Step 1: Proxy intercepts the method

Spring’s JDK proxy or CGLIB intercepts the call.

It checks:

- “Is this a CRUD method implemented by `SimpleJpaRepository`?”  
  → Then delegate to the target object.

- “Is this a derived query method?”  
  → Then execute using the stored `RepositoryQuery`.


Since `findByEmail` is derived, the **proxy handles it itself**.

---

### 🔹 Step 2: Proxy fetches the prebuilt query metadata

The proxy finds the corresponding `RepositoryQuery` object built during startup (for `findByEmail`).

---

### 🔹 Step 3: Query execution (via `EntityManager`)

The `RepositoryQuery` object executes the query like this:

`Query query = entityManager.createQuery("select u from User u where u.email = ?1"); query.setParameter(1, "john@gmail.com"); return query.getSingleResult();`

That’s it. ✅  
So the **proxy directly uses the `EntityManager`** for execution — it does **not** go through `SimpleJpaRepository`.

---

### 🔹 Step 4: Result is returned to caller

The result (a `User` entity) is returned to the proxy, which passes it back to your code.

---

## 🧠 4️⃣ Key Difference from `save()`

| Aspect          | `save()`                                  | `findByEmail()`                                                 |
|-----------------|-------------------------------------------|-----------------------------------------------------------------|
| Implemented in  | `SimpleJpaRepository`                     | Not implemented                                                 |
| Who executes it | Proxy → SimpleJpaRepository               | Proxy only                                                      |
| At startup      | Just registers CRUD                       | Builds a `RepositoryQuery` via parsing                          |
| At runtime      | Delegates to `SimpleJpaRepository.save()` | Uses prebuilt JPQL & executes via `EntityManager.createQuery()` |
| Query creation  | None (fixed logic)                        | Derived dynamically from method name                            |



How deleteBy... (derived delete) actually works

In Spring Data JPA, a method like:

deleteByName(String name);
Internally it does NOT directly fire:
DELETE FROM user WHERE name = ?

Instead, by default it behaves like:

Step 1: Fetch entities (uses persistence context)
List<User> users = findByName(name);
Step 2: Delete one by one
for (User user : users) {
entityManager.remove(user);
}

👉 This means:

Entities become managed
Then marked as REMOVED
Persistence context knows everything
🔥 So PC does know

Because deletion happens via:

EntityManager.remove()

And that:

Updates persistence context
Tracks entity state
Ensures consistency
⚠️ When does your concern become valid?

When you use bulk delete:

@Modifying
@Query("DELETE FROM User u WHERE u.name = :name")

Now:

Direct SQL runs ❌
Persistence context is bypassed ❌
Existing entities in memory become stale ❌

👉 This is exactly the same problem as bulk UPDATE

Key difference (THIS is the real answer)
Type	Uses Persistence Context?	How delete happens
deleteByName() (derived)	✅ Yes	Fetch + remove()
@Query DELETE (bulk)	❌ No	Direct SQL
Why Spring designed it this way

Because:

JPA is entity lifecycle driven
Not just SQL execution engine

So derived delete:
✔ Keeps consistency
✔ Works with cascades
✔ Triggers entity callbacks

Interview-ready answer

Derived delete queries in Spring Data JPA do use the persistence context. Internally, they first fetch the entities and then call EntityManager.remove() on each, so the persistence context is aware of the removal. Only bulk delete queries using @Query bypass the persistence context.

If you want, I can go one level deeper into:

why fetching before delete is expensive
and
when to prefer bulk delete vs derived delete (very practical + interview combo)
why cant update behave the same way?

Good — this is exactly the right follow-up, and the answer is subtle.

Short answer:

Update can behave the same way — and that’s exactly what save() already does.
But derived queries don’t support it because it would be inefficient, ambiguous, and against JPA’s design.

1. Yes, update CAN behave like delete

What you described is actually this:

List<User> users = findByName(name);
for (User user : users) {
user.setStatus("ACTIVE");
}

With Hibernate:

Entities are managed
Changes tracked via dirty checking
On commit → UPDATE SQL is generated

👉 So:
✔ Uses persistence context
✔ Safe
✔ Consistent

This is the correct JPA way

2. Then why not allow updateByName()?

Because it would secretly do this:

findByName(name)  // SELECT
→ loop
→ multiple UPDATEs
🔴 Problem 1: Performance disaster

Imagine:

updateByStatus("INACTIVE")
10,000 rows match
JPA would:
Fetch all 10,000 rows ❌
Update one by one ❌

👉 Instead of:

UPDATE user SET status='ACTIVE' WHERE status='INACTIVE';
🔴 Problem 2: Ambiguity

What should this mean?

updateByName(String name);
Update what field? ❓
Set to what value? ❓

Derived queries work well for:

conditions (WHERE clause)
But not for:
SET clause
🔴 Problem 3: Misleading abstraction

Derived query name:

updateByName("A")

Looks like:
👉 One DB query

But actually becomes:
👉 SELECT + multiple UPDATEs

❌ Very misleading
❌ Hard to reason about performance

3. Why delete is still allowed?

Because:

deleteByName("A")
Semantics are clear ✅ (just remove)
No ambiguity (no "SET" part)
Even if it fetches, behavior is predictable
4. JPA design philosophy (core reason)

In Spring Data JPA:

Reads → Query-driven (JPQL / derived queries)
Writes → Entity state-driven (save, dirty checking)

👉 Mixing both (like derived update) breaks this model



First: What is JPQL?

JPQL (Java Persistence Query Language) in Spring Data JPA:

Works on entities, not tables
Is executed by providers like Hibernate
🔑 Key idea

JPQL interacts with the persistence context, but does not always fully rely on it like findById() does.

1. SELECT JPQL → Uses persistence context ✅

Example:

@Query("SELECT u FROM User u WHERE u.id = :id")
User getUser(Long id);
What happens internally:
JPQL → converted to SQL
Query executed on DB
Result comes back
Hibernate checks:

👉 “Is this entity already in persistence context?”

YES → return existing managed object ✅
NO → create new managed entity and store it ✅
🔥 Important behavior
User u1 = repo.findById(1L).get();
User u2 = repo.getUser(1L);

👉 u1 == u2 → true (same object)

Because of persistence context identity guarantee

2. JPQL UPDATE / DELETE → Does NOT use persistence context ❌

Example:

@Modifying
@Query("UPDATE User u SET u.name = 'X'")
What happens:
Direct SQL execution
Skips persistence context ❌

👉 So:

Existing managed objects still have old values ❌
Leads to stale data problem



Derived queries → map to JPQL SELECT
There is no JPQL equivalent for derived UPDATE
And JPQL UPDATE itself:
Bypasses persistence context ❌
Is unsafe unless explicitly used




🔑 Two completely different delete behaviors
1. Derived delete (deleteBy...) ✅

In Spring Data JPA:

deleteByName(String name);
How it works:
findByName(name)   // SELECT
→ entityManager.remove(entity)  // per entity

👉 Uses persistence context

Entities become managed
Marked as REMOVED
Lifecycle callbacks triggered
Cascades work
Fully consistent
2. JPQL DELETE ❌ (very different)
   @Modifying
   @Query("DELETE FROM User u WHERE u.name = :name")
   How it works:
   Directly converted to SQL:
   DELETE FROM user WHERE name = ?

👉 No persistence context involvement

No fetching ❌
No remove() ❌
No lifecycle callbacks ❌
No dirty checking ❌
🔥 Critical difference
Feature	Derived delete	JPQL DELETE
Uses persistence context	✅ Yes	❌ No
Fetches entities	✅ Yes	❌ No
Calls remove()	✅ Yes	❌ No
Cascades	✅ Yes	❌ No
Lifecycle callbacks	✅ Yes	❌ No
Performance	❌ Slower	✅ Fast
⚠️ Big problem with JPQL DELETE
User user = repo.findById(1L).get();

repo.deleteByQuery("A"); // JPQL DELETE

👉 Now:

DB row is gone
But user object still exists in memory ❌
Persistence context is stale
Fix for this
@Modifying(clearAutomatically = true)

or manually:

entityManager.clear();
🧠 When to use what?
Use derived delete when:
You need consistency
Cascades matter
Entity callbacks matter
Use JPQL DELETE when:
Large data (bulk delete)
Performance is critical
You handle consistency manually
🎯 Interview-ready answer

No, JPQL DELETE does not work like derived delete queries. Derived delete queries fetch entities and call EntityManager.remove(), so they use the persistence context and maintain consistency. In contrast, JPQL DELETE is a bulk operation that directly executes SQL and bypasses the persistence context, which can lead to stale data in managed entities.

If you want, I can show:

a real bug scenario caused by JPQL DELETE
or how cascade delete behaves differently in both cases (very commonly asked)
why can derived query update behave same as derive query deletby?

It sounds like it should work the same way — “find → loop → apply operation” — but for UPDATE that breaks down for fundamental reasons, not just implementation choice.

Let’s walk it cleanly.

🔑 First: What derived deleteBy is really doing

In Spring Data JPA:

deleteByName("A");

Internally:

findByName("A") → List<User>
for each user → entityManager.remove(user)

👉 Key point:

The operation (DELETE) is fully defined without extra input

🔥 Now try same idea for UPDATE

Imagine Spring allowed:

updateByName("A");

To mimic delete, it would need to do:

findByName("A") → List<User>
for each user → ???

👉 And here is the problem:

❌ Problem 1: UPDATE needs a “SET” clause

DELETE:

deleteByName → clear meaning

UPDATE:

updateByName → update WHAT? to WHAT?

There is no way to express this in a method name.

Even if you try:

updateStatusByName(String name, Status status);

Now Spring must:

Interpret status as SET value
Map it to entity field
Apply it

👉 This becomes:

Complex
Error-prone
Hard to generalize
❌ Problem 2: JPA already solved UPDATE differently

With Hibernate:

User u = repo.findById(id).get();
u.setStatus("ACTIVE");

👉 Hibernate:

Tracks change (dirty checking)
Generates efficient SQL

So:

“find → modify → auto update” already exists

❌ Problem 3: Performance trap (big one)

If derived update behaved like delete:

updateByStatus("INACTIVE", "ACTIVE");

Internally:

findByStatus("INACTIVE")  // loads thousands
→ loop
→ update each

👉 Instead of:

UPDATE user SET status='ACTIVE' WHERE status='INACTIVE';

Huge difference:

❌ N updates vs 1 query
❌ Memory overhead
❌ Slow

✅ How to handle this
Option 1: Auto clear
@Modifying(clearAutomatically = true)

👉 Clears persistence context after query



save() does NOT create JPQL — it directly results in SQL via the ORM.