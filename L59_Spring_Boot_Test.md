UNIT TEST :
    
    A unit test is a type of software test that checks whether a small, individual part of a program (called a unit) works correctly.
    A “unit” is usually:
    
        a function
        a method
        a class
        or a small module

1. Finds bugs early

        If a method has a problem, a unit test detects it immediately.
        
        Without tests:
        
            bug may reach production
            debugging becomes harder later

2. Makes refactoring safe

       Developers frequently improve code structure.
       Without tests:fear of breaking existing behavior
    
       With tests:
    
           you can modify code confidently
           tests confirm behavior still works
    
       This is one of the biggest real-world benefits.

3. Improves code quality

       When writing testable code, developers naturally create:
    
       smaller methods
       cleaner design
       better separation of concerns
       lower coupling
    
       So testing indirectly improves architecture.

4. Saves debugging time

       Manual testing repeatedly is slow.
       Instead of:
        
           running the whole application
           clicking through UI
           testing manually every time
    
       You run tests in seconds.

----------------------------------------------------------------------------------------------------------------------------------


What is JUnit 5?

      JUnit 5 is the modern testing framework used in Java for writing and running tests.
      
      It is mainly used for:
         
         unit testing
         integration testing
         test automation


What JUnit 5 provides

1. Test execution engine

         JUnit can:
         
            discover tests
            run tests
            generate reports

2. Assertions

         Used to verify expected behavior.
         Example:
   
            assertEquals(10, calculator.add(5,5));

3. Test lifecycle management

      Run code:
   
         before each test
         after each test
         before all tests
         after all tests
   
      Example:
   
         create DB connection before tests
         clean resources after tests

4. Better annotations

         JUnit 5 provides annotations like:
   
| Annotation    | Purpose                    |
| ------------- | -------------------------- |
| `@Test`       | Marks a test method        |
| `@BeforeEach` | Runs before every test     |
| `@AfterEach`  | Runs after every test      |
| `@BeforeAll`  | Runs once before all tests |
| `@AfterAll`   | Runs once after all tests  |


5. Parameterized testing

         Run same test with multiple inputs.
         
         Example:
         
            test login with many usernames
            test calculator with many numbers

6. Integration with tools

      JUnit works with:
   
            Maven
            Gradle
            IntelliJ
            Eclipse
            Jenkins
            GitHub Actions
            Spring Boot

JUnit 5 architecture has 3 parts:

| Part     | Purpose                      |
| -------- | ---------------------------- |
| Platform | Runs test engines            |
| Jupiter  | Main JUnit 5 testing API     |
| Vintage  | Supports old JUnit 3/4 tests |


1. Jupiter

Jupiter is the part you mostly use daily.

It provides:

      @Test
      assertions
      lifecycle annotations
      parameterized tests
      extensions

So Jupiter = API for writing modern tests.

2. Platform

         Platform is the foundation that runs tests.
         
         It:
         
         discovers tests
         launches tests
         executes test engines
         integrates with IDEs and build tools


Platform does:

      scan classpath
      find test engines
      detect Jupiter tests
      run them
      show report


What is a Test Engine?

      A test engine is something that knows:
      
      “How to run a particular kind of test.”
      
      Examples:
      
      Engine	           Runs
      Jupiter Engine	JUnit 5 tests
      Vintage Engine	JUnit 4 tests
      
      Platform manages these engines.


3. Vintage

         Vintage exists for backward compatibility.
         
         It allows old JUnit tests to still run inside JUnit 5 projects.

---------------------------------------------------------------------------------------------------------------------------------

1. @TEST

         @Test is an annotation in JUnit 5 used to mark a method as a test method.
         It tells JUnit:
            “Run this method as a test.”

2. @BeforeEach

          Runs before every test method.

         @BeforeEach
         void setup() {
         System.out.println("Before test");
         }

         Used for:
         
            object creation
            initializing test data
            setup logic

3. @AfterEach

       Used for:
      
         closing resources
         cleanup
         resetting state

4. @BeforeAll

       Runs once before all tests.
       Used for expensive setup:
   
         DB connection
         server startup
         shared objects

5. @AfterAll

          Runs once after all tests.
       Used for:
   
       releasing shared resources
       stopping servers

   6. @DisplayName

            Gives readable test names.
   
            @DisplayName("Addition should return correct result")
            @Test
            void testAdd() {
   
            }

7. @Disabled

       Skips a test.
       @Disabled
       @Test
       void testFeature() {

       }

8. @ParameterizedTest

            Runs same test with multiple inputs.
      
            @ParameterizedTest
            @ValueSource(ints = {1, 2, 3})
            void testNumbers(int number) {
            assertTrue(number > 0);
            }

         Runs test for:
      
         1
         2
         3

Very important annotation in real projects.

12. @ValueSource

         Provides input values to parameterized tests.
         
         @ValueSource(strings = {"java", "spring"})


---------------------------------------------------------------------------------------------------------------------------------


What are Assertions in JUnit 5?

      Assertions are methods used to verify whether:
      actual result == expected result
      
      They decide whether a test:
      
            PASSES ✅
            FAILS ❌
      
      Without assertions:
            
            JUnit only runs code
            it cannot verify correctness


1. assertEquals()

         Checks equality.
         
         @Test
         void testAdd() {
         assertEquals(5, 2 + 3);
         }

Meaning:

      expected = 5
      actual   = 2 + 3

      If equal → PASS

2. assertNotEquals()

         Checks values are different. 
         assertNotEquals(10, 5 + 2);

3. assertTrue()

       Checks condition is true.
       assertTrue(5 > 2);

Real-world:

      assertTrue(user.isActive());

4. assertFalse()

         Checks condition is false.
         assertFalse(5 < 2);

5. assertNull()

         Checks object is null.
         String name = null;
         assertNull(name);

6. assertNotNull()

         Checks object is NOT null.
         User user = new User();
         assertNotNull(user);

7. assertThrows()

       Checks exception is thrown.

VERY IMPORTANT.

      @Test
      void shouldThrowException() {
      
          assertThrows(
              ArithmeticException.class,
              () -> {
                  int x = 10 / 0;
              }
          );
      }

      If exception occurs → PASS
      If no exception → FAIL


--------------------------------------------------------------------------------------------------------------------------------


What is Mockito?

      Mockito is a Java mocking framework used in testing to create fake objects (called mocks).
      These fake objects simulate real dependencies during unit testing.

Why Mockito is needed

      In real applications, classes depend on other classes.

      Service
      ↓
      Repository
      ↓
      Database

During unit testing:

      we only want to test the Service
      NOT database
      NOT external APIs
      NOT other systems
      
      Mockito helps isolate the class being tested.

Solution using Mockito

Mockito creates fake repository.

      @Mock
      UserRepository repo;
      
      Now:
      
      no database needed
      faster tests
      isolated testing




1. @Mock ⭐ MOST IMPORTANT

         Creates a fake/mock object.
         
         Example
         @Mock
         UserRepository repo;

         Mockito creates:
         
            fake UserRepository
            no real DB calls
            no real logic execution

Why use @Mock?

      To isolate the class being tested.
      Instead of:Service → Real DB
      you get:Service → Fake Repository
Example
      
      when(repo.findUser())
      .thenReturn(new User("Bharath"));

Now repository returns fake data.



2. @InjectMocks ⭐ VERY IMPORTANT

         Creates actual object and injects mocks into it.

Example
      
      @InjectMocks
      UserService service;

Suppose:

      class UserService {
      
          private UserRepository repo;
      
          UserService(UserRepository repo) {
              this.repo = repo;
          }
      }

      Mockito automatically injects mocked repo.
      Without @InjectMocks
      
      You must manually do:
      
         service = new UserService(repo);
      
      Mockito automates this.

Combined Example
      
      @Mock
      UserRepository repo;
      
      @InjectMocks
      UserService service;

Meaning:

      Create fake repo
      Inject into real service



3. @ExtendWith(MockitoExtension.class) ⭐ VERY IMPORTANT

         Enables Mockito in JUnit 5.

Example
      
      @ExtendWith(MockitoExtension.class)
      class UserServiceTest {
      
      }

Without this:

      Mockito annotations won't initialize automatically.
Why needed?

JUnit 5 needs extension mechanism to:

      initialize mocks
      process annotations
      inject dependencies

Full Example
      
      import org.junit.jupiter.api.Test;
      import org.junit.jupiter.api.extension.ExtendWith;
      
      import org.mockito.InjectMocks;
      import org.mockito.Mock;
      import org.mockito.junit.jupiter.MockitoExtension;
      
      import static org.mockito.Mockito.*;
      import static org.junit.jupiter.api.Assertions.*;
      
      @ExtendWith(MockitoExtension.class)
      class UserServiceTest {
      
          @Mock
          UserRepository repo;
      
          @InjectMocks
          UserService service;
      
          @Test
          void testUser() {
      
              when(repo.findUser())
                  .thenReturn(new User("Bharath"));
      
              User user = service.getUser();
      
              assertEquals("Bharath", user.getName());
          }
      }

| Annotation                            | Purpose                        |
| ------------------------------------- | ------------------------------ |
| `@Mock`                               | Creates fake object            |
| `@InjectMocks`                        | Injects mocks into real object |
| `@Spy`                                | Partial mock                   |
| `@Captor`                             | Captures method arguments      |
| `@ExtendWith(MockitoExtension.class)` | Enables Mockito in JUnit 5     |


----------------------------------------------------------------------------------------------------------------------------------


Mockito mocks objects by creating a dynamic fake implementation of a class or interface at runtime.

It uses:
      
      reflection
      bytecode generation
      proxy/subclass creation

internally.

You write:
      
      @Mock
      UserRepository repo;

But internally Mockito creates something like:

      class FakeUserRepository extends UserRepository {
      
          @Override
          User findUser() {
              return fakeValue;
          }
      }

This fake object replaces the real object during testing.

Real Understanding

Suppose you have:

      interface UserRepository {
      User findUser();
      }

Normally:

      repo.findUser();
      calLs real implementation.

Mockito creates fake implementation

Internally something conceptually like:

      class MockitoGeneratedClass
      implements UserRepository {
      
          @Override
          User findUser() {
              return mockedResult;
          }
      }

So no database code exists there.

Then how does when(...).thenReturn(...) work?

You define behavior:

      when(repo.findUser())
      .thenReturn(new User("Bharath"));

Mockito internally stores:
         
         Method: findUser()
         Return: User("Bharath")

Later when:

      repo.findUser()

    is called,

Mockito intercepts it and returns stored value.


--------------------------------------------------------------------------------------------------------------------------------


doNothing() in Mockito

      doNothing() is used to tell Mockito:
      “When this void method is called, do nothing.”
      It is mainly used for methods that return void.

Why needed?

For normal methods:

      when(repo.findUser()).thenReturn(user);
      works because method returns value.

But for void methods:

      emailService.sendEmail();
      there is no return value.

So Mockito provides:

      doNothing()
      doThrow()
      doAnswer()

style syntax.

Example

      class EmailService {
      
          void sendEmail(String msg) {
              // sends real email
          }
      }

Mocking it
      
      doNothing()
      .when(emailService)
      .sendEmail("Hello");

Meaning:
      
      If sendEmail("Hello") is called,
      don't execute anything.
But important point

      For pure mocks (@Mock):
      void methods already do nothing by default.
      
      So this:
      
      @Mock
      EmailService emailService;
      
      already behaves like:
      
      do nothing
      Then why use doNothing()?
      
      Mostly used:
      
      for readability
      with spies
      explicit behavior definition



verify() in Mockito ⭐ VERY IMPORTANT

      verify() checks whether a method was called.

      Example
      verify(repo).findUser();

Meaning:

      Ensure findUser() was called.
Why useful?

Sometimes testing is not just about return values.

You also want to verify:

      interactions happened
      methods were invoked
      correct dependencies were used

Real Example
Service
      
      class UserService {
      
          private UserRepository repo;
      
          UserService(UserRepository repo) {
              this.repo = repo;
          }
      
          User getUser() {
              return repo.findUser();
          }
      }
Test
      
      @Test
      void testGetUser() {
      
          service.getUser();
      
          verify(repo).findUser();
      }

Mockito checks:

      was findUser() called?
      
      If yes:PASS
      Else:FAIL

Verify specific count
      
      Called once
      verify(repo, times(1)).findUser();
      Called twice
      verify(repo, times(2)).findUser();
      Never called
      verify(repo, never()).findUser();

Verify with arguments
      
      verify(emailService)
      .sendEmail("Hello");

Checks exact argument passed.

Verify using matchers
      
      verify(emailService)
      .sendEmail(anyString());
Full Example
      
      @ExtendWith(MockitoExtension.class)
      class UserServiceTest {
      
          @Mock
          UserRepository repo;
      
          @InjectMocks
          UserService service;
      
          @Test
          void testGetUser() {
      
              User user = new User("Bharath");
      
              when(repo.findUser())
                  .thenReturn(user);
      
              User result = service.getUser();
      
              assertEquals("Bharath", result.getName());
      
              verify(repo).findUser();
          }
      }


-----------------------------------------------------------------------------------------------------------------------------------


AAA Pattern (Good Testing Practice)

Very important concept.

      Structure tests like:
      
      Arrange
      Act
      Assert
      
      Example:
      
      // Arrange
      when(repo.findUser()).thenReturn(user);
      
      // Act
      User result = service.getUser();
      
      // Assert
      assertEquals("Bharath", result.getName());


----------------------------------------------------------------------------------------------------------------------------------


What is @Spy in Mockito?

      @Spy creates a partial mock.

Meaning:

      real object is created
      real methods execute by default
      but specific methods can be mocked/stubbed   

Difference Between @Mock and @Spy

| Feature              | `@Mock`      | `@Spy`          |
| -------------------- | ------------ | --------------- |
| Real object created? | No           | Yes             |
| Real methods run?    | No           | Yes             |
| Default behavior     | Fake         | Real            |
| Usage                | Full mocking | Partial mocking |


Example of @Mock
      
      @Mock
      List<String> list;
      
      This is completely fake.
      
      list.size();
      
      returns default value:
      
      0
      
      No real ArrayList logic runs.

Example of @Spy
      
      @Spy
      ArrayList<String> list;
      
      Now real object exists.
      
      Real methods execute
      list.add("Java");
      
      System.out.println(list.size());
      
      Output:
      
      1

Because actual ArrayList methods executed.

Then why use Spy?

      Sometimes you want:
      
      mostly real behavior
      but override only few methods
      
      That is where Spy helps.


---------------------------------------------------------------------------------------------------------------------------------


1. when().thenReturn() ⭐ Most Common

Syntax:

      when(mock.method())
      .thenReturn(value);
Example
      
      when(repo.findUser())
      .thenReturn(new User("Bharath"));

Meaning:

      If findUser() is called,
      return fake user.
      Works well for normal mocks
      
      @Mock
      UserRepository repo;
      
      because real method does NOT execute.
      
      Internal Behavior
      
      Mockito actually calls:
      
      repo.findUser()
      
      during stubbing process.
      
      But since it is a mock:
      
      no real logic exists
      safe to call

2. doReturn().when() ⭐ Important for Spy

Syntax:
      
      doReturn(value)
      .when(mock)
      .method();

Example

      doReturn(new User("Bharath"))
      .when(repo)
      .findUser();

      Same result:fake value returned

BUT internal behavior differs.

Key Difference

| Feature                            | `when()`        | `doReturn()`   |
| ---------------------------------- | --------------- | -------------- |
| Calls real method during stubbing? | Yes             | No             |
| Safe for Spy?                      | Sometimes risky | Safe           |
| Most commonly used?                | Yes             | Mainly for Spy |


Why is this important?
      
      Because with @Spy,
      real methods execute.

Spy Example
      
      @Spy
      ArrayList<String> list;

Problem with when()
when(list.get(0))
.thenReturn("Java");

During stubbing:

      list.get(0)
      actually executes real method.

But list is empty.

      So:IndexOutOfBoundsException

occurs.
      
      Safe version
      doReturn("Java")
      .when(list)
      .get(0);

Now real method is NOT called during stubbing.
Mockito directly stores fake behavior.


--------------------------------------------------------------------------------------------------------------------------



1. Mockito.any()

Used when:“I don’t care what exact object/value is passed.”

Problem Without any()

Suppose:
      
      Mockito.when(userRepository.save(existingUser))
      .thenReturn(existingUser);
      
      Mockito expects:
      
      exact same object
      
      If another User object comes:
      ❌ stub won’t match
      
      Solution → any()

      Mockito.when(userRepository.save(Mockito.any(User.class)))
      .thenReturn(existingUser);
      
      Meaning:
      
      “If save() gets ANY User object, return existingUser.”
      
      Visual Understanding
      
      Without any():
      
      save(sameObjectOnly)
      
      With any():
      
      save(any User object)

Common Examples
      
      Any User
      Mockito.any(User.class)
      Any String
      Mockito.anyString()
      Any Long
      Mockito.anyLong()
      Any Integer
      Mockito.anyInt()
      Real Example
      Mockito.when(userRepository.findById(Mockito.anyLong()))
      .thenReturn(Optional.of(user));

Now:
      
      findById(1L)
      findById(100L)
      findById(999L)

all return same mocked value.

2. eq()

         Used when:
         
         “Some arguments should be exact,
         others can be anything.”
         
         Why eq() Exists
         
         Mockito rule:
         If one matcher used (any()),
         ALL arguments should use matchers.
         
         Example
         
         Suppose method:
         
         updateUser(Long id, String name)
         WRONG
         Mockito.when(service.updateUser(1L, Mockito.anyString()))
         
         Mockito error occurs.
         
         Because:
         
         one raw value
         one matcher
         
         mixed together.
         
         CORRECT → use eq()
         Mockito.when(
         service.updateUser(
         Mockito.eq(1L),
         Mockito.anyString()
         )
         )
         Meaning of eq()
         eq(1L)
         
         means:
         
         “Argument must equal 1L.”
         
         Real Example
         Mockito.when(
         userRepository.findByNameAndStatus(
         Mockito.eq("Bharath"),
         Mockito.anyString()
         )
         )
         .thenReturn(user);
         
         Meaning:
         
         first arg must be "Bharath"
         second can be anything

 3. @CsvSource
      
         This is from JUnit 5 parameterized testing.
      
         Used when:
      
         “Pass multiple parameters into test.”
      
         Example
         @ParameterizedTest
         @CsvSource({
         "1, Bharath",
         "2, Kumar",
         "3, Rahul"
         })
         void test(Long id, String name) {
      
             System.out.println(id);
             System.out.println(name);
         }
      
         JUnit runs test 3 times.