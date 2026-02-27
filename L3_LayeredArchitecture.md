# Layered Architecture – Complete & Presentable Notes



## 1. What is Layered Architecture?

Layered Architecture is a **software design pattern** that organizes an application into **horizontal layers**, where each layer has a **well-defined responsibility**.

Each layer:

* Focuses on a single concern
* Communicates with adjacent layers
* Hides its internal implementation

Think of it like a **multi-floor building 🏢**:

* Top floors → User interaction
* Middle floors → Business rules
* Bottom floors → Data storage

---

## 2. Goals of Layered Architecture

* Separation of concerns
* Loose coupling
* High cohesion
* Maintainability
* Testability
* Scalability

---

## 3. Standard Layers (High Level)

```
Presentation Layer
Business / Service Layer
Utility Layer
Configuration Layer
Persistence / Data Access Layer
Database Layer
```

---

## 4. Presentation Layer (Controller / UI Layer)

### Responsibility

* Handles **user requests**
* Performs **request validation**
* Converts input/output to DTOs
* Delegates work to Service layer

### Examples

* REST Controllers (Spring MVC)
* Web UI (React, JSP)
* Mobile UI

### Spring Example

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    @GetMapping("/{id}")
    public OrderDTO getOrder(@PathVariable Long id) {
        return orderService.getOrder(id);
    }
}
```

### Should NOT

* Contain business logic
* Access database directly

---

## 5. Business Layer (Service Layer)

### Responsibility

* Core **business logic**
* Validation rules
* Transaction management
* Orchestrates repositories & utilities

### Characteristics

* Stateless
* Reusable
* Testable

### Spring Example

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    @Transactional
    public OrderDTO getOrder(Long id) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new BusinessException("Order not found"));
        return OrderMapper.toDTO(order);
    }
}
```

📌 **Most critical layer of the application**

---

## 6. Utility Layer (Helper / Common Layer)

### What is Utility Layer?

The Utility layer contains **reusable, stateless helper logic** used across multiple layers.

### Responsibility

* Common helper methods
* Formatting
* Date/time handling
* Calculations
* Mapping logic
* Validation helpers

### Examples

* String utilities
* Date utilities
* File handling
* Encryption helpers
* Mappers (DTO ↔ Entity)

### Example

```java
public class DateUtil {

    public static LocalDateTime nowUTC() {
        return LocalDateTime.now(ZoneOffset.UTC);
    }
}
```

### Rules

* No business rules
* No database access
* Should be **stateless**

---

## 7. Configuration Layer

### What is Configuration Layer?

Contains application-wide **configuration and wiring logic**.

### Responsibility

* Bean definitions
* External configuration
* Security setup
* Caching setup
* API clients configuration

### Examples

* Spring `@Configuration` classes
* `application.yml / application.properties`
* Security configuration

### Spring Example

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

### application.yml Example

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/app_db
```

---

## 8. Persistence Layer (DAO / Repository Layer)

### Responsibility

* Interacts with database
* Executes CRUD operations
* Maps objects to tables

### Examples

* DAO classes
* Spring Data JPA Repositories

### Spring Example

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

### Should NOT

* Contain business logic
* Handle HTTP requests

---

## 9. Database Layer

### Responsibility

* Data storage
* Indexing
* Constraints
* Transactions

### Examples

* MySQL
* PostgreSQL
* Oracle
* MongoDB

---

## 10. DTOs in Layered Architecture

### What is DTO?

DTO (Data Transfer Object) is used to transfer data **between layers**.

### Why DTOs?

* Avoid exposing entities
* Reduce payload size
* Improve security
* API flexibility

### Example

```java
public class OrderDTO {
    private Long id;
    private Double price;
}
```

---

## 11. Entity vs DTO vs Utility Model

| Type    | Purpose               |
| ------- | --------------------- |
| Entity  | Database mapping      |
| DTO     | Data transfer         |
| Utility | Reusable helper logic |

---

## 12. Layer Communication Rules

### Strict Layering (Recommended)

```
Controller → Service → Repository → Database
```

### Relaxed Layering

```
Controller → Service
Controller → Repository ❌ (Not recommended)
```

---

## 13. Exception Handling by Layer

| Layer      | Exception Type      |
| ---------- | ------------------- |
| Controller | HTTP / Validation   |
| Service    | BusinessException   |
| Repository | DataAccessException |

---

## 14. Transaction Management

* Managed in **Service Layer**

```java
@Transactional
public void transferMoney(Long from, Long to) {
    debit(from);
    credit(to);
}
```

---

## 15. Testing Strategy

### Unit Testing

* Controller → Mock Service
* Service → Mock Repository
* Utility → Pure unit tests

### Integration Testing

* Repository + Database

---

## 16. When to Use Layered Architecture?

✅ Enterprise applications
✅ Spring Boot / Java EE projects
✅ Monolithic systems

❌ Very small applications

---

## 17. Comparison with Other Architectures

| Architecture       | Key Difference       |
| ------------------ | -------------------- |
| MVC                | UI-focused pattern   |
| Clean Architecture | Dependency inversion |
| Hexagonal          | Ports & adapters     |
| Microservices      | Distributed systems  |

---

## 18. Sample Project Structure (Spring Boot)

```
com.example.app
 ├── controller
 ├── service
 ├── repository
 ├── utility
 ├── config
 ├── dto
 ├── entity
 └── exception
```

---

## 19. Common Interview Questions

1. Why Service layer is important?
2. Difference between Utility and Service?
3. Where do you manage transactions?
4. Can Controller access Repository?
5. DTO vs Entity?

---

## 20. Key Takeaways 🔑

* Layered Architecture promotes clean code
* Service layer holds business rules
* Utility layer is reusable & stateless
* Configuration centralizes setup
* Strict layering improves maintainability

---

✅ **This document is ready to use as `.md` notes for study, GitHub, or documentation.**
