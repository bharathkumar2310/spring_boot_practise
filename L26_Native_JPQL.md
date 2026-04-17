![img.png](Images/NQ1.png)

![img_1.png](Images/NQ2.png)

![img_2.png](Images/NQ3.png)

![img_3.png](Images/NQ4.png)

![img_4.png](Images/NQ5.png)

![img_5.png](Images/NQ6.png)

![img_6.png](Images/NQ7.png)

![img_7.png](Images/NQ8.png)

![img.png](Images/NQ8'.png)

![img_1.png](Images/NQ9.png)

![img_2.png](Images/NQ10.png)

![img_3.png](Images/NQ11.png)

![img_4.png](Images/NQ12.png)
![img_5.png](Images/NQ13.png)

![img_6.png](Images/NQ14.png)

![img_7.png](Images/NQ15.png)

![img_8.png](Images/NQ16.png)

![img.png](Images/CQ1.png)

![img_1.png](Images/CQ2.png)

![img_2.png](Images/CQ3.png)

![img_3.png](Images/CQ4.png)

![img_4.png](Images/CQ5.png)

![img_5.png](Images/CQ6.png)

![img_6.png](Images/CQ7.png)

![img_7.png](Images/CQ8.png)

![img_8.png](Images/CQ9.png)

![img_9.png](Images/CQ10.png)

![img_10.png](Images/CQ11.png)

![img_11.png](Images/CQ12.png)

![img_12.png](Images/CQ13.png)



🧠 First: What is a Named Native Query?
@NamedNativeQuery(
name = "User.findByEmail",
query = "SELECT * FROM user WHERE email = ?",
resultClass = User.class
)

👉 It’s a predefined SQL query, written once and reused.

🎯 Why do we need it?
🔥 1. When JPQL is NOT enough

JPQL has limitations:

❌ No DB-specific features
❌ Limited functions
❌ No advanced SQL (CTE, window functions, etc.)

👉 Example:

WITH ranked_users AS (...)
SELECT * FROM ranked_users

👉 ❌ Not possible in JPQL
👉 ✅ Use native query

🔥 2. Performance / DB optimization

Sometimes you want:

Index hints
Optimized joins
DB-specific tuning

👉 Only possible with native SQL

🔥 3. Complex Queries

Example:

Multiple joins
Aggregations
Subqueries
Window functions

👉 JPQL becomes messy or impossible

🔥 4. Reusability (Named Query advantage)

Instead of:

@Query(value = "SELECT * FROM user WHERE email = ?", nativeQuery = true)

👉 You define once:

@NamedNativeQuery(name = "User.findByEmail", ...)

👉 Then reuse:

@Query(name = "User.findByEmail", nativeQuery = true)
🔥 5. Separation of Concerns

Keeps SQL outside repository methods:

Cleaner repo interface
Centralized query definition
⚠️ Important: Is it REQUIRED?

👉 ❌ NO — it is NOT required

You can directly write:

@Query(value = "SELECT * FROM user WHERE email = ?", nativeQuery = true)

👉 Named queries are just an alternative



for Native queries:


1. You call method
2. Proxy intercepts call
3. QueryLookupStrategy decides type
4. StringBasedJpaQuery created
5. EntityManager.createNativeQuery(...) called
6. SQL executed

What is a Named Query?

In Spring Data JPA / JPA:

@NamedQuery(
name = "User.updateName",
query = "UPDATE User u SET u.name = :name WHERE u.id = :id"
)

👉 It’s just:

A predefined JPQL query
Stored at startup
Reused later


