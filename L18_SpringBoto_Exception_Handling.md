Exception Handling:


![img.png](Images/EH1.png)
 
![img_1.png](Images/EH2.png)


1. Say we are not using @ExceptionHandler/@Controller Advice we just catch the exception then 
   the exception will not go to any of the resolvers , it works in the same as it works in jav

![img_4.png](Images/EH5.png)

2. we are not using @ExceptionHandler/@Controller Advice we just throw the exception without catching
   the thrown ExceptionObject goes to dispatcher servlet
   there dispatcher servlet calls handler exception resolver composite
   which will invoke each resolvers to resolve it
   First it goes into exceptionhanlerexception resolver ,since it does not have @ExceptionHandler/ @ControllerAdvice
  it cannot resolve, next it goes to response status exception handler which also cannot resolve then it goes to default 
  error exception handler which also cannot resolve then it goes to defulat error attributes
  it will give default error msg and status(500 internal server error) which will be set by response handler in HttpServlet Response


![img_2.png](Images/EH3.png)
![img_3.png](Images/EH4.png)


![img_5.png](Images/EH6.png)

![img_6.png](Images/EH7.png)

3. ExceptionHandler/ControllerAdvice

ExceptionHandler is for handling exception in that particular controller
Controller Advice is used for global exception handling


Application starts
↓
Beans created
↓
BeanPostProcessors finish
↓
ExceptionHandlerExceptionResolver.afterPropertiesSet()
↓
Scans controllers & advice
↓
Caches exception handler mappings
↓
App ready


SO when it goes to exception handler exception resolver it checks for the @exception/@controllerAdvice mapping if present it calls that excption method
and handles it there


Controller throws
↓
DispatcherServlet catches
↓
HandlerExceptionResolverComposite starts
↓
ExceptionHandlerExceptionResolver.resolveException()


exception Handler direct returns to dispatcher servlet if response of handler is a string
else return responsEntity where status and body is set


![img_7.png](Images/EH8.png)

![img_8.png](Images/EH9.png)


![img_9.png](Images/EH10.png)

![img_10.png](Images/EH11.png)


![img_11.png](Images/EH12.png)


![img_12.png](Images/EH13.png)

![img_13.png](Images/EH14.png)



2. @Response Status

Here this wont be handled by exception handler exception resolver but it will be handled by response status exception handler

![img.png](img.png)

![img_1.png](img_1.png)


![img_2.png](img_2.png)


![img_3.png](img_3.png)

![img_4.png](img_4.png)


3. Default Handler Exception Resolver

![img_5.png](img_5.png)

The DefaultHandlerExceptionResolver is a built-in Spring MVC component that implements the HandlerExceptionResolver interface. Its main job is to handle standard Spring MVC exceptions that occur during request processing, like:

        HttpRequestMethodNotSupportedException
        HttpMediaTypeNotSupportedException
        MissingServletRequestParameterException
        ServletRequestBindingException
        NoHandlerFoundException
        ConversionNotSupportedException
        TypeMismatchException
        HttpMessageNotReadableException / HttpMessageNotWritableException
