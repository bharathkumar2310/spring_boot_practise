1. What is Spring Security?
        
        Spring Security is a framework used to secure Spring applications.
        It provides authentication, authorization, CSRF protection, session management, and password encryption. 
        It integrates with Spring Boot and helps secure web applications and REST APIs.

2. Authentication vs Authorization?

              Authentication	Authorization
              Who are you?	    What can you access?
        
        Example:
        
        login = authentication
        admin access = authorization

3. What is Authentication?

Verifying identity.

Example:
    
    username/password
    OTP
    JWT token
4. What is Authorization?

Checking permissions after authentication.

Example:

hasRole("ADMIN")

5. What is SecurityFilterChain?

🔥 VERY IMPORTANT

Spring Security works mainly using:

servlet filters

Requests pass through:

Filter Chain
6. What is DelegatingFilterProxy?

Bridge between:

servlet container
Spring-managed security filters
7. What is UsernamePasswordAuthenticationFilter?

Processes login requests.

Reads:

username
password
8. What is AuthenticationManager?

Responsible for authentication logic.

9. What is AuthenticationProvider?

Actual authentication implementation.

Example:

database validation
LDAP
JWT validation
10. What is UserDetailsService?

Loads user from DB.

loadUserByUsername()
11. What is UserDetails?

Represents authenticated user.

Contains:

username
password
authorities

    Learn all these from Spring_Security.md
-------------------------------------------------------------------------------------------------------------------

12. What is PasswordEncoder?

        A Password Encoder is used to securely store passwords by converting plain text passwords into hashed/encrypted form.
        In Spring Security, the PasswordEncoder interface provides methods to:
            
            encode() → converts raw password into hashed password
            matches() → compares raw password with stored hashed password
        
        This improves security because passwords are not stored as plain text in the database.

        Commonly Used Encoder
        
            BCryptPasswordEncoder

1. Encoding

Encoding means converting data from one format to another format using a reversible algorithm.

    Can be decoded back to original data
    Used for data transmission/storage
    Not meant for security

Example:

    Base64 encoding

Encoding is basically used for special characters

    Characters like:space  ?  &  =  /  %

already have special purposes in URLs.

Example:
    
    ?  -> starts query parameters
    &  -> separates parameters
    =  -> key/value separator
    
    So if user data contains these characters, they must be encoded to avoid confusion.

4. Character Encoding

        Used to represent characters in bytes.
        
        Examples:
        
            UTF-8
            ASCII

This allows systems to correctly understand text.


2. Hashing

        Hashing converts data into a fixed-size hash value using a one-way algorithm.
        
        Cannot easily get original data back
        Used for security
        Mainly used for passwords
        
        Example:
        
        BCrypt
        SHA-256

13. Why should passwords never be stored plain text?

If DB leaks:

    all accounts compromised
    Always hash passwords.

14. What is BCrypt?

        Popular password hashing algorithm.
        
        Features:
        
        salted hashing
        adaptive cost factor
15. What is SecurityContextHolder?

🔥 MOST ASKED

Stores authenticated user details.

Accessible anywhere:

SecurityContextHolder.getContext()
16. Where is authentication stored?

Inside:

SecurityContext

Usually thread-local storage.

17. What is JWT?

JWT = JSON Web Token

Used for:

stateless authentication
18. JWT Structure?

3 parts:

Header.Payload.Signature
19. What is inside JWT payload?

Claims like:

username
roles
expiration
20. Is JWT encrypted?

❌ No

JWT is usually:

encoded
signed

NOT encrypted by default.

21. Why JWT signature important?

Prevents token tampering.
If payload modified:

    signature validation fails
22. Session-based auth vs JWT?
    Session	JWT
    server stores session	client stores token
    stateful	stateless
    scalable difficulty	scalable
23. What is Stateless Authentication?

        Server stores NO client session.
        Each request contains auth token.

24. Why JWT useful for microservices?

        No shared session storage needed.
        Each service can validate token independently.

25. What is OncePerRequestFilter?

🔥 VERY IMPORTANT

Custom JWT filters usually extend:

OncePerRequestFilter

    Ensures filter runs once per request.

26. JWT Authentication Flow?
    
        User logs in
        Server validates credentials
        JWT generated
        Client stores token
        Client sends token in headers
        Server validates JWT
27. Where is JWT usually sent?

HTTP Header:

    Authorization: Bearer <token>
28. What is CSRF?

        Cross-Site Request Forgery.
        
        Attacker tricks authenticated user into sending requests.
        
        
        CSRF stands for Cross-Site Request Forgery.
        
        It is a security attack where a malicious website tricks an authenticated user into performing unwanted actions on another website.
        
        Example:
        
        User logs into banking website
        Session cookie is stored in browser
        User visits malicious website
        Malicious site sends request to bank using existing session cookie
        Bank thinks request came from valid user
        Example Scenario
        
        Suppose this endpoint transfers money:
        
        POST /transfer
        
        Attacker creates hidden form:
        
        <form action="https://bank.com/transfer" method="POST">
           <input type="hidden" name="amount" value="10000">
        </form>
        
        If the user is already logged into the bank, browser automatically sends session cookie with request.
        
        Server may process the transfer thinking it is a genuine request.

29. Why often disable CSRF in JWT apps?

Because JWT APIs are:

stateless
not session-cookie based

Common:

csrf.disable()



30. What is difference between permitAll() and ignoring()?

    permitAll	                             ignoring
    passes through security filters	     bypasses filters completely

| Type               | Purpose                         | Uses Key? | Reversible? | Common Usage                       | Examples                                 |
| ------------------ | ------------------------------- | --------- | ----------- | ---------------------------------- | ---------------------------------------- |
| Hashing Algorithms | Generate fixed hash value       | No        | No          | Password storage, integrity checks | BCrypt, SHA-256, SHA-512, PBKDF2, SCrypt |
| Signing Algorithms | Verify authenticity & integrity | Yes       | No          | JWT signing, digital signatures    | RS256, HS256, ES256, RSA, HMAC           |
