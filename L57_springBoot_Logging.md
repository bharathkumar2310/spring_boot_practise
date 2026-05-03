🧠 1. What is logging in Spring?

In Spring Boot, logging is used to:
    
    Track application flow
    Debug issues
    Monitor production behavior

Spring Boot uses:
    
    👉 SLF4J (API)
    👉 Logback (default implementation)

🔥 2. Log levels (VERY IMPORTANT)

You must know these clearly:
    
    Level	Use
    TRACE	Very detailed (rare)
    DEBUG	Debugging
    INFO	Normal flow
    WARN	Something unexpected
    ERROR	Failure


Example:
    
    private static final Logger log = LoggerFactory.getLogger(MyClass.class);
    
    log.info("User created");
    log.debug("User details: {}", user);
    log.error("Exception occurred", e);

⚙️ 3. Configuration (application.properties)
    
    logging.level.root=INFO
    logging.level.com.myapp=DEBUG

👉 Controls logging per package

📁 4. Log output configuration
    
    Console + File
    logging.file.name=app.log

👉 Logs get written to file

🔥 5. Log pattern (basic idea)
    
    logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n

👉 Controls format

🚀 6. Actuator + Logging (VERY IMPORTANT)

With Spring Boot Actuator:

👉 You can change log level at runtime
    
    POST /actuator/loggers/com.myapp
    {
    "configuredLevel": "DEBUG"
    }

🔥 No restart required

🧠 7. Best practices (INTERVIEW GOLD)
    
    ✅ Use correct log levels
    Don’t use ERROR for everything ❌
    Don’t use DEBUG in production logs ❌
    ✅ Avoid sensitive data
    
    ❌ Never log:
    
    Passwords
    Tokens
    Personal data
    ✅ Use placeholders (important)
    log.info("User {} logged in", userId);
    
    ✔ Efficient
    ❌ Avoid string concatenation


🔥 8. Exception logging (very important)
try {
// code
} catch (Exception e) {
log.error("Error occurred", e);
}

👉 Always pass exception object


🧠 What is @Slf4j?

@Slf4j is from Project Lombok and is used to automatically create a logger instance for your class.

It saves you from writing boilerplate code.

🔥 Without @Slf4j

Normally, you write:

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyService {
private static final Logger log = LoggerFactory.getLogger(MyService.class);

    public void process() {
        log.info("Processing...");
    }
}
⚡ With @Slf4j
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class MyService {

    public void process() {
        log.info("Processing...");
    }
}

👉 Lombok automatically generates:

    private static final Logger log = LoggerFactory.getLogger(MyService.class);
🧠 What logger does it use?

    It uses SLF4J, which is the standard logging API in Java.

🔥 Why is it useful?
    
    ✔ Removes boilerplate
    ✔ Cleaner code
    ✔ Standard logging approach
    ✔ Widely used in Spring Boot projects

⚠️ Important notes
    
    1. Lombok dependency required
       <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       </dependency>
       2. Works at compile time
    
    👉 Lombok generates the logger during compilation
    👉 You won’t see it in source code, but it exists in bytecode
    
    3. IDE plugin needed

For IntelliJ/Eclipse → install Lombok plugin
Otherwise, errors may show

TRACE < DEBUG < INFO < WARN < ERROR

🔥 So what about log.debug()?
Case 1: Log level = DEBUG
logging.level.root=DEBUG

✔ log.debug() → ✅ printed
✔ log.info() → ✅ printed
✔ log.error() → ✅ printed