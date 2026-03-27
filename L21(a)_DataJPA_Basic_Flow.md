
# ⚙️ COMPLETE INTERNAL FLOW OF `save()`

---

## 🧩 1️⃣ Application Startup Phase (Wiring Repositories)

Before `save()` is even called, Spring sets up a huge amount of infrastructure:

### a. Repository Interface Scanning

When the app starts:

- `@EnableJpaRepositories` triggers `JpaRepositoriesRegistrar`.

- It scans all interfaces extending `JpaRepository`, `CrudRepository`, etc.


Example:

`public interface UserRepository extends JpaRepository<User, Long> {}`

Spring generates a **proxy bean** for this interface.

---

### b. Repository Bean Creation

For each interface:

- Spring creates a **dynamic proxy** via **`JpaRepositoryFactoryBean`**.

- This factory creates a `JpaRepositoryFactory` which produces the actual implementation — **`SimpleJpaRepository`**.


So at runtime:

`userRepository → proxy → SimpleJpaRepository`

---

## 🧩 2️⃣ You Call `userRepository.save(user)`

Now the actual flow begins.

You call from your service or controller:

`userRepository.save(user);`

Behind the scenes:

1. The call hits the **proxy**.

2. Proxy delegates to the actual implementation class: **`SimpleJpaRepository`**.

3. The implementation calls the `EntityManager` internally.


---

## 🧱 3️⃣ Inside `SimpleJpaRepository.save()`

The real method looks like this:

`@Transactional public <S extends T> S save(S entity) {     if (entityInformation.isNew(entity)) {         entityManager.persist(entity);         return entity;     } else {         return entityManager.merge(entity);     } }`

So `save()` checks:

- Is this entity new (`isNew = true`) → call `persist()`

- Else → call `merge()`


---

## 🧩 4️⃣ How does `isNew()` work?

Spring uses:

`JpaEntityInformation.isNew(entity)`

This inspects:

- The entity’s `@Id` field:

    - If `null` → **new entity**

    - If has a value → **might be existing**

- Or uses `@Version` for optimistic locking in some cases.


✅ So new entities → `persist()`,  
🌀 Detached/existing entities → `merge()`.

---

## 🧩 5️⃣ When `persist()` is called (New Entity)

If the entity is new:

`entityManager.persist(entity);`

Internally:

- Hibernate gets the entity’s metadata (`EntityPersister`).

- Adds it to the **Persistence Context** (first-level cache).

- Marks it as **managed**.

- Schedules an **INSERT** statement to be executed at flush time.


No SQL yet — Hibernate just queues it.

---

### Example:

`User u = new User(null, "John", 30); userRepository.save(u);`

→ `persist(u)`  
→ Added to Persistence Context  
→ On flush:

`INSERT INTO user (name, age) VALUES ('John', 30);`

---

## 🧩 6️⃣ When `merge()` is called (Existing Entity)

If `entityInformation.isNew()` is false:

`entityManager.merge(entity);`

- Hibernate looks for an existing entity in the Persistence Context with the same ID.

- If found → copies fields from given entity to that managed one.

- If not found → fetches it from DB or creates a new managed copy.

- The merged instance is returned and tracked.

- Hibernate schedules an **UPDATE** at flush time if any field changed.


---

### Example:

`User u = new User(1L, "John", 32); userRepository.save(u);`

→ `merge(u)`  
→ Managed copy created  
→ On flush:

`UPDATE user SET name='John', age=32 WHERE id=1;`

---

## 🧩 7️⃣ What Happens During @Transactional

Remember, `save()` is annotated with `@Transactional`.

When your method (like in a Service) calls `save()`, Spring:

1. Starts a **transaction** via `PlatformTransactionManager`.

2. Creates a **Persistence Context (EntityManager)**.

3. Binds it to the current thread.


So now:

- All operations (`persist`, `merge`) are tracked.

- SQL is **not** immediately sent.

- On **commit**, the context is **flushed**.


---

## 🧩 8️⃣ Flush Phase (Commit Time)

When the transaction is about to commit:

1. Hibernate’s `Session` (EntityManager implementation) calls `flush()`.

2. During flush:

    - Hibernate compares current managed entities vs their snapshot.

    - Finds what changed → generates appropriate SQL.

3. SQL statements (INSERT/UPDATE/DELETE) are executed.

4. The Persistence Context is then cleared or closed.


---

### Example Flow:

`@Transactional public void saveUser() {     User u = new User(null, "Alice", 20);     userRepository.save(u);     u.setAge(21); }`

✅ At the end of the transaction:

- Hibernate sees the entity `u` changed (`age` modified).

- Generates **one single INSERT** (not both).

  `insert into user (name, age) values ('Alice', 21);`


Because it does **dirty checking** before flush.

---

## 🧩 9️⃣ Dirty Checking Mechanism

Hibernate internally keeps a **snapshot** (original copy) of every managed entity.

When you modify a managed entity:

- It compares current values vs snapshot.

- If differences found → marks entity as **dirty**.

- During flush, generates appropriate `UPDATE`.


That’s why you don’t need to call `save()` again after changing a managed entity inside a transaction.

---

## 🧩 🔄 Flow Summary Table

|Step|Layer|Class / Component|What Happens|
|---|---|---|---|
|1|Spring Boot|`@EnableJpaRepositories`|Repository scanning and proxy creation|
|2|Repository|`JpaRepositoryFactoryBean` → `SimpleJpaRepository`|Real implementation of your interface|
|3|App Call|`userRepository.save(user)`|Method hits proxy → SimpleJpaRepository|
|4|Decision|`isNew()` check|Based on ID or version|
|5|New Entity|`EntityManager.persist()`|Adds entity to Persistence Context|
|6|Existing Entity|`EntityManager.merge()`|Merges state to managed copy|
|7|Transaction|`@Transactional`|Binds Persistence Context to thread|
|8|Flush|Hibernate Session|Executes queued SQL statements|
|9|Commit|TransactionManager|Commits DB transaction, clears context|

---

## 🧠 Visualization of Flow

`[Controller]      ↓ [Service Layer]      ↓ (calls) UserRepository.save(user)      ↓ [Proxy Repository Bean]      ↓ SimpleJpaRepository.save()      ↓ EntityManager.persist() or merge()      ↓ Hibernate Persistence Context      ↓ Flush on Commit → SQL INSERT/UPDATE      ↓ Database`