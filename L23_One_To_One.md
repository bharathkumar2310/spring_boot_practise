OneToOne Unidirectional : 


![img.png](Images/OneToOne1.png)

![img_1.png](Images/OneToOne2.png)

![img_2.png](Images/OneToOne3.png)



```java

@Entity
public class UserClass {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;

    private String userName;

    @OneToOne
    @JoinColumn(name = "add_foreign_key" , referencedColumnName = "addId")
    private UserAddress userAddress;
}
```


```java
package com.example.jpa.oneTOne;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class UserAddress {

    @Id
    private Long addId;

    private String address;
}

```


Foreign Key ----> owning side ----> Here User class has the foreign key (add_foreign_Key) so it is the owning side
mappedBy ------> nonOwning side ----> for unidirectional we dont have mapped by


Since this is unidirectional It is **not required to add joinColumn** since Spring knows where to add the joinColumn which is on the One To One Side
And for unidirectinal there is no mappedBy



![img_3.png](Images/OneToOne4.png)

![img.png](Images/OneToOne5.png)



CASCADE TYPE :


Without cascade :

SCENARIO 1 : Without cascade u reference the child obj in parent obj and try to save parent obj without saving child obj
---> Will get trnsient error

```java

@Entity
public class UserClass {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;

    private String userName;

    @OneToOne
    @JoinColumn(name = "add_foreign_key" , referencedColumnName = "addId")
    private UserAddress userAddress;
}
```
```java

@PostMapping("/saveUser")
    public UserClass saveUser(@RequestBody  UserClass userClass) {
        return userRepo.save(userClass);
    }
```

say the payload is this

    {
    "userName" : "Loosu",
        "userAddress" : {
            "address" : "Loosu Theru"
        }
    }

Since we didnot give cascade it wil throw Persistent instance of 'com.example.jpa.oneTOne.UserClass' references an unsaved transient instance of 'com.example.jpa.oneTOne.UserAddress' (persist the transient instance before flushing)

because for saving User u need the foreign key value also since cascade is not given UserAddress is not saved
and whie inserting user first it needs to insert User but since cascde is not given it wont

🔥 What actually happens internally

You send:

        {
        "userName": "Loosu",
        "userAddress": {
        "address": "Loosu Theru"
        }
        }
Step-by-step:

    Spring converts JSON → objects
    UserClass user = new UserClass();
    user.setUserAddress(new UserAddress()); // transient object

You call:

        userRepo.save(user);
Hibernate tries to persist UserClass
It sees:@OneToOne

    private UserAddress userAddress;

👉 But:

userAddress is TRANSIENT (not saved, no ID)
No cascade = PERSIST
❌ Exact problem

Hibernate refuses to proceed because:

“You are referencing an object that is not yet saved in DB”

So it throws:

TransientPropertyValueException

SCENARIO 2 : Without cascade u reference the child obj in parent obj and try to save parent obj after saving child obj
---> Works fine


But if u save the userAddress fisrt and save user then it will work fine

```java

@PostMapping("/saveUser")
    public UserClass saveUser(@RequestBody  UserClass userClass) {
        UserAddress userAddress = userClass.getUserAddress();
        userAddressRepo.save(userAddress);
        return userRepo.save(userClass);
    }
```


SCENARIO 3 : Without cascade and without  reference of the child obj in parent obj and try to save parent obj
---> Parent Obj gets saved with foreign key as null

```java
@PostMapping("/saveUser1")
    public UserClass saveUser1(@RequestBody UserDto userDto) {
        UserClass userClass = new UserClass();
        userClass.setUserName(userDto.getUserName());

        return userRepo.save(userClass);
    }
```

With Cascade : 

@Entity
public class UserClass {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;

    private String userName;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "add_foreign_key" , referencedColumnName = "addId")
    private UserAddress userAddress;
}


---> saving userClass is enough it wil automatically save useraddress first and then save user
But say if it is sequence Spring will get the id from db before inserting and set So the insertion order doesnot matter if it is sequence

---> Say if we dont reference child on parent object and save parent obj , then it will not try to save child , it will only save parent with foreign key as null

---> If we save useraddress first and save user with cascade 
on 1st trans it saves useraddress
on second trans it saves address it does not save useraddress again since useraddreess wi be in detached satte

| Scenario             | IDENTITY     | SEQUENCE                 |
| -------------------- | ------------ | ------------------------ |
| Child not saved      | ❌ Error      | ❌ Error                  |
| Child saved first    | ✅ Works      | ✅ Works                  |
| No child reference   | ✅ FK null    | ✅ FK null                |
| Cascade persist      | ✅ Works      | ✅ Works (more efficient) |
| Manual + cascade     | ✅ Works      | ✅ Works                  |
| ID generation timing | After insert | Before insert            |


![img.png](Images/OneToOne6.png)

![img_1.png](Images/OneToOne7.png)

![img_2.png](Images/OneToOne8.png)

![img_3.png](Images/OneToOne9.png)

![img_4.png](Images/OneToOne10.png)

![img_5.png](Images/OneToOne11.png)

![img_6.png](Images/OneToOne12.png)

![img_7.png](Images/OneToOne13.png)

![img_8.png](Images/OneToOne14.png)

![img_9.png](Images/OneToOne15.png)

![img_10.png](Images/OneToOne16.png)

![img_11.png](Images/OneToOne17.png)

![img_12.png](Images/OneToOne18.png)

![img_13.png](Images/OneToOne19.png)

![img_14.png](Images/OneToOne21.png)

![img_15.png](Images/OneToOne22.png)

![img_16.png](Images/OneToOne23.png)

![img_17.png](Images/OneToOne24.png)

![img_18.png](Images/OneToOne25.png)



FETCH TYPE :

@oneToOne is eager by default
If u fetch user it will also fetch address 

For Lazy :


🔥 SCENARIO 1

    @GetMapping("/getById")
    public UserClass getUser(@RequestParam Long id) {
        return userRepo.findById(id).orElse(null);
    }
🧠 What happens

    findById() runs → session opens & closes
    UserClass returned with:
    userAddress = proxy (NOT loaded)
    Serialization starts (Jackson)

👉 It calls:

    getUserAddress().getAddress()
    Hibernate tries to fetch

❌ But:

    Session CLOSED
💥 Result

    LazyInitializationException


🔥 SCENARIO 2

    @GetMapping("/getById")
    @Transactional
    public UserClass getUser(@RequestParam Long id) {
        return userRepo.findById(id).orElse(null);
    }
🧠 What happens

    Transaction starts
    findById() → user loaded (address still LAZY proxy)
    Method returns

👉 IMPORTANT:

    Transaction ends BEFORE serialization
    Session CLOSED ❌
💥 Result

    Same as Scenario 1:
    
    LazyInitializationException
⚠️ Key learning

@Transactional alone is NOT enough

🔥 SCENARIO 3

        @GetMapping("/getById")
        public UserClass getUser(@RequestParam Long id) {
            UserClass uClass = userRepo.findById(id).orElse(null);
            uClass.getUserAddress().getAddress(); // 🔥
            return uClass;
        }
🧠 What happens

    findById() → session opens & closes
    Immediately after:
    uClass.getUserAddress().getAddress();

👉 Hibernate tries to fetch LAZY data

❌ But:

    Session already CLOSED
💥 Result

    LazyInitializationException


    🔥 Short answer why error?
    
    Because Hibernate needs an active Session (and DB connection) to execute SQL, and that Session is closed when the transaction ends.

🔥 SCENARIO 4

    @GetMapping("/getById")
    @Transactional
    public UserClass getUser(@RequestParam Long id) {
        UserClass uClass = userRepo.findById(id).orElse(null);
        uClass.getUserAddress().getAddress(); // 🔥
        return uClass;
    }
🧠 What happens

    Transaction starts → session OPEN
    findById() → user loaded
    You access:
    getUserAddress().getAddress();

👉 Hibernate:

    Executes SQL
    Loads UserAddress
    Method returns
    Transaction ends
    Serialization happens

👉 Now:

userAddress already LOADED ✅
✅ Result

    ✔️ Works perfectly
    ✔️ No exception



        
        Here all Scenarios might even work 
        
        👉 Because of Open Session In View (OSIV)
        
        🔥 What is OSIV?
    Open Session In View (OSIV) is a pattern in Spring Boot where:
    
    👉 The Hibernate Session stays open for the entire HTTP request, not just during the database/transaction layer.
        👉 With OSIV enabled:
        ✅ Yes — the same Persistence Context is used for the whole HTTP request
        Spring Boot has this enabled by default:
        
        spring.jpa.open-in-view=true
        🧠 What OSIV does
        
        Keeps the Hibernate Session OPEN even after the transaction ends, until the response is fully rendered (serialization done)

⚠️ Why OSIV is considered BAD in production

Even though it's convenient, it has serious downsides:

❌ 1. Hidden database queries (very dangerous)

    return user;
    Looks simple, but during serialization:
    user.getOrders() → triggers query
    user.getOrders().getItems() → more queries
    👉 This causes N+1 query problem

❌ 2. Poor performance

    Queries happen late (during JSON conversion)
    Hard to track and optimize
    Can slow down APIs significantly

❌ 3. Breaks separation of concerns

    Controller layer starts triggering DB queries
    Business logic leaks into view layer

❌ 4. Hard to debug

    SQL runs outside transaction
    Logs don’t clearly show where queries originate

![img.png](Images/OneToOne26.png)

![img_1.png](Images/OneToOne27.png)

![img_2.png](Images/OneToOne28.png)

![img_3.png](Images/OneToOne29.png)

![img_4.png](Images/OneToOne30.png)

![img_5.png](Images/OneToOne31.png)

![img_6.png](Images/OneToOne32.png)

![img_7.png](Images/OneToOne33.png)

![img_8.png](Images/OneToOne34.png)

![img_9.png](Images/OneToOne35.png)


@JsonIgnore is an annotation from the Jackson library (com.fasterxml.jackson.annotation) used in Java (commonly in Spring Boot) to exclude a field or method from JSON serialization and/or deserialization.

🔹 Simple meaning

It tells Jackson:
👉 “Ignore this property when converting between Java object ↔ JSON”


🔥 UPDATE Scenarios (Unidirectional OneToOne)
    
    Entity (same as yours)
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "add_foreign_key", referencedColumnName = "addId")
    private UserAddress userAddress;

🔹 SCENARIO 1: Update only parent (no change in child)

    UserClass user = userRepo.findById(id).get();
    user.setUserName("New Name");
    userRepo.save(user);

✅ Result:

    Only UserClass updated
    UserAddress untouched
    No extra query for child

👉 Cascade does NOT trigger because child is not modified

🔹 SCENARIO 2: Update child via parent (WITH cascade)

    UserClass user = userRepo.findById(id).get();
    user.getUserAddress().setAddress("New Address");
    userRepo.save(user);

✅ Result:

    UserAddress updated automatically
    No need to call userAddressRepo.save()

👉 Because of:

    cascade = CascadeType.ALL

🔹 SCENARIO 3: Update child via parent (WITHOUT cascade)

    user.getUserAddress().setAddress("New Address");
    userRepo.save(user);

❌ Important:

    If transaction is active → it may still work (dirty checking)
    If detached object / no transaction → ❌ child NOT updated

👉 Safe approach:

    userAddressRepo.save(user.getUserAddress());

🔹 SCENARIO 4: Replace child object (NEW address)

    UserAddress newAddress = new UserAddress();
    newAddress.setAddress("Brand new");
    user.setUserAddress(newAddress);
    userRepo.save(user);
👉 With Cascade

✅ Works:

    New address inserted
    FK updated in UserClass
👉 Without Cascade

❌ Error: TransientPropertyValueException

👉 Same reason as insert scenario

🔹 SCENARIO 5: Remove child reference (set null)

    user.setUserAddress(null);
    userRepo.save(user);

👉 With or Without Cascade

✅ Result:

Foreign key becomes NULL

⚠️ BUT:

Old UserAddress row still exists in DB (orphan)
🔥 IMPORTANT (Interview Point)

To delete orphan automatically:

@OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
🔥 DELETE Scenarios

🔹 SCENARIO 1: Delete parent (WITHOUT cascade)

    userRepo.delete(user);

❌ Result:

    If FK constraint exists → ❌ Exception
    Because child still referenced

👉 DB says:

    “Cannot delete parent, child exists”

🔹 SCENARIO 2: Delete parent (WITH cascade)

    @OneToOne(cascade = CascadeType.ALL)
    userRepo.delete(user);

✅ Result:

    First deletes UserClass
    Then deletes UserAddress

👉 Actually Hibernate orders it properly:

    Delete child
    Delete parent

🔹 SCENARIO 3: Delete only child

    userAddressRepo.delete(address);

⚠️ Dangerous:

UserClass still has FK pointing to deleted row
❌ Leads to:
Constraint violation OR
Broken reference

👉 Always:

    user.setUserAddress(null);
    userRepo.save(user);
    userAddressRepo.delete(address);

🔹 SCENARIO 4: Remove relation (orphan handling)
user.setUserAddress(null);
userRepo.save(user);
👉 Without orphanRemoval
FK = NULL
Child still exists ❗
👉 With orphanRemoval = true
@OneToOne(orphanRemoval = true)

✅ Result:

Child row automatically deleted

🔥 SCENARIO 5: Delete parent when child is LAZY

No difference in behavior for delete.

👉 Fetch type does NOT affect delete
(it only affects loading)


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


ONE TO ONE BIDIRECTIONAL

    Bidirectional mapping:
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "add_foreign_key") // owning side
    private UserAddress userAddress;
    @OneToOne(mappedBy = "userAddress") // inverse side
    private UserClass user;

❌ Case 1: Only inverse side set

    address.setUser(user);
    userRepo.save(user);

💥 Result:

        FK NOT updated
        Relationship NOT saved

👉 Even though cascade is present!

✅ Case 2: Only owning side set

    user.setUserAddress(address);
    userRepo.save(user);

✔️ DB works fine
✔️ FK updated

⚠️ BUT:

In-memory object graph is inconsistent
address.getUser() → NULL
✅✅ BEST PRACTICE (ALWAYS DO THIS)

    user.setUserAddress(address);
    address.setUser(user);

👉 This is the real difference from unidirectional

🔥 UPDATE Case (Important nuance)
Updating child:

    user.getUserAddress().setAddress("New");

✔️ Works same as unidirectional

BUT:

👉 If you replace the child:

    UserAddress newAddress = new UserAddress();
    user.setUserAddress(newAddress);

⚠️ You must also do:

newAddress.setUser(user);

Otherwise:

    Inverse side broken
    Future operations buggy