


Once Spring has:

`SecurityContextHolder.getContext().setAuthentication(authentication);`

this means **Spring now officially knows who the user is** for this request.

From that moment on, **every filter, controller, and service** in that request can access this `Authentication` object to know:

- who the user is (`getName()` / `getPrincipal()`)

- what roles they have (`getAuthorities()`)

- whether they are authenticated (`isAuthenticated()`)


---

Let’s trace what happens _after that point._

---

## ⚙️ 1️⃣ The Security Filter Chain continues

After the filter that performed authentication (e.g. `UsernamePasswordAuthenticationFilter` or your `JwtAuthenticationFilter`) sets the authentication into the `SecurityContext`,  
control passes to the **next filters** in the chain.

Example flow (simplified):
```
SecurityContextPersistenceFilter
↓
JWTAuthenticationFilter / UsernamePasswordAuthenticationFilter
↓
ExceptionTranslationFilter
↓
FilterSecurityInterceptor
↓
Controller

```


Now the upcoming filters (especially `FilterSecurityInterceptor`) will **use the Authentication** from the `SecurityContext`.

---

## 🧩 2️⃣ FilterSecurityInterceptor — the Authorization Gatekeeper

This filter is the **last checkpoint** before your controller executes.

Its job:

- Read the `Authentication` from the `SecurityContext`

- Check what authorities (roles) the user has

- Compare them with what the endpoint requires (defined in your `HttpSecurity` config or annotations)


Example:
```
.authorizeHttpRequests()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated();

```

Internally, `FilterSecurityInterceptor` → calls → `AccessDecisionManager`.

---

## ⚖️ 3️⃣ AccessDecisionManager — makes the access decision

Spring’s `AccessDecisionManager` receives:

- The `Authentication` (from the context)

- The target resource (e.g., `/admin`)

- The access attributes (required roles)


It loops through all `AccessDecisionVoter`s to check permissions.

Pseudocode:

```
for (AccessDecisionVoter voter : voters) {
    int result = voter.vote(auth, object, attributes);
    if (result == ACCESS_GRANTED) return;
}
throw new AccessDeniedException("Forbidden");

```

If any voter denies → user gets **403 Forbidden**.

---

## 🧱 4️⃣ If authorized → Controller executes

If all voters grant access, `FilterSecurityInterceptor` lets the request proceed to your controller or REST endpoint.

Now in your controller, you can directly access user details:

```
@GetMapping("/me")
public String me(Authentication auth) {
    return auth.getName(); // current logged-in username
}

```
or

```
@GetMapping("/me")
public String me() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    return auth.getName();
}

```
---

## 🧠 5️⃣ Inside the Application (Controllers / Services)

Anywhere in your application, you can read from the `SecurityContext`:
```
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
Collection<? extends GrantedAuthority> roles = auth.getAuthorities();

```

This means your **business logic** can use the same context to make domain-level access decisions (like “only the owner can edit their post”).

---

## 🧹 6️⃣ After request completion → SecurityContext is cleared

At the end of every request, the first filter (`SecurityContextPersistenceFilter`) does cleanup:

- It ensures the `SecurityContext` for this request thread is cleared.

- In session-based security: it also stores the context in the HTTP session.

- In stateless JWT: it simply clears it (since there’s no session).


Code (conceptually):

```
try {
    filterChain.doFilter(request, response);
} finally {
    SecurityContextHolder.clearContext();
}

```

This prevents **user data leaking** between threads.

![img.png](Images/auth1.png)

![img_1.png](Images/auth2.png)

![img_2.png](Images/auth3.png)

![img_3.png](Images/auth4.png)

![img_4.png](Images/auth5.png)

![img_5.png](Images/auth6.png)

![img_6.png](Images/auth7.png)

![img_7.png](Images/auth8.png)

![img_8.png](Images/auth9.png)

![img_9.png](Images/auth10.png)

![img_10.png](Images/auth11.png)

![img_11.png](Images/auth12.png)

![img_12.png](Images/auth13.png)

![img_13.png](Images/auth14.png)

![img_14.png](Images/auth15.png)

![img_15.png](Images/auth16.png)



|Type|Applies To|Mechanism|Framework Layer|
|---|---|---|---|
|**Web Interceptor / Filter**|HTTP requests before controller|Servlet Filter Chain / HandlerInterceptor|Spring MVC|
|**AOP Proxy Interceptor**|Method calls on beans|Spring AOP (Proxy + MethodInterceptor)|Service / Repository Layer|


## ⚖️ Core Difference (in one line)

|Concept|**HandlerInterceptor**|**MethodInterceptor**|
|---|---|---|
|**Used in**|Spring MVC (Web Layer)|Spring AOP (Service/Business Layer)|
|**Intercepts**|HTTP request before and after controller method|Method invocation on a Spring bean|
|**Goal**|Pre-/post-processing of web requests|Add cross-cutting logic around method calls|
|**Entry Point**|`DispatcherServlet` pipeline|Proxy object created by Spring AOP|

---

## 🧩 1️⃣ What is a **HandlerInterceptor**

- It intercepts **HTTP requests** handled by controllers.

- Works at the **MVC layer**, before/after controller execution.


### 🔹 Used for:

✅ Logging  
✅ Authentication (JWT, API Key)  
✅ Request validation  
✅ Timing / metrics

### 🔹 Example:

```
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("Before controller: " + request.getRequestURI());
        return true; // continue
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        System.out.println("After controller, before view rendering");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {
        System.out.println("After complete request");
    }
}

```

### 🔹 Lifecycle:

`Client → Filter → HandlerInterceptor → Controller → View`

### 🔹 Registration:

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor());
    }
}

```
---

## 🧠 2️⃣ What is a **MethodInterceptor**

- It intercepts **method calls** made on Spring beans.

- Works at the **AOP layer**, usually in service or repository classes.


### 🔹 Used for:

✅ Transaction management (`@Transactional`)  
✅ Security (`@PreAuthorize`)  
✅ Caching (`@Cacheable`)  
✅ Logging, metrics, retry logic

### 🔹 Example:

```
public class LoggingInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: " + invocation.getMethod().getName());
        Object result = invocation.proceed();  // call actual method
        System.out.println("After: " + invocation.getMethod().getName());
        return result;
    }
}

```

### 🔹 Lifecycle:

```
public class LoggingInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: " + invocation.getMethod().getName());
        Object result = invocation.proceed();  // call actual method
        System.out.println("After: " + invocation.getMethod().getName());
        return result;
    }
}

```
### 🔹 Configuration:

Spring automatically wires `MethodInterceptor`s when you use annotations like:

- `@Transactional` → `TransactionInterceptor`

- `@PreAuthorize` → `MethodSecurityInterceptor`


So you rarely register them manually.


--------------------------------------------------------------------------------------------


### Step 1: You enable method security

In your config:

`@EnableMethodSecurity public class SecurityConfig {}`

This tells Spring to create **AOP proxies** around beans that use security annotations like `@PreAuthorize`.

---

### Step 2: Spring wraps your bean in a proxy

For example:

`UserService → wrapped in a proxy by MethodSecurityAdvisor`

The proxy adds a **MethodInterceptor** called `MethodSecurityInterceptor`.

So internally:

`UserServiceProxy → MethodSecurityInterceptor → Real UserService`

---

### Step 3: Interception flow

When you call:

`userService.deleteUser(123L);`

Here’s what happens:

```
Proxy intercepts call
   ↓
MethodSecurityInterceptor (MethodInterceptor)
   ↓
ExpressionBasedPreInvocationAdvice (evaluates @PreAuthorize)
   ↓
If passes → invoke real method
   ↓
If fails → throw AccessDeniedException

```

For `@PostAuthorize`, the interceptor adds another step _after_ the method:

```
Invoke real method
   ↓
Get return value
   ↓
Evaluate post-authorization expression
   ↓
If passes → return value
If fails → throw AccessDeniedException

```