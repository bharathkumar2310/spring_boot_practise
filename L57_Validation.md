
@Valid is part of Java Bean Validation (JSR-380 / Jakarta Validation) and is used in Spring to automatically validate incoming data (usually request bodies, query params, or path variables).
👉 Bean Validation (JSR 303 → JSR 380 → Jakarta Validation) is a specification defined by Java/Jakarta
Hibernate Validator is a common implementor


| Category | Annotation         | Purpose                       | Example                       |
| -------- | ------------------ | ----------------------------- | ----------------------------- |
| Core     | `@Valid`           | Triggers validation           | `@Valid Address addr`         |
| Null     | `@NotNull`         | Must not be null              | `@NotNull String name`        |
| Null     | `@Null`            | Must be null                  | `@Null String temp`           |
| Empty    | `@NotEmpty`        | Not null & not empty          | `@NotEmpty List roles`        |
| Empty    | `@NotBlank`        | Not null & not blank (String) | `@NotBlank String name`       |
| Size     | `@Size`            | Size limit                    | `@Size(min=2,max=10)`         |
| Number   | `@Min`             | ≥ value                       | `@Min(18)`                    |
| Number   | `@Max`             | ≤ value                       | `@Max(60)`                    |
| Number   | `@Positive`        | > 0                           | `@Positive int salary`        |
| Number   | `@PositiveOrZero`  | ≥ 0                           | `@PositiveOrZero int exp`     |
| Number   | `@Negative`        | < 0                           | `@Negative int loss`          |
| Number   | `@NegativeOrZero`  | ≤ 0                           | `@NegativeOrZero int balance` |
| Decimal  | `@DecimalMin`      | min decimal                   | `@DecimalMin("10.5")`         |
| Decimal  | `@DecimalMax`      | max decimal                   | `@DecimalMax("100.5")`        |
| Decimal  | `@Digits`          | digits limit                  | `@Digits(5,2)`                |
| String   | `@Email`           | valid email                   | `@Email String email`         |
| String   | `@Pattern`         | regex match                   | `@Pattern(...)`               |
| Boolean  | `@AssertTrue`      | must be true                  | `@AssertTrue boolean ok`      |
| Boolean  | `@AssertFalse`     | must be false                 | `@AssertFalse boolean flag`   |
| Date     | `@Past`            | must be past                  | `@Past LocalDate dob`         |
| Date     | `@PastOrPresent`   | ≤ today                       | `@PastOrPresent`              |
| Date     | `@Future`          | must be future                | `@Future LocalDate event`     |
| Date     | `@FutureOrPresent` | ≥ today                       | `@FutureOrPresent`            |




import jakarta.validation.Valid;
import jakarta.validation.constraints.*;
import java.time.LocalDate;
import java.util.List;

class Address {

    @NotBlank
    String city;

    @Size(min = 5, max = 10)
    String zip;
}

class User {

    // Null / Empty
    @NotNull
    @NotBlank
    String name;

    @Null
    String temporaryField;

    // Size
    @Size(min = 2, max = 5)
    List<@NotBlank String> roles;

    // Number
    @Min(18)
    @Max(60)
    int age;

    @Positive
    int salary;

    @PositiveOrZero
    int experience;

    @Negative
    int loss;

    @NegativeOrZero
    int balance;

    // Decimal
    @DecimalMin("10.5")
    @DecimalMax("1000.5")
    double price;

    @Digits(integer = 5, fraction = 2)
    double accountBalance;

    // String
    @Email
    String email;

    @Pattern(regexp = "^[A-Z]{5}[0-9]{4}[A-Z]$")
    String panNumber;

    // Boolean
    @AssertTrue
    boolean termsAccepted;

    @AssertFalse
    boolean banned;

    // Date
    @Past
    LocalDate dob;

    @PastOrPresent
    LocalDate createdDate;

    @Future
    LocalDate eventDate;

    @FutureOrPresent
    LocalDate bookingDate;

    // Nested Object
    @Valid
    Address address;
}


@PostMapping("/users")
public String createUser(@Valid @RequestBody User user) {
return "Valid User";
}

🔹 High-level idea

When Spring sees:
    
    @PostMapping
    public void create(@Valid @RequestBody User user) {}

👉 It does:

    Convert JSON → User object
    Trigger validation engine
    Validation engine uses reflection + metadata to check constraints
🔹 Step-by-step internal flow

1. Request hits DispatcherServlet
   Entry point of Spring MVC
2. JSON → Object conversion
   Done by HttpMessageConverter (Jackson)
   Now you have a User object
3. Spring detects @Valid
   Spring checks method parameters
   Sees @Valid
   Calls a Validator (Bean Validation API)
4. Validator kicks in (Hibernate Validator)

This is where reflection happens 👇

🔹 How validation actually works internally

✅ Step A: Build metadata (once, cached)

Validator scans the class:
    
    class User {
    @NotBlank
    String name;
    
        @Email
        String email;
    }

👉 Using reflection, it reads:
    
    Fields
    Annotations on fields
    Annotation attributes (like min, max)

It builds something like:
    
    User:
    name → @NotBlank
    email → @Email

👉 This metadata is cached (important for performance)

✅ Step B: Validate object (runtime)

For each field:

Use reflection to get field value:

    field.get(user)
For each annotation:
Find corresponding validator

Example:
    
    @NotBlank → NotBlankValidator
    @Email → EmailValidator
✅ Step C: Call validator logic

Each constraint has a validator:
    
    class NotBlankValidator implements ConstraintValidator<NotBlank, String> {
    
        public boolean isValid(String value, ...) {
            return value != null && !value.trim().isEmpty();
        }
    }

👉 This method is executed

✅ Step D: Collect violations

If invalid: Create ConstraintViolation
        Store:
        field name
        error message
        invalid value
5. If violations exist

Spring throws: MethodArgumentNotValidException



| Feature                 | `@Valid`                   | `@Validated`                                          |
| ----------------------- | -------------------------- | ----------------------------------------------------- |
| Origin                  | Jakarta Bean Validation    | Spring Framework                                      |
| Package                 | `jakarta.validation.Valid` | `org.springframework.validation.annotation.Validated` |
| Validation trigger      | ✅ Yes                      | ✅ Yes                                                 |
| **Validation Groups**   | ❌ Not supported            | ✅ Supported                                           |
| Method-level validation | ❌ No                       | ✅ Yes                                                 |
| Use in Spring           | Basic validation           | Advanced validation                                   |



🔹 🔥 KEY FEATURE: Validation Groups

This is the real reason @Validated exists

🔸 Example: Different rules for Create vs Update
    
    interface CreateGroup {}
    interface UpdateGroup {}
    class User {
    
        @NotNull(groups = UpdateGroup.class)
        private Long id;
    
        @NotBlank(groups = CreateGroup.class)
        private String name;
    }

Controller:
    
    @PostMapping("/create")
    public void create(@Validated(CreateGroup.class) @RequestBody User user) {}
    
    @PutMapping("/update")
    public void update(@Validated(UpdateGroup.class) @RequestBody User user) {}

👉 Now:

Create → name required
Update → id required


🔹 Method-Level Validation (BIG difference 🔥)
❌ @Valid does NOT work here:
    
    public void process(@Min(1) int id) {} // ❌ ignored

    ✅ @Validated works:
    @Validated
    @Service
    class UserService {
    
        public void process(@Min(1) int id) {}
    }
    
    👉 Now validation is applied to method parameters