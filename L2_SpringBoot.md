SPRINGBOOT: 

     ➡ Spring Boot is a framework built on top of the Spring Framework to build production-ready web and REST applications.
     ➡ It reduces boilerplate code by using auto-configuration and convention over configuration.

 ADVANTAGES : 

1️⃣ Auto-Configuration

    ➡ Spring Boot supports auto-configuration, so there is no need to write explicit configuration code
    (like configuring DispatcherServlet, ViewResolvers, DataSource, etc.)
    ➡ Based on the dependencies present in the classpath, Spring Boot automatically configures beans.

📌 Example:

    spring-boot-starter-web → auto configures:
        DispatcherServlet
        Tomcat
        Jackson
        Spring MVC setup

2️⃣ Dependency Management

    ➡ Spring Boot provides starter dependencies, which are a set of related dependencies grouped together.
    ➡ No need to:
    
        Add individual dependencies
        Manage dependency versions manually
    
    📌 Example:
    
        spring-boot-starter-web
        spring-boot-starter-data-jpa
        spring-boot-starter-security

    ➡ Versions are managed internally using Spring Boot dependency management (BOM).

3️⃣ Embedded Server

    ➡ Spring Boot provides embedded servers like:
    
        Tomcat (default)
        Jetty
        Undertow

    ➡ No need to externally deploy WAR files.

📌 Just run:

        public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        }


➡ In traditional Spring:

    Need to install Tomcat separately
    Deploy application manually

5️⃣ Production-Ready Features (Actuator) ⭐

    ➡ Built-in Spring Boot Actuator

Provides:

    Health checks (/actuator/health)
    Metrics
    Info
    Environment details

Very important for microservices & DevOps.



------------------------------------------------------------------------------------------------------------------------------------------


