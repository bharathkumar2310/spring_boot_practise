![img.png](Images/fa1.png)

![img_1.png](Images/fa2.png)

![img_2.png](Images/fa3.png)

![img_3.png](Images/fa4.png)

![img_4.png](Images/fa5.png)

![img_5.png](Images/fa6.png)

![img_6.png](Images/fa7.png)

![img_7.png](Images/fa8.png)

🧠 What happens when you use `spring-session-jdbc`

When you include this dependency:

`<dependency>   <groupId>org.springframework.session</groupId>   <artifactId>spring-session-jdbc</artifactId> </dependency>`

Spring Boot automatically switches session storage from **in-memory (Tomcat)** to **JDBC** — i.e., it now stores session data in a **database table** instead of server memory.

---

## 🗄️ So… which database?

It uses **the same datasource** you configured for your application.

✅ That means:  
If your `application.properties` has something like:

`spring.datasource.url=jdbc:mysql://localhost:3306/myapp spring.datasource.username=root spring.datasource.password=pass`

Then the **session tables** are also created and stored in that same `myapp` database.

---

## 🧩 Tables Created by Spring Session JDBC

Spring will create two main tables (automatically if you enable schema initialization):

1. **`SPRING_SESSION`**

2. **`SPRING_SESSION_ATTRIBUTES`**


You can also create them manually using the schema files included in the library, for example:

![img_8.png](Images/fa9.png)

![img_9.png](Images/fa10.png)

![img_10.png](Images/fa11.png)

![img_11.png](Images/fa12.png)

![img_12.png](Images/fa13.png)

👉 `HttpSecurity`

is a **builder object** provided by Spring Security  
that lets you configure **how HTTP requests are secured** —  
that means authentication, authorization, CSRF, sessions, filters, etc.



## 3️⃣ What `HttpSecurity` actually does internally

It’s basically a **builder for filters**.

- Every time you call something like `.formLogin()`, `.authorizeHttpRequests()`, or `.csrf()`,  
  you’re telling Spring Security:  
  “Add this specific filter or configuration into the filter chain.”


### Example

```
http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
    )
    .formLogin(withDefaults())
    .logout(withDefaults());

```

✅ The **cookie is sent in the **same HTTP response** that Spring Security sends **after successful login** (usually a redirect).