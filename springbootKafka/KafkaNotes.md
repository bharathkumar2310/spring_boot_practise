Kafka:

    ---> Kafka is a distributed event streaming platform designed for high throughput and real time data processing 
    ---> It acts as Distributed publish-subscribe messaging system where we can publish and subscribe to consume real time data
    while maintaining scalability and fault tolerance


    ---> Distributed means the sytem runs on different machines rather than a single one
             ---> Kafka runs as cluster of servers called brokers

    ---> high throughput refers to amount of data processed within a second
          --->Kafka can handle a very large number of messages per second efficiently using partitioning, batching, and sequential disk writes.



Challenges without Kafka : 

       ---> Without Kafka every Microservice should communicate with each other like this

                    Order → Payment
                    Order → Inventory
                    Order → Notification

       This causes 
                1. tight coupling :
                2. hard to scale and maintain
                3. less fault tolerance

✅ 1. Decoupling Services

Say if one of Payment, Inventory, Notifaction fails/ slows down Order service also fails/ slows down

                --With kafka orderService just publishes events to Kafka , irrespective of other MS fails/slows down
                -- These events will be stored in Kafka and whenerever other MS starts working it can consume from Kafka

               --Thus kafka ensures high fault tolerance and helps in decoupling 2 diff MS

✅ 2. Handling High Traffic (Scalability)

Without Kafka: Direct REST calls can overload services
Difficult to handle spikes (e.g., Big Billion Day sale)

            -With Kafka: Kafka stores events in partitions
            --Multiple consumers can process in parallel
            --Easily scalable horizontally


✅ 3. Reliability and Fault Tolerance

Without Kafka : If any MS crashes data is lost

        --With Kafka data is stored in disc

✅ 4. Asynchronous Communication


**_Kafka is needed when you want scalable, reliable, fault-tolerant, asynchronous communication between services._**