1. What is Spring Boot and why was it created?

       SpringBoot is a framework built on top of Spring to reduce further boilerplate codes

        BuiltIn web server 

                    With Spring we need to manually deploy jar to external web server
                    But with spring it provides inbuilt web server

        Autoconfiguration
                    
                    With Spring say if we need to build rest endpoint and connect with Db we need to configure 
                    dispatcher servlet, datasource etc manually but with SpringBoot based on dependency it autocnfigures them by itself
                    We can customize the config or use the same


        Starter dependencies
                    
                    With spring if u need to build web service u need to add springmvc, jackson, and so on dependencies
                    But with springboot it provides a starter web dpe(single dep) which is a colection of multiple dependencies like tomcat, springmvc, jackson etc
       
        Spring Actuators

                    Spring Boot makes application production ready with the help of actuators whcih helps in monitoring and managing app which is npt provided by Spring

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

2. What is Auto-Configuration?

Spring Boot automatically configures beans based on:

    classpath
    properties
    existing beans

👉 Example:
If spring-boot-starter-web is present → Tomcat + DispatcherServlet auto-configured.

👉 Internal:
Uses:

@EnableAutoConfiguration
spring.factories (or newer AutoConfiguration.imports)

🔹  Entry Point – @SpringBootApplication

When you run your app, this annotation pulls in:
    
    @EnableAutoConfiguration ✅ (this is the key)
    @ComponentScan
    @Configuration

So the real story starts with @EnableAutoConfiguration.

🔹 How Spring finds auto-config classes
        
        Spring Boot uses a mechanism called: 👉 SpringFactoriesLoader
        It scans this file inside dependencies: META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
        
        Earlier (pre Spring Boot 3), it used:META-INF/spring.factories
        
        This file contains a list of auto-configuration classes, like:
            
            DataSourceAutoConfiguration
            HibernateJpaAutoConfiguration
            WebMvcAutoConfiguration

🔹 These classes are NOT blindly applied

        Every auto-config class is full of conditions like:
                
                @ConditionalOnClass
                @ConditionalOnMissingBean
                @ConditionalOnProperty
                @ConditionalOnWebApplication
        
        👉 This is the core idea:
        Spring Boot only applies configuration if conditions match your project

🔹 Example Flow (very important for interviews)

Say you added:
        
        spring-boot-starter-data-jpa
        What happens internally:
        Spring Boot sees HibernateJpaAutoConfiguration
        It checks:
        @ConditionalOnClass(EntityManager.class)
        
        ✔ If JPA is on classpath → continue
        
        @ConditionalOnMissingBean
        
        ✔ If you didn’t define your own bean → continue
        
        It auto-creates:
        DataSource
        EntityManagerFactory
        TransactionManager
        
        👉 That’s why you don’t write boilerplate config

🔹Condition Evaluation Engine
    
    Internally, Spring Boot uses:
    
    👉 ConditionEvaluator
            
            It evaluates all @Conditional annotations and decides:
            
            Apply config ✅
            Skip config ❌
🔹 Order Matters
        
        Auto-config classes are applied in order using:
        
        @AutoConfigureAfter
        @AutoConfigureBefore
        
        This avoids conflicts like:
        
        DataSource must be ready before JPA
🔹 How it avoids overriding YOUR config

Very important concept 👇
        
        @ConditionalOnMissingBean
        
        👉 If you define your own bean → Spring Boot backs off
        
        So:
        
        You override → Boot respects you
        You don’t define → Boot helps you

    How does Spring Boot avoid loading all jars?


        Spring Boot uses a metadata optimization file:
        
            META-INF/spring-autoconfigure-metadata.properties
        
        This file contains extracted info about conditions like:
        
            org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration.ConditionalOnClass=javax.sql.DataSource
        
        👉 Important:
        
        This metadata is generated at build time
        It allows Spring Boot to check conditions WITHOUT loading the class 
        The Trick: Pre-filtering (without class loading)
        
        Before loading any auto-config class:
        
            Step 1: Read metadata (cheap, no class loading)
        
        Spring Boot checks:
            
            @ConditionalOnClass
            @ConditionalOnMissingClass
        
        Using string-based class name checks (via ClassLoader, not full class load)
        
        👉 Example:
        
        @ConditionalOnClass(DataSource.class)
        
        Spring Boot checks:
        
        Is "javax.sql.DataSource" present in classpath?
        
        ✔ If NO → skip loading this auto-config class entirely
        ✔ If YES → proceed
        
        4. When are annotations actually evaluated?
        
        Only after a class passes metadata filtering:
        
        Step 2: Load the class
        Step 3: Evaluate full conditions
        
        Now Spring evaluates:
        
        @ConditionalOnBean
        @ConditionalOnProperty
        @ConditionalOnMissingBean
        custom conditions
        
        👉 These require actual context → so class must be loaded


-----------------------------------------------------------------------------------------------------------------------------------------

3. How does Spring Boot know what to configure?

        Through: @Conditional annotations

Examples:
                            
    @ConditionalOnClass
    @ConditionalOnMissingBean
    @ConditionalOnProperty

👉 This is how Boot avoids overriding your custom beans.




-----------------------------------------------------------------------------------------------------------------------------------

4. What are Starters?

        Predefined dependency bundles.

Example:
    
    spring-boot-starter-web
    spring-boot-starter-data-jpa

👉 Why useful?
    
    Avoids dependency conflicts.


--------------------------------------------------------------------------------------------------------------------------------------

5. Difference between @Component, @Service, @Repository?
    
        👉 To tell Spring: “create an object (bean) of this class and manage it.”
        This is called component scanning + dependency injection.

All are same internally (component scanning), but:
    
    @Service → business logic
    @Repository → adds exception translation
    @Component → generic

-------------------------------------------------------------------------------------------------------------------------------------


6. What is Dependency Injection?

        Spring injects dependencies instead of you creating them.

👉 Types:
    
    Constructor (preferred)
    Field (bad practice)
    Setter

👉 Why constructor?
    
    Immutable
    testable

-----------------------------------------------------------------------------------------------------------------------------------

7. What is ApplicationContext?

Core container managing:

beans
lifecycle
dependency injection

👉 Types:

AnnotationConfigApplicationContext
WebApplicationContext

-----------------------------------------------------------------------------------------------------------------------------------

8. What happens when Spring Boot app starts?

Flow:

    Main Method Execution
        The application starts from the main() method.
    SpringApplication.run() is called to bootstrap the app.
        SpringApplication Bootstrapping
        Creates and configures a SpringApplication instance.
        Determines application type (Servlet / Reactive / Non-web).
    Environment Preparation
        Loads configuration from:
            application.properties / application.yml
            OS environment variables
            Command-line arguments
            Resolves active profiles
            Prepares a ConfigurableEnvironment
    ApplicationContext Creation
        Creates the IoC container (ApplicationContext)
        Type depends on app:
        Web → AnnotationConfigServletWebServerApplicationContext
        Apply Initializers & Listeners
        ApplicationContextInitializer runs
        SpringApplicationRunListeners get triggered
        Load Bean Definitions
        From:
        @Configuration classes
        @SpringBootApplication

@SpringBootApplication is a combination of:

@Configuration
@EnableAutoConfiguration
@ComponentScan

🚀 Continue the Flow

Auto-Configuration
Triggered by @EnableAutoConfiguration

Reads from:
        
        META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
        Applies configurations conditionally using:
        @ConditionalOnClass
        @ConditionalOnMissingBean
        @ConditionalOnProperty

Component Scanning
    
    Scans base package and sub-packages
        Detects:
        @Component
        @Service
        @Repository
        @Controller
    Registers them as beans

Bean Creation & Dependency Injection
        
        Spring creates beans
        Resolves dependencies using:
        Constructor injection (preferred)
        Field / Setter injection

Embedded Server Startup
If web app:
    
    Starts embedded server like:
    Tomcat (default)
    Jetty / Undertow
    Deploys application inside it

Application Ready
ApplicationRunner and CommandLineRunner execute
App is ready to serve requests


🔹 1. CommandLineRunner
📌 Definition

    An interface used to run code after startup, receiving raw command-line arguments.

📌 Method

    void run(String... args)

📌 Example
    
    @Component
    public class MyRunner implements CommandLineRunner {
    @Override
    public void run(String... args) {
    System.out.println("App started with args: " + Arrays.toString(args));
    }
    }
📌 Input Style

If you run:

    java -jar app.jar hello world

Then:

    args = ["hello", "world"]

🔹 2. ApplicationRunner
📌 Definition

    Similar to CommandLineRunner, but provides a more structured way to access arguments.

📌 Method

    void run(ApplicationArguments args)

📌 Example
    
    @Component
    public class MyAppRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) {
    System.out.println("Option Names: " + args.getOptionNames());
    System.out.println("Non-option args: " + args.getNonOptionArgs());
    }
    }
🔥 Key Difference (Interview Gold)

| Feature             | CommandLineRunner | ApplicationRunner          |
| ------------------- | ----------------- |----------------------------|
| Argument Type       | String[]          | ApplicationArguments       |
| Structured Parsing  | ❌ No              | ✅ Yes                   | 
| Supports named args | ❌ No              | ✅ Yes (`--name=value`)     |



------------------------------------------------------------------------------------------------------------------------------

9. What is Embedded Server?

Spring Boot includes:
    
    Tomcat (default)
    Jetty
    Undertow

👉 Why?

    No need for external deployment (WAR).

--------------------------------------------------------------------------------------------------------------------------

10. What is @SpringBootApplication?

Combination of:
    
    @Configuration
    @EnableAutoConfiguration
    @ComponentScan

--------------------------------------------------------------------------------------------------------------------------

11. What is Component Scanning?
        
        Component Scanning is the process by which Spring automatically detects classes annotated with stereotypes like @Component, @Service, @Repository, and @Controller, 
        and registers them as beans in the Spring IoC container.

👉 Default:

    Same package + subpackages

Base Package Determination

       Spring starts scanning from the package where your main class is located.
       It scans that package + all sub-packages

📌 Example:
        
        com.app
        ├── controller
        ├── service
        ├── repo

If your main class is in com.app, everything is scanned.

Classpath Scanning
    
       Spring uses reflection + metadata reading (ASM library)
       It does NOT load every class, it reads annotations efficiently

Detecting Candidate Components
    
    Spring looks for:
        
        @Component (base)
        @Service
        @Repository
        @Controller
        @RestController
        @Configuration
    
    📌 Important:
    All of these are meta-annotated with @Component

Bean Definition Creation

    For each detected class:
    
    Spring creates a BeanDefinition
    Stores metadata (class name, scope, dependencies)

Bean Registration in IoC Container
    
       BeanDefinitions are registered in the ApplicationContext
       Later, actual objects (beans) are created

Dependency Injection Happens
    
       Spring resolves dependencies between scanned beans
       Uses:
       Constructor injection (preferred)
       Setter / Field injection

👉 Interview trap:
Wrong package → beans not detected.


Where ASM is Used
👉 Phase: Classpath Scanning (Early Startup)

Spring uses ASM (bytecode reading) to:
    
    Read annotations like:
    @Component
    @Service
    @Configuration
    Check class metadata:
    Class name
    Methods
    Annotations
    Do all this without loading the class into JVM
    ✅ Why ASM?
    Faster 🚀
    Memory efficient 💡
    Avoids unnecessary class loading

📌 Internal class used:
    
    MetadataReader
    ClassPathScanningCandidateComponentProvider

🔥 Example

    Spring scans .class file and checks:

❌ Without loading:
    
    @Service
    class UserService {}
    
    ✔ It just reads bytecode → detects @Service

⚙️ Where Reflection is Used
    
    👉 Phase: Bean Creation & Dependency Injection (Later Startup)
    
    Once Spring decides:
    👉 “Yes, this is a bean”
    
    Then it uses reflection to:
    
    Load the class
    Create object instance
    Call constructors
    Inject dependencies
    Invoke lifecycle methods
    🔥 Example
    @Service
    class UserService {
    public UserService(Repo repo) {}
    }

Reflection is used to:
    
    Find constructor
    Instantiate object
    Inject Repo


--------------------------------------------------------------------------------------------------------------------------


12. What is Bean Lifecycle?

    Instantiation
    Dependency injection
    Initialization
    Destruction

👉 Hooks:

@PostConstruct
@PreDestroy

Read BeanLifeCYle.md

-------------------------------------------------------------------------------------------------------------------
13. Singleton vs Prototype scope?
    Singleton → one instance (default)
    Prototype → new object every time

👉 Interview twist:
Spring manages lifecycle only for singleton.


Read BeanScope.md

----------------------------------------------------------------------------------------------------------------------------------

14. What is @Configuration?

            Marks class as bean configuration.

👉 Difference from @Component:
@Configuration supports proxying (CGLIB)
    
    @Bean
    public A a() {
    return new A(b());  // 👈 THIS is why b() is called
    }
    
    @Bean
    public B b() {
    return new B();
    }

Case 1: With @Configuration ✅

@Configuration

    class AppConfig { ... }
    
    Spring creates a CGLIB proxy
    When b() is called inside a():
    It does NOT call the method directly

It checks:

    “Do I already have bean B?”

If yes → returns same instance (singleton)

👉 So even though b() is called, it is intercepted
    
    
    Case 2: With @Component ❌
    @Component
    class AppConfig { ... }
    No proxy
    Direct Java call

So:
    
    Every time b() is called → new object created
    Breaks singleton behavior
---------------------------------------------------------------------------------------------------------

15. What is @Bean?
        
        Defines a bean manually.
        @Bean is an annotation used on a method inside a @Configuration class to tell Spring that the object returned by that method should be managed as a bean in the IoC container.

👉 Used when:
    
    third-party classes
    custom initialization
    
    Spring processes @Configuration class
    Finds methods annotated with @Bean
    Creates a BeanDefinition
    Calls the method using reflection
    Registers returned object as a bean

----------------------------------------------------------------------------------------------------------------------
16. What is Profiles?

        Profiles in Spring Boot allow you to define environment-specific configurations and load them selectively based on the active environment (like dev, test, prod).
Example:

    dev
    prod
    
    👉 Use:
    spring.profiles.active=dev

Example files:
    
    application-dev.properties
    application-prod.properties

Activate profile:

    spring.profiles.active=dev

👉 Only dev configuration gets loaded

    @Profile("dev")
    @Service
    class DevService {}


spring.profiles.active=dev,test

    Multiple profiles can be allowed

--------------------------------------------------------------------------------------------------------------------------------

17. application.properties vs application.yml?

Same purpose.

👉 YAML advantages:

hierarchical structure
cleaner for complex configs

1. application.properties
📌 Format

        Key-value pairs

📌 Characteristics
    
    Flat structure
    Simple and easy for small configs

2. application.yml
📌 Format

    YAML (hierarchical, indentation-based)
   📌 Characteristics
    
       Supports nested structure
       More readable for complex configs

------------------------------------------------------------------------------------------------------------------------------------------
18. What is Externalized Configuration?

Instead of hardcoding:

    String url = "jdbc:mysql://localhost:3306/test";

You write:

    spring.datasource.url=jdbc:mysql://localhost:3306/test

👉 Now it can be changed without touching code

Config outside code:

    properties file
    OS environment variables
    command line


1. 🌍 Environment Flexibility

       Dev / Test / Prod can have different configs
2. 🔄 No Rebuild Required

       Change config without recompiling
   
   3. 🔐 Security

          Sensitive data (passwords, API keys) kept outside code
   
  4. ⚙️ Better Maintainability

         Clean separation of logic vs configuration

⚙️ How Spring Boot Handles It Internally
    
    1. Environment Creation
       Spring creates a ConfigurableEnvironment
       2. PropertySources Loaded
          Multiple sources are added (priority order)
          3. Resolution
             When you use:
        
                 @Value
                 @ConfigurationProperties
    
    👉 Spring resolves values from these sources

⚖️ Property Source Priority (Very Important)
    
    Highest → Lowest:
    
    Command-line arguments
    JVM system properties
    OS environment variables
    application.properties / .yml
    Default values in code

👉 Higher priority overrides lower

-----------------------------------------------------------------------------------------------------------------------------------

19. What is @Value vs @ConfigurationProperties?

        @Value → single value
        @ConfigurationProperties → bulk binding

👉 Preferred:

    @ConfigurationProperties for scalability

1. @Value

   📌 What it does

        Injects individual property values from configuration.

📌 Example
    
    @Value("${app.name}")
    private String appName;

📌 Features
        
        Works with single values
        Supports SpEL (Spring Expression Language)

    @Value("#{2 * 3}")
    private int result;

🔹 2. @ConfigurationProperties
📌 What it does

    Binds group of related properties into a POJO.

📌 Example
    
    application.yml
    
    app:
    name: MyApp
    version: 1.0

Java class
    
    @ConfigurationProperties(prefix = "app")
    @Component
    public class AppConfig {
    private String name;
    private String version;
    }

-------------------------------------------------------------------------------------------------------------------------------------------------


20. What is Lazy Initialization?

        Beans created only when needed.
        
        👉 Default:
        
            Eager
        
        We can use @Lazy to cerate a bean only when needed
        
        1. 🚀 Improve Startup Time
           2. 🧠 Avoid Unnecessary Bean Creation
           3. 🔄 Break Circular Dependencies (Interview Favorite)
        
        Problem:
        
        Beans connecting to:
            
            Databases
            APIs
            Message brokers
        
        👉 These may be:
            
            Slow
            Not always needed
        
        ✔ Lazy avoids unnecessary connections at startup

-----------------------------------------------------------------------------------------------------------------------------------------------

21. _What is Circular Dependency?_

        A circular dependency occurs when two or more beans depend on each other directly or indirectly, forming a cycle.

A → B → A

Error : BeanCurrentlyInCreationException

    👉 Spring solves using proxies (not with constructor injection).

🔥 Constructor Injection → Fails
    
    Circular dependency is NOT allowed
    Because both objects must be fully created first

⚠️ Field / Setter Injection → Works (with limitation)
    
    @Service
    class A {
    @Autowired
    private B b;
    }
    
    @Service
    class B {
    @Autowired
    private A a;
    }

👉 Spring can:
    
    Create objects first
    Inject dependencies later

✔ So it may work

🚀 How Spring Solves It (Internally)

Spring uses 3-level caching:
    
    Singleton Objects (fully initialized)
    Early References (partially created beans)
    Object Factories

👉 Allows injecting a partially constructed bean

🔥 How to Fix Circular Dependency
    
    ✅ 1. Use @Lazy (Most Common)
    @Service
    class A {
    public A(@Lazy B b) {}
    }
    
    👉 Delays creation → breaks cycle
    
    ✅ 2. Switch to Setter Injection
    Allows delayed injection
    ✅ 3. Redesign (Best Solution)
    
    👉 Break the dependency chain
    
    Introduce third service
    Use events or interfaces


---------------------------------------------------------------------------------------------------------------------

22. What is Spring Boot DevTools?

        Spring Boot DevTools is a development-time tool that improves developer productivity by enabling automatic restart, live reload, and enhanced development features.
Without DevTools:
    
    Every code change → manual restart ❌
    Slower development

With DevTools:
    
    Code change → auto restart ✅
    Faster feedback loop 🚀

🔥 Key Features
🔹 1. Automatic Restart (Most Important)
    
    Monitors classpath for changes
    Restarts application automatically

    👉 Only changed classes are reloaded, not full JVM restart

🔹 2. Live Reload
    
    Automatically refreshes browser when UI changes

    👉 Works with browser extensions

🔹 3. Development-Friendly Defaults

Automatically disables:
    
    Template caching
    Static resource caching

    👉 So changes reflect immediately

🔹 4. Remote DevTools (Optional)

    Allows remote application updates (rarely used)

--------------------------------------------------------------------------------------------------------------------------------
23. What is CommandLineRunner?

Runs code after startup.

--------------------------------------------------------------------------------------------------------------------------------


24. What is ApplicationRunner?

Same as above but supports arguments.

--------------------------------------------------------------------------------------------------------------------------------

25. What is Logging in Spring Boot?

Default:

SLF4J + Logback

👉 Levels:

TRACE, DEBUG, INFO, WARN, ERROR

--------------------------------------------------------------------------------------------------------------------------------

26. What is @Slf4j?

From Lombok

👉 Automatically creates logger:

log.info("Hello");
--------------------------------------------------------------------------------------------------------------------------------

27. What is Actuator?

Production monitoring tool.

👉 Gives:

health
metrics
beans

--------------------------------------------------------------------------------------------------------------------------------


28. Why not expose all actuator endpoints?

Security risk:

internal data leak
system info exposure

--------------------------------------------------------------------------------------------------------------------------------

29. What is Dependency Management in Spring Boot?

Handled via:

parent POM
BOM (Bill of Materials)

    “Dependency Management in Spring Boot is the automatic handling of dependency versions using a BOM, ensuring compatibility and reducing the need to manually specify versions.”

🚀 How It Actually Works
🔹 1. Your Project Uses a POM

Every Maven project (including Spring Boot) has a pom.xml
👉 This is your main build file

🔹 2. Spring Boot Uses a BOM Inside It

Spring Boot provides a BOM:

👉 spring-boot-dependencies

This BOM:

    Manages dependency versions


Ensures compatibility
🔥 Two Ways Spring Boot Uses BOM

✅ Option 1: Using Parent POM (Most Common)
    
    <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>...</version>
    </parent>

👉 This parent POM:
    
    Automatically includes the BOM internally
    You don’t see it, but it’s there
    
    ✔ Easiest approach

✅ Option 2: Using BOM Directly (Without Parent)
    
    <dependencyManagement>
    <dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>...</version>
    <type>pom</type>
    <scope>import</scope>
    </dependency>
    </dependencies>
    </dependencyManagement>

👉 Used when:

You don’t want to use Spring Boot parent POM

--------------------------------------------------------------------------------------------------------------------------------

30. What is Fat JAR?
        
        A Fat JAR (also called an Uber JAR) is a single executable JAR file that contains:
        
        Your application code
        All required dependencies (libraries)
        Embedded server
        
        👉 Everything bundled into one file

With Fat JAR:

Just run:

    java -jar app.jar
    
    ✔ Simple deployment
    ✔ No external setup

🧠 How It’s Created

Using Maven:
    
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    
    👉 This plugin:
    
    Repackages normal JAR → Fat JAR

---------------------------------------------------------------------------------------------------------------------------------------



