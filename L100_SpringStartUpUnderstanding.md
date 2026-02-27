INTERNAL FLOW OF SPRING STARTUP:


1. All code will be in byte code when we run spring boot application
2. When we give SpringApplication.run(Application.class, args) ---> Application class will be loaded into memory
3. Spring creates applicationcontext based on none/servlet/webflux
4. Now since Application class is loaded Spring will have the class object of Application class it uses reflection to read the meta data annotations
5. It collects all the meta data like @SpringBootApp --> @Config + @ComponentScan + @AutoConfig(which package to scan, which classes to autoconfig etc);
4. Sprign calls applicationcontext.refresh()
 
               a. Performs component scanning (ASM)
               b. Processes auto-configuration
               c. Creates BeanDefinitions 
               d. Initializes BeanPostProcessors
               e. Instantiates singleton beans via reflection
               f. Applies proxies and lifecycle callbacks

(d). Registers BeanPostProcessors 

    Spring finds all BeanDefinitions that implement BeanPostProcessor (from user code or auto-configuration)
    It instantiates these BPPs immediately via reflection
    Registers them in beanFactory.beanPostProcessorList
    ✅ This happens before any other singleton beans are created

(a) Performs component scanning (ASM) 

     Based on the package info(which package it needs to scan) it  Reads .class files using ASM (MetadataReader)
     It does not use reflection here and classes are not loaded it just reads byte code and get their meta data
     and stores it in bean defintiion


        ```java
        GenericBeanDefinition bd = new GenericBeanDefinition();
        bd.setBeanClassName(metadataReader.getClassMetadata().getClassName());
        bd.setScope("singleton"); // default
        bd.setLazyInit(false);
        bd.setAutowireMode(AUTOWIRE_BY_TYPE);
        
        
        ```

b. Process AutoConfig: 

        After user components are scanned and their BeanDefinitions stored, Spring triggers auto-configuration
        Auto-config classes are also read via ASM, and their BeanDefinitions are registered before any bean is created

5. After all bean definitions are created now only the bean instansiation happens (cl ass loading also) 

6. after bean instanziated it goes throw each and every postprocessorbean

```java
public Object postProcessBeforeInitialization(Object bean, String beanName) {
System.out.println("Before Init: " + beanName);
return bean;
}
```

7. For autowiring and post construct also we have a specific bean processor which does that using reflection


        AutowiredAnnotationBeanPostProcessor → injects dependencies via reflection
        CommonAnnotationBeanPostProcessor → handles @PostConstruct



------------------------------------------------------------------------------------------------------------------------------------------




A BeanPostProcessor (BPP) is:

      A Spring extension point that allows you to modify a bean before and after initialization.
      It acts like a hook into the bean lifecycle.

DEFINITION:
```java

      public interface BeanPostProcessor {
      
          Object postProcessBeforeInitialization(Object bean, String beanName);
      
          Object postProcessAfterInitialization(Object bean, String beanName);
      }

```
EXACT FLOW :
1. Instantiate bean
2. Populate properties (DI)
3. postProcessBeforeInitialization()
4. @PostConstruct
5. postProcessAfterInitialization()
6. Bean ready


We can also manually create Bean Post Processor

```java
      @Component
      public class MyCustomBeanPostProcessor implements BeanPostProcessor {
      
          @Override
          public Object postProcessBeforeInitialization(Object bean, String beanName) {
              System.out.println("Before Init: " + beanName);
              return bean;
          }
      
          @Override
          public Object postProcessAfterInitialization(Object bean, String beanName) {
              System.out.println("After Init: " + beanName);
              return bean;
          }
      }

```
How it works :

Spring creates these beanPostProcessors using autoconfiguration based on our dependency and store it in

      List<BeanPostProcessor> beanPostProcessors

After that component scanning takes place and each bean is created

After bean creation each bean hits postProcessBeforeInitialization of each bean processor
A special BeanPostProcessor called CommonAnnotationBeanPostProcessor invokes the @PostConstruct method.
After postconstruct each bean hits postProcessAfterInitialization of each bean processor



🧠 Important: There Are 3 Different “Post Processors”

| Type                                | Works On        | Runs When             |
| ----------------------------------- | --------------- | --------------------- |
| BeanDefinitionRegistryPostProcessor | BeanDefinitions | Before beans exist    |
| BeanFactoryPostProcessor            | BeanDefinitions | Before beans exist    |
| BeanPostProcessor                   | Bean instances  | After bean is created |


🔥 First: Who Registers ConfigurationClassPostProcessor?

This one is special.

It is NOT discovered by component scanning.

It is manually registered by Spring when the ApplicationContext is created.

Inside:

AnnotationConfigApplicationContext

Spring automatically registers:

      ConfigurationClassPostProcessor
      AutowiredAnnotationBeanPostProcessor
      CommonAnnotationBeanPostProcessor
...

These are infrastructure beans.

So:

      👉 They are not found by bytecode scanning
      👉 They are programmatically registered


