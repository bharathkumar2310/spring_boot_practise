One To Many Unidirectional :

```java 
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;

    private String name;

    @OneToMany
    private List<Products>productsList;

}
```


```java

package com.example.jpa.OneToMany;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Products {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;

    private String productItem;
}

```


In one to many unidirectional when join column is not provided it creates a new table having both foreign keys

in this scenario say 

Without cascade

SCENARIO 1 : 
1. without cascade when we try to save customer

```java
 @PostMapping("/saveCustomer")
    public Customer saveCustomer(@RequestBody Customer customer) {
        return customerRepo.save(customer);
    }

```

Customer customer = 

            {
                "name" : "Loosu",
                "productsList" : [
                {"productItem" : "Soap"},
                {"productItem" : "Shampoo"}
                ]
            }


Here customer has refernce to products but while saving customer though it doesnot have foreign key(new table is present for foreign key)  it checks if the child is persisted or not 
but the child is in transient state only

So it will throw Persistent instance of 'com.example.jpa.OneToMany.Customer' references an unsaved transient instance of 'com.example.jpa.OneToMany.Products' (persist the transient instance before flushing)


But when u save product  like this

                {"productItem" : "Soap"},

Product gets saved but foreign key table will not be updated


SCENARIO 2 : Without cascade , with referencing child objects , we first save chid and then parent

    @PostMapping("/saveCustomer2")
    public Customer saveCustomer2(@RequestBody Customer customer) {
        for(Products product : customer.getProductsList()) {
        productsRepo.save(product);
        }
        return customerRepo.save(customer);
    }


Works perfectly fine first saves products, then saves customer , then creates a relation ship entry in new table



SCENARIO 3:  Without cascade when you try to save customer alone without referencing Product

    {
        "name" : "Loosu"
    }

Customer alone gets saved and no relation ship table entry is created




NOw if we put joinColum in OneToMany instead of creating a new table for relation ship foreig key is created in the many side


```java 
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long Id;

    private String name;

    @OneToMany
    @JoinColumn(name = "customer_id" , referencedColumnName = "Id")
    private List<Products>productsList;

}
```

In scenario 1 and 3 same things happen
In scenario 2 same result happens but since instead of table it uses foreign key in many side

1. Saves products first with foreign key as null  and then saves customer  then uses customerId and update it for Products foreign key




WITH CASCADE : 


It will save customer first, then products with foreign key as null then updates the product with customer foreign key
Hibernate sometimes optimizes and directly inserts with FK:

“In unidirectional OneToMany with JoinColumn, Hibernate may require an extra update because the child entity doesn’t own or know the relationship, so FK cannot always be set during insert.”

So this is slightly inefficient so always use bidirectional

![img.png](Images/OM1.png)

![img_1.png](Images/OM2.png)

![img_2.png](Images/OM3.png)

![img_3.png](Images/OM4.png)

![img_4.png](Images/OM5.png)

![img_5.png](Images/OM6.png)

![img_6.png](Images/OM7.png)

![img_7.png](Images/OM8.png)

![img_8.png](Images/OM9.png)

![img.png](Images/OM10.png)

![img_1.png](Images/OM11.png)

![img_2.png](Images/OM13.png)

![img_3.png](Images/OM14.png)

![img_4.png](Images/OM15.png)

![img_5.png](Images/OM16.png)

![img_6.png](Images/OM17.png)

![img_7.png](Images/OM18.png)

![img_8.png](Images/OM19.png)

by default, orphanRemoval is false


FETCH TYPE :

@oneToMany is lazy by default
If u fetch customer it will not fetch products 


for Lazy :

When spring.jpa.open-in-view=false

SCENARIO 1 :

    @GetMapping("/findById1")
    public Customer findById1(@RequestParam Long id) {
        return customerRepo.findById(id).orElse(null);
    }

Here since it is lazy loading jpa will only fetch customer and whie serilaizing it will try to do another query to get products
but since spring.jpa.open-in-view=false the session is aready closed and we cant make a query 
so we get Could not write JSON: Cannot lazily initialize collection of role 'com.example.jpa.OneToMany.Customer.productsList' with key '1' (no session)]


SCENARIO 2 : 

    @GetMapping("/findById2")
    @Transactional
    public Customer findById2(@RequestParam Long id) {
        return customerRepo.findById(id).orElse(null);
    }

Same as scenrio 1 will get Cannot lazily initialize collection of role 'com.example.jpa.OneToMany.Customer.productsList' with key '1' (no session)]


SCENARIO 3 : 

    @GetMapping("/findById3")
    public Customer findById3(@RequestParam Long id) {
        Customer customer =  customerRepo.findById(id).orElse(null);
        customer.getProductsList().size();
        return customer;
    }
Same as scenrio 1 will get Cannot lazily initialize collection of role 'com.example.jpa.OneToMany.Customer.productsList' with key '1' (no session)]


SCENARIO 4 :

    @GetMapping("/findById4")
    @Transactional
    public Customer findById4(@RequestParam Long id) {
        Customer customer =  customerRepo.findById(id).orElse(null);
        customer.getProductsList().size();
        return customer;
    }

Since we put transactional session is not closed when we call customer.getProductsList().size(), so hibernate can make a sql query and fetch customer along with products


When spring.jpa.open-in-view=true all 4 scenarios will give correct output since session will be open till serialization happens


🔥 When do you NEED cascade for delete?
🔴 Case 1: FK exists in child (@JoinColumn)
@OneToMany
@JoinColumn(name = "customer_id")
List<Products> productsList;

👉 DB:

Products → customer_id (FK)
❌ If you delete customer WITHOUT cascade
customerRepo.delete(customer);

👉 DB state:

Products still reference customer_id ❌

💥 Result:

ForeignKeyConstraintViolationException
✅ Fix options
✔️ Option 1: Use cascade
@OneToMany(cascade = CascadeType.REMOVE)

👉 Hibernate:

Deletes products
Then deletes customer
✔️ Option 2: Manually delete children
productsRepo.deleteAll(customer.getProductsList());
customerRepo.delete(customer);
🔥 Case 2: Join Table (NO @JoinColumn)
@OneToMany
List<Products> productsList;

👉 DB:

Customer
Products
Customer_Products (join table)
❌ Delete without cascade
customerRepo.delete(customer);

👉 Hibernate:

Deletes join table rows
Deletes customer

✅ Products remain
✅ No exception



-------------------------------------------------------------------------------------------------------------------------------------------------



