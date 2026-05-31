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



ZOOKEEPER :

    ZooKeeper is a distributed co-ordinating system
    Think of it as a manager/registry that keeps track of everything happening in a distributed system.

🔧 What it does in Kafka:

    Keeps track of which brokers are alive
    Maintains cluster metadata (topics, partitions, leaders)
    Performs leader election (decides which broker handles a partition)
    Stores configuration info


⚙️ Case 1: ZooKeeper is temporarily down (cluster already running)

✅ What still works:
    
    Producers can send messages
    Consumers can read messages
    Existing brokers continue working

👉 Why?
Because brokers already have metadata cached locally
    
    ❌ What breaks:
    ❌ No new topic creation
    ❌ No partition reassignment
    ❌ No new broker registration
    ❌ No leader election if a broker fails


FLOW :

    1. Start ZooKeeper
       2. Start Kafka Broker
       3. Broker connects to ZooKeeper
       4. Broker creates an ephemeral znode in ZooKeeper
       5. ZooKeeper now knows the broker is alive
    
    broker.id=1
    zookeeper.connect=localhost:2181
    
    
    When the broker is down ZooKeeper knows and does leader allocation


**_BROKER :_**

    Broker is a server that stores and serves data

🔧 Responsibilities of a Broker:

    Stores messages (data) in topics
    Handles producer writes
    Handles consumer reads
    Manages partitions


🔹 Broker is created by:

    Running Kafka server
    bin/kafka-server-start.sh config/server.properties

👉 That starts a broker instance


A producer sends messages to a broker, not directly to a topic.
However, the message is sent for a specific topic, and the broker stores it in that topic's partition.

Step-by-step

Suppose:
    
    Topic: orders
    Partitions: P0, P1, P2

Producer sends:

    producer.send(new ProducerRecord<>("orders", "Order #123"));

Flow:
        
        Producer
        |
        | 1. Sends message for topic "orders"
        v
        Broker (leader of the target partition)
        |
        | 2. Appends message to partition log
        v
        orders-P1
        How does the producer know which broker?
        
        Before sending data, the producer asks Kafka metadata:
        
        Producer
        |
        | Metadata Request
        v
        Broker
        
        Broker responds:
        
        Topic orders
        P0 -> Broker 1 (Leader)
        P1 -> Broker 2 (Leader)
        P2 -> Broker 3 (Leader)
        
        Now the producer knows:
        orders-P1 leader = Broker 2
        and sends the message directly to Broker 2.



When a consumer consumes messages, it also talks to brokers, not directly to ZooKeeper (or KRaft controllers).

Consumer flow

Suppose:
    
    Topic: orders
    Partition P0
    Leader: Broker 1
    1. Consumer starts
       Consumer
       |
       | Metadata Request
       v
       Broker
    
    The broker returns:
    
    orders
    P0 -> Broker 1
    2. Consumer joins a consumer group
    
    If the consumer belongs to a group:
    
    group.id=order-group
    
    it joins the group and gets assigned partitions.
    
    Example:
    
    Consumer A -> P0
    Consumer B -> P1
    Consumer C -> P2
    3. Consumer fetches messages
    
    The consumer sends a Fetch Request to the broker that is the leader for its partition.
    
    Consumer A
    |
    | Fetch messages from P0
    v
    Broker 1
    
    Broker responds:
    
    Offset 0 -> Order1
    Offset 1 -> Order2
    Offset 2 -> Order3

    4. Consumer processes messages
       Read Order1
       Read Order2
       Read Order3
       5. Consumer commits offset
    
    After processing, the consumer records its progress.
    
    Example:
    
    Last processed offset = 2
    Next offset to read = 3

Kafka stores this offset so that if the consumer restarts, it can continue from where it left off.


**_Producer :_**

    A Producer sends data to Kafka.
    Producer is responsible for publishing records to Kafka topics with partitioning and delivery guarantees.

Key points:

    Sends messages to a topic
Can choose partition:
    
    Round-robin
    Key-based (same key → same partition)
Supports:
    
    Retries
    Batching
    Compression



    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
    </dependency>




    
    spring.kafka.bootstrap-servers=localhost:9092
    
    //Producer configs
    spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
    spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
    
    //Reliability (important)
    spring.kafka.producer.acks=all
    spring.kafka.producer.retries=3
    
    spring.kafka.producer.enable-idempotence=true
    spring.kafka.producer.max-in-flight-requests-per-connection=5
    



```java

@Service
public class KafkaProducerService {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    private static final String TOPIC = "my-topic";

    public void sendMessage(String message) {
        kafkaTemplate.send(TOPIC, message);
    }
}

```


```java

@RestController
@RequestMapping("/kafka")
public class KafkaController {

    @Autowired
    private KafkaProducerService producerService;

    @PostMapping("/send")
    public String send(@RequestParam String msg) {
        producerService.sendMessage(msg);
        return "Message sent";
    }
}

```
// Sending with key

    kafkaTemplate.send("my-topic", "order123", message);


Topic

    A Topic is a logical category of messages.
    👉 It’s NOT a physical storage—just a logical name
    
    Kafka clients use topics to produce and consume data
    Say a producer produces data to a topic means these data are logically classified as this topic entities
  
Example:

    orders
    payments
    logs

**_Partition (Very Important) :**_

    Each topic is split into partitions.
    Partition is where message live inside topics
    Each topic will create one or more partitions
    Each partition is an ordered immutable sequence of records
    Each record inside a partition is assigned a sequential number called offsets
    Ordering can be done inside partitions
    Partitions are the physical storage units where messages are stored.

Why partitions?
    
    Parallelism
    Scalability

Each partition:
    
    Is an ordered, immutable log
    Has offsets (0,1,2,3...)


1. Parallel Consumption

        Imagine a topic: orders  with 1 partition:
        
        orders
        |
        +--> P0
        
        If you have 3 consumers in the same consumer group:
        
        Consumer A
        Consumer B
        Consumer C
        
        Only one consumer can read that partition.
        
        P0 -> Consumer A
        
        Consumers B and C stay idle.
        
        Now with 3 partitions:
        
        orders
        |
        +--> P0
        +--> P1
        +--> P2
        
        Kafka can assign:
        
        P0 -> Consumer A
        P1 -> Consumer B
        P2 -> Consumer C
        
        Now all consumers work in parallel.

2. Higher Throughput

       Suppose one broker can handle: 1000 messages/sec

With 1 partition on 1 broker: 1000 msgs/sec

With 10 partitions spread across 5 brokers:

    Broker 1 -> P0, P1
    Broker 2 -> P2, P3
    Broker 3 -> P4, P5
    Broker 4 -> P6, P7
    Broker 5 -> P8, P9

You can process many times more messages because work is distributed.

3. Distribution Across Brokers

        Without partitions:
        
        orders
        |
        v
        Broker 1
        
        If Broker 1 fills up its disk, you're stuck.
        
        With partitions:
        
        P0 -> Broker 1
        P1 -> Broker 2
        P2 -> Broker 3
        
        Storage and network load are distributed.

   4. Fault Tolerance

           Partitions can be replicated.
        
           Example:
            
               P0 Leader   -> Broker 1
               P0 Follower -> Broker 2
        
           If Broker 1 crashes:
        
           Broker 2 becomes leader
        
           Consumers and producers continue working.

Example:
    
    Partition 0 → [msg1, msg2, msg3]
    Partition 1 → [msgA, msgB]
    
    👉 Ordering is guaranteed only within a partition

    Kafka messages sent from producer has 2 properties (key, value)
    Messages from same key go to same partition so that consumer can read sequentially

    
    1 partition → only 1 consumer in a group
    1 consumer → can read many partitions

**_Broker :**_

    A Broker is a Kafka server.

Responsibilities:
    
    Stores partitions
    Handles read/write requests
    Replicates data

Cluster:

    Multiple brokers = Kafka cluster

**Consumer :**

    A Consumer reads data from Kafka.

Key points:
    
    Pull-based (consumer requests data)
    Reads using offsets
    Can replay data anytime

    
    # -----------------------------
    # Kafka Broker
    # -----------------------------
    spring.kafka.bootstrap-servers=localhost:9092
    
    # -----------------------------
    # Consumer Basics
    # -----------------------------
    spring.kafka.consumer.group-id=my-group
    spring.kafka.consumer.auto-offset-reset=earliest
    
    # -----------------------------
    # Deserialization
    # -----------------------------
    spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
    spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
    
    # -----------------------------
    # Offset Commit
    # -----------------------------
    spring.kafka.consumer.enable-auto-commit=true
    spring.kafka.consumer.auto-commit-interval=1000
    
    # -----------------------------
    # Concurrency (parallel consumers)
    # -----------------------------
    spring.kafka.listener.concurrency=3

```java
@Service
public class KafkaConsumerService {

    @KafkaListener(topics = "my-topic", groupId = "my-group")
    public void consume(String message) {
        System.out.println("Received: " + message);
    }
}
```
here groupId is consumer group




**_Consumer Group :**_

    A Consumer Group = multiple consumers working together.

    When producres produce more messages than consumer polls , then consumer groups are created(Insatnce of consumers are created)
    Used for scalable method consumption
    
    no two consumer from same consumer group can poll same partition
    
    If there are 5 partition and 3 consumer in a consumer group 3 partition --> consumer 1, 1 partition --> consumer 2, 1 partition --> consumer 3
    


Rules:
    
    Each partition → consumed by only ONE consumer in group
    Different groups → can read same data independently
Example:
    
    Topic has 3 partitions
    Group has 3 consumers → parallel processing

-----------------------------------------------------------------------------------------------------------------------------------

**_3.1 Replication :**_

Each partition has:
    
    Leader
    Followers

Flow:
    
    Producer writes → Leader
    Followers replicate

**_🧠 3.2 ISR (In-Sync Replicas)**_

    ISR = replicas that are fully caught up.

Why important?

    Only ISR replicas can become leader

**_⚖️ 3.3 Acknowledgments (acks)**_

Producer config:
    
    acks=0 → fire and forget
    acks=1 → leader only
    acks=all → leader + ISR (strongest)

**_🔄 3.4 Offset**_

    Offset = unique ID for each message in a partition

Used for:
    
    Tracking consumption
    Reprocessing data

4.1 Log (Commit Log)

Kafka stores data as:

    Append-only log files

Key properties:
    
    Sequential writes (fast)
    Immutable (no updates)

🧹 4.2 Log Retention

    Kafka doesn’t delete immediately.

Policies:
    
    Time-based (e.g., 7 days)
    Size-based

🧼 4.3 Log Compaction

    Keeps only latest value per key.

Used for:
    
    State systems
    CDC (Change Data Capture)

⚙️ 5. Metadata & Coordination

🧭 5.1 ZooKeeper (Old)

Apache ZooKeeper
        
        Tracks brokers
        Leader election
        Stores metadata

⚡ 5.2 KRaft (New)

Kafka replaces ZooKeeper with:

    Internal Raft-based system

👉 Interview line:

Modern Kafka uses KRaft instead of ZooKeeper for metadata management.



KAFKA REPLICATION :

👉 Replication means:

    Kafka stores multiple copies of the same partition across different brokers

Why?
    
    Fault tolerance
    High availability
    Data durability

📦 Partition Replicas

For each partition:
    
    1 Leader
    0 or more Followers

👑 Leader
    
    Handles all reads and writes
    Producers & consumers talk only to leader

👥 Followers
    
    Copy data from leader
    Do not serve client requests

🔢 Replication Factor (RF)

    👉 Number of copies of a partition

Example:

    RF = 3 → 1 leader + 2 followers

🔄 4. How replication works
Step-by-step flow:
    
    Producer sends message → Leader
    Leader writes to its log
    Followers pull data from leader
    Followers store the same data

👉 Important:

    Kafka uses pull-based replication, not push

🔄 Exact mechanism

Each follower continuously does:

Sends FetchRequest to leader
Includes:
    
    Partition
    Current offset it has
Leader responds with:
        New records (if any)



What is ISR exactly (deep view)?

    ISR = In-Sync Replicas
    👉 But not just “caught up”—it’s stricter.

✅ Conditions to be in ISR

A follower is in ISR if:
    
    It is alive
    It is fetching data regularly
    Its lag is within threshold

🔧 Key configs
    
    replica.lag.time.max.ms
    replica.lag.max.messages (older)

👉 If follower doesn’t fetch within time → removed from ISR

📉 Example

    Leader → offset 100
    
    Follower B2 → offset 100 ✅ (in ISR)
    Follower B3 → offset 80 ❌ (out of ISR)

ISR = [Leader, B2]

🧠 3. High Watermark (CRITICAL CONCEPT)

    This is where most people fail interviews.

👉 High Watermark (HW) =

    The highest offset that is fully replicated across ISR

Why needed?

    To avoid this problem:
    
    Leader writes message
    Crashes before followers copy it
    New leader doesn’t have that message

👉 → data inconsistency

🔒 Solution

    Consumers can only read up to High Watermark

📦 Example
Leader log:      [0,1,2,3,4,5]
Follower log:    [0,1,2,3]

ISR = [Leader, Follower]

High Watermark = 3

👉 Consumers see only up to 3, NOT 5

🧠 4. How acknowledgments (acks) work internally

🔁 Step-by-step with acks=all
    
    Producer sends message → Leader
    Leader writes to its log
    Followers fetch the data
    Once all ISR replicas confirm, leader updates HW
    Leader sends ACK to producer

⚖️ Difference between acks

    🔹 acks=0
        
    No waiting
    Fast but unsafe

🔹 acks=1
    
    Leader writes → ACK immediately
    Followers may not have data
    Risk of data loss

🔹 acks=all (or -1)

    Wait for ISR replication
Safest

    🔥 Important nuance

    ACK is sent only after message reaches all ISR, NOT all replicas



Example with ack = all


Suppose:

    Partition P0
    
    Broker 1 -> Leader
    Broker 2 -> Follower
    Broker 3 -> Follower
    
    Normally all replicas are caught up:
    
    ISR = {Broker1, Broker2, Broker3}
    
    Now a producer sends a message.
    
    Producer
    |
    v
    Leader (Broker1)
    
    The leader writes the message and replicates it to Broker2 and Broker3.
    
    Only after all ISR members have the message does Kafka send:
    
    ACK -> Producer
    What if one replica falls behind?
    
    Suppose Broker3 becomes slow:
    
    Broker1 -> Offset 100
    Broker2 -> Offset 100
    Broker3 -> Offset 80
    
    Broker3 is no longer "in sync".
    
    Kafka removes it from ISR:
    
    ISR = {Broker1, Broker2}
    
    Now when a new message arrives:
    
    Producer
    |
    v
    Broker1
    
    Kafka only waits for:
    
    Broker1
    Broker2
    
    It does NOT wait for Broker3.
    
    Then ACK is sent.
    
    Producer <- ACK
    Why?
    
    If Kafka waited for every replica, one slow broker could block the entire cluster.
    
    Instead Kafka waits only for replicas that are considered healthy and caught up (ISR).
    
    Example
    
    Replication Factor = 3
    
    Broker1 -> Leader
    Broker2 -> Follower
    Broker3 -> Follower (slow)
    
    Current ISR:
    
    {Broker1, Broker2}
    
    Producer sends message M1.
    
    M1 -> Broker1
    M1 -> Broker2
    
    ACK sent immediately.
    
    Kafka does not wait for Broker3.
    
    Later Broker3 catches up:
    
    M1 -> Broker3

🧠 5. Putting it all together
🔄 Full flow
    
    Producer → Leader
    Leader → write to log
    Followers → pull (fetch)
    Followers → replicate
    Leader → updates High Watermark
    Leader → sends ACK (based on acks config)
    Consumer → reads only up to HW

🚨 6. What if follower is slow?
    
    Stops fetching → removed from ISR
    Now ISR shrinks

👉 Effect:
    
    acks=all becomes faster (fewer replicas to wait for)
    But fault tolerance reduces ⚠️

🚨 7. What if leader crashes before HW update?

👉 That data is lost

BUT:

    Consumers never saw it (because HW not updated)

👉 Kafka ensures consistency over availability


-----------------------------------------------------------------------------------------------------------


. Start ZooKeeper (Step-by-step)

    ✅ Step 1: Go to Kafka folder
    ZooKeeper comes bundled with Apache Kafka.

cd kafka/
✅ Step 2: Start ZooKeeper server
🔹 Linux / Mac:

    bin/zookeeper-server-start.sh config/zookeeper.properties
🔹 Windows:

    bin\windows\zookeeper-server-start.bat config\zookeeper.properties
✅ Step 3: Verify it started

You’ll see logs like:

    binding to port 0.0.0.0/0.0.0.0:2181

    👉 Default port: 2181

⚡ 3. Start Kafka (after ZooKeeper)

    bin/kafka-server-start.sh config/server.properties

👉 Important:

    ZooKeeper must be running before Kafka (in older versions)

🧪 4. Optional: Test ZooKeeper

Run:

    bin/zookeeper-shell.sh localhost:2181

You should get a shell like:

    [zk: localhost:2181(CONNECTED) 0]


👉 In newer Kafka versions:

    You don’t need ZooKeeper at all

Instead, Kafka runs in KRaft mode.

    Start Kafka WITHOUT ZooKeeper (KRaft mode)

    bin/kafka-storage.sh format -t <uuid> -c config/kraft/server.properties
    bin/kafka-server-start.sh config/kraft/server.properties
🎯 Interview Tip

If asked:

“How do you start Kafka?”

Say:

    In older versions, we start ZooKeeper first and then Kafka broker. In newer versions using KRaft mode, Kafka can be started independently without ZooKeeper.


👉 To create multiple brokers:

    Start Kafka multiple times
    Each instance must have:
        Unique broker.id
        Unique port
        Separate log directory

⚙️ 2. Step-by-step (local setup)
✅ Step 1: Duplicate config file

Go to:

    config/server.properties

Create copies:
    
    server-1.properties
    server-2.properties
    server-3.properties

✅ Step 2: Modify each config

🔹 Broker 1

    broker.id=1
    listeners=PLAINTEXT://localhost:9092
    log.dirs=/tmp/kafka-logs-1
🔹 Broker 2
    
    broker.id=2
    listeners=PLAINTEXT://localhost:9093
    log.dirs=/tmp/kafka-logs-2
🔹 Broker 3
    
    broker.id=3
    listeners=PLAINTEXT://localhost:9094
    log.dirs=/tmp/kafka-logs-3

⚠️ Important:
    
    Ports must be different
    Log dirs must be different
    broker.id must be unique

✅ Step 3: Start ZooKeeper (old mode)

    bin/zookeeper-server-start.sh config/zookeeper.properties
✅ Step 4: Start brokers

Open 3 terminals:
    
    bin/kafka-server-start.sh config/server-1.properties
    bin/kafka-server-start.sh config/server-2.properties
    bin/kafka-server-start.sh config/server-3.properties

🧪 3. Verify brokers

Run:

    bin/zookeeper-shell.sh localhost:2181 ls /brokers/ids

👉 Output:

[1, 2, 3]
📦 4. Create topic with replication
    
    bin/kafka-topics.sh --create \
    --topic my-topic \
    --partitions 3 \
    --replication-factor 3 \
    --bootstrap-server localhost:9092

👉 Now:

Each partition has:
    
    1 leader
    2 followers

⚡ 5. Modern Kafka (KRaft mode)

No ZooKeeper needed.

You still:
    
    Run multiple brokers
    But configs use config/kraft/server.properties

-----------------------------------------------------------------------------------------------------------------------





















🧠 5. ISR (In-Sync Replicas)

ISR = replicas that are fully caught up

Includes leader + healthy followers
Only ISR replicas are considered safe
Example:
Leader → B1
Follower → B2 (in sync)
Follower → B3 (lagging ❌)

ISR = [B1, B2]
⚖️ 6. Acknowledgment (acks)

Controls durability from producer side:

acks=0 → no guarantee
acks=1 → only leader writes
acks=all → leader + ISR confirm ✅
🔥 Important line:

Highest reliability = acks=all + replication

🚨 7. What happens if leader fails?
Scenario:

Leader = B1 crashes

Steps:
Kafka detects failure
Picks new leader from ISR
Example:
New Leader → B2
Follower → B3

👉 No data loss (if ISR was up-to-date)

⚠️ 8. What if follower is slow?

If follower lags:

Removed from ISR

👉 Why?

To prevent stale data becoming leader
⚠️ 9. Unclean Leader Election (VERY IMPORTANT)

Edge case:

No ISR available ❌

Two options:
1. Wait for ISR (safe)
   No data loss
   But downtime
2. Elect out-of-sync replica
   Faster recovery
   ⚠️ Data loss possible

Config:

unclean.leader.election.enable=true
🧱 10. Where data is stored?

Each replica stores:

Same log file
Same offset sequence

👉 Kafka = distributed commit log

⚡ 11. Replication guarantees

Kafka provides:

At least once (default)
Exactly once (with idempotent producer + transactions)
🧠 12. Important Interview Concepts
🔥 Q: Why not replicate in same broker?

Because if broker fails, all replicas are lost

🔥 Q: Can consumer read from follower?

No (by default), only from leader

🔥 Q: What ensures no data loss?

ISR + acks=all + proper replication factor


-----------------------------------------------------------------------------------------------------------------------------


Is ISR an array?
    
    👉 Conceptually: YES
    👉 Internally: it’s a set/list of replica IDs

Think of ISR like:

    ISR = {Leader, Follower1, Follower2}

But:
    
    It’s dynamic
    Brokers are added/removed automatically
Example

    Replicas: [B1, B2, B3]
    
    Leader = B1
    Followers = B2, B3

ISR = [B1, B2]   (B3 is slow → excluded)

👉 So:
    
    All replicas ≠ ISR
    ISR ⊆ Replicas

🧠 2. What does acks=all REALLY mean?

This is the key confusion 👇

❌ Wrong understanding
    
    “Data is sent to all followers”
    Kafka does NOT push data to followers

✅ Correct understanding

    Leader waits until all ISR replicas have pulled and stored the data, then sends ACK

🧩 Step-by-step (important)

Let’s say:
    
    Replicas = [B1, B2, B3]
    ISR = [B1, B2]   (B3 is out of sync)
    When producer sends message with acks=all:
    Producer → Leader (B1)
    Leader writes message

Followers fetch:
    
    B2 fetches → ✅ (in ISR)
    B3 fetches → irrelevant (not in ISR)
    Once B1 + B2 have data → ACK sent

👉 So:

    acks=all = wait for all ISR, NOT all replicas

🧠 3. Why not wait for all replicas?

Because:
    
    Some replicas may be slow or down
    Waiting for all → system becomes slow/unavailable

👉 ISR ensures:

    Only healthy replicas matter

🧠 4. Important relationship

    All Replicas = Leader + Followers
ISR ⊆ All Replicas
🧠 5. One critical insight (interview gold)

👉 If ISR shrinks:
    
    Before:
    ISR = [B1, B2, B3]
    
    After B3 slow:
    ISR = [B1, B2]

Now:
    
    acks=all waits for only B1 & B2
    System becomes faster ⚡
    But less fault-tolerant ⚠️

🧠 6. What if ISR = only Leader?
ISR = [B1]

👉 Then:

acks=all = same as acks=1
No real fault tolerance



Producer properties :

# -----------------------------
# Kafka Broker Connection
# -----------------------------
spring.kafka.bootstrap-servers=localhost:9092

# -----------------------------
# Key & Value Serialization
# -----------------------------
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

# -----------------------------
# Reliability & Delivery
# -----------------------------
spring.kafka.producer.acks=all
spring.kafka.producer.retries=5
spring.kafka.producer.enable-idempotence=true

# -----------------------------
# Performance Tuning
# -----------------------------
spring.kafka.producer.batch-size=16384
spring.kafka.producer.linger-ms=5
spring.kafka.producer.buffer-memory=33554432

# -----------------------------
# Ordering & In-flight Requests
# -----------------------------
spring.kafka.producer.max-in-flight-requests-per-connection=5

# -----------------------------
# Compression (optional)
# -----------------------------
spring.kafka.producer.compression-type=snappy

# -----------------------------
# Timeout Settings
# -----------------------------
spring.kafka.producer.delivery-timeout-ms=120000
spring.kafka.producer.request-timeout-ms=30000

# -----------------------------
# Optional: Default Topic
# -----------------------------
spring.kafka.template.default-topic=my-topic


| Property            | What it does           | Why it matters                |
| ------------------- | ---------------------- | ----------------------------- |
| `bootstrap-servers` | Kafka broker address   | Entry point to cluster        |
| `key-serializer`    | Converts key → bytes   | Required for sending keys     |
| `value-serializer`  | Converts value → bytes | Required for sending messages |

| Property             | Meaning                  | Deep Insight                            |
| -------------------- | ------------------------ | --------------------------------------- |
| `acks`               | When broker sends ACK    | `all` = waits for ISR (safest)          |
| `retries`            | Retry failed sends       | Helps during transient failures         |
| `enable-idempotence` | Avoid duplicate messages | Guarantees exactly-once (producer side) |

| Property        | Meaning                        | Behavior                             |
| --------------- | ------------------------------ | ------------------------------------ |
| `batch-size`    | Size of batch (bytes)          | Larger batch = better throughput     |
| `linger-ms`     | Wait time before sending batch | Adds delay to collect more messages  |
| `buffer-memory` | Total memory for buffering     | Controls how much data can be queued |

| Property              | Meaning                       | Why needed       |
| --------------------- | ----------------------------- | ---------------- |
| `delivery-timeout-ms` | Total time to deliver message | Includes retries |
| `request-timeout-ms`  | Wait time for broker response | Prevents hanging |



Consumer Config :

| Property                                                          | Category       | What it does                           | Why it matters / Interview Insight                     |
| ----------------------------------------------------------------- | -------------- | -------------------------------------- | ------------------------------------------------------ |
| `spring.kafka.bootstrap-servers`                                  | Connection     | Kafka broker address                   | Entry point to cluster                                 |
| `spring.kafka.consumer.group-id`                                  | Grouping       | Consumer group identifier              | Enables load balancing & parallel consumption          |
| `spring.kafka.consumer.client-id`                                 | Identification | Unique consumer instance ID            | Useful for logging/monitoring                          |
| `spring.kafka.consumer.auto-offset-reset`                         | Offset         | Where to start if no offset            | `earliest` (read from start), `latest` (only new data) |
| `spring.kafka.consumer.enable-auto-commit`                        | Offset         | Auto commit offsets                    | Easy but risk of data loss                             |
| `spring.kafka.consumer.auto-commit-interval`                      | Offset         | Commit frequency                       | Only applies if auto-commit=true                       |
| `spring.kafka.consumer.key-deserializer`                          | Serialization  | Converts key bytes → object            | Mandatory                                              |
| `spring.kafka.consumer.value-deserializer`                        | Serialization  | Converts value bytes → object          | Mandatory                                              |
| `spring.kafka.consumer.max-poll-records`                          | Polling        | Max records per poll                   | Controls batch size                                    |
| `spring.kafka.consumer.fetch-min-bytes`                           | Fetch          | Minimum data to fetch                  | Improves throughput                                    |
| `spring.kafka.consumer.fetch-max-wait`                            | Fetch          | Max wait time for fetch                | Controls latency vs batching                           |
| `spring.kafka.consumer.max-partition-fetch-bytes`                 | Fetch          | Max data per partition                 | Prevents memory overload                               |
| `spring.kafka.consumer.session-timeout-ms`                        | Heartbeat      | Time before broker marks consumer dead | If missed → rebalance triggered                        |
| `spring.kafka.consumer.heartbeat-interval-ms`                     | Heartbeat      | Frequency of heartbeat                 | Must be < session timeout                              |
| `spring.kafka.consumer.max-poll-interval-ms`                      | Processing     | Max time between polls                 | Long processing → consumer kicked out                  |
| `spring.kafka.listener.concurrency`                               | Parallelism    | Number of consumer threads             | Max = number of partitions                             |
| `spring.kafka.listener.ack-mode`                                  | Offset Control | When offset is committed               | `manual`, `record`, `batch`, etc.                      |
| `spring.kafka.listener.poll-timeout`                              | Polling        | Time consumer waits for records        | Impacts responsiveness                                 |
| `spring.kafka.consumer.isolation-level`                           | Transactions   | Read committed/uncommitted data        | `read_committed` for EOS                               |
| `spring.kafka.consumer.properties.spring.json.trusted.packages`   | JSON           | Allowed packages for deserialization   | Needed for JSON payloads                               |
| `spring.kafka.consumer.properties.spring.json.value.default.type` | JSON           | Default class type                     | Helps deserialize JSON properly                        |



Inside partions there are multiple log segments these are where the data is logged

📦 2. What is Log Retention?

Kafka stores data as:

    Partition → Log → Log Segments → Messages

👉 Retention applies to:

Log segments, not individual messages
🧠 3. Two types of retention

    ⏱️ 1. Time-based retention

    log.retention.hours=168   # 7 days

👉 Meaning:

    Messages older than 7 days are deleted
📦 2. Size-based retention

    log.retention.bytes=1073741824   # 1 GB

👉 Meaning:
    
    If log size exceeds limit → old data deleted
    ⚡ Kafka uses whichever limit is reached first


📦 Example
    
    Segment-1 → messages (0–100)
    Segment-2 → messages (101–200)
    Segment-3 → messages (201–300)

If Segment-1 is old:

    👉 Entire segment is deleted

⚙️ 5. Log Segment (VERY IMPORTANT)

    Each partition is split into chunks:
    log.segment.bytes=1GB

👉 Why?
    
    Faster deletion
    Efficient disk usage


1. Consumer Rebalancing (MOST IMPORTANT)
   🧠 What is it?

        Rebalancing = Kafka redistributes partitions among consumers in a group

⚙️ When does rebalance happen?
    
    New consumer joins group
    Consumer crashes
    Consumer is too slow (max.poll.interval.ms)
    Topic partitions change

🔄 Step-by-step flow (VERY IMPORTANT)
    
    1. Consumers send JoinGroup request
       2. Kafka picks a Group Leader
       3. Leader decides partition assignment
       4. Partitions REVOKED from existing consumers
       5. New assignment distributed
       6. Consumers start consuming again

⚠️ Problem during rebalance

      Consumption stops temporarily
      Duplicate processing can happen
      Latency spike

🔥 3. Consumer Lag
🧠 Definition

    Lag = Latest offset - Committed offset

📦 Example
    
    Latest offset = 1000
    Consumer offset = 800

    Lag = 200
⚠️ Why lag happens?
    
    Slow consumer
    High traffic
    Rebalance
    Processing delays

🚨 Why it matters?
    
    High lag = system falling behind
    Can lead to data loss (due to retention)
📊 Monitoring tools
    
    Kafka UI
    Burrow
    Prometheus + Grafana

🔥 4. Exactly-Once Semantics (EOS)
🧠 Problem

    Retries can cause duplicates:

    Producer sends → ACK lost → Retry → duplicate message ❌
✅ Solution

    Exactly-once = No duplicates + No loss

⚙️ How Kafka achieves this
1. Idempotent Producer

        enable-idempotence=true

👉 Prevents duplicate writes

2. Transactions

        Producer can send multiple messages atomically:

Begin → Send → Commit

3. Consumer side

       isolation.level=read_committed

👉 Reads only committed data

🔄 Full flow
    
    Producer → Transaction start
    → Send messages
    → Commit transaction
    
    Consumer → reads only committed messages



----------------------------------------------------------------------------------------------------------------------------------


1. Time-based retention

Configuration: retention.ms=604800000

This is: 7 days

Example:
    
    Day 1 -> Order A
    Day 2 -> Order B
    Day 3 -> Order C

After 7 days:
    
    Order A deleted
    Order B deleted
    Order C deleted

even if nobody consumed them.

2. Size-based retention

Configuration:retention.bytes=1073741824

This is: 1 GB

Suppose: Partition size = 1.2 GB

    Kafka starts deleting the oldest log segments until size falls below 1 GB.
    
    Oldest data removed
    Newest data retained
    3. Both together
    
    You can configure both:
    
    retention.ms=7 days
    retention.bytes=1 GB

Kafka deletes data when either limit is reached.


---------------------------------------------------------------------------------------------------------------------------------

What is Read Offset?

    The read offset is where the consumer is currently reading.
    Stored by consumer

Suppose Kafka has:
    
    Offset 0 -> A
    Offset 1 -> B
    Offset 2 -> C
    Offset 3 -> D

Consumer starts:

Read Offset = 0
    Reads A:
    
    Current Read Offset = 1
    
    Reads B:
    
    Current Read Offset = 2
    
    Reads C:
    
    Current Read Offset = 3
    
    So the read offset keeps moving as the consumer reads messages.


What is Commit Offset?

The commit offset is Kafka's saved checkpoint.

It tells Kafka:

    "I have successfully processed messages up to this point."

Example:

    Offset 0 -> A
    Offset 1 -> B
    Offset 2 -> C
    Offset 3 -> D

Consumer processes:

    A
    B
    C

Then commits:

    Commit Offset = 3
    
    This means:
    
    Next time start from Offset 3
    
    because offsets 0,1,2 are already processed.
    
Why do we need commit offsets?

Imagine:
    
    Offset 0 -> A
    Offset 1 -> B
    Offset 2 -> C
    Offset 3 -> D

Consumer processes:

A
B
C

Then crashes.

Without commit offsets:
    
    Restart
    ↓
    Where should Kafka start?
    
    Kafka wouldn't know.
    
    With commit offsets:
    
    Committed Offset = 3
    
    After restart:
    
    Start from Offset 3
    
    Continue with D.

Where is it stored?

    Kafka stores committed offsets in a special internal topic:
    
    __consumer_offsets

Different Consumer Groups

Now suppose:
    
    Group A
    Group B

Both consume the same topic.

orders-P0

Then both groups have their own commit offsets.
    
    Group A Offset = 100
    Group B Offset = 45

----------------------------------------------------------------------------------------------------------------------------------


1. At Most Once

       Read Message
       ↓
       Commit Offset
       ↓
       Process Message

Code idea:

    record = consumer.poll()
    consumer.commitSync();  // commit first
    process(record);        // process later

What can go wrong?

    Message: Offset 1 -> Pay ₹200

Consumer does:
    
    Read Offset 1
    Commit Offset = 2

Now Kafka thinks:

    Offset 1 successfully handled

Then:

    Server crashes ❌
    before processing payment.

Result:

    ₹200 payment never processed
    When consumer restarts:Starts from Offset 2
    Offset 1 is permanently lost.

Why use it?

Only when losing messages is acceptable.

Examples:
    
    Website click tracking
    Analytics
    Metrics
    Monitoring

Example:
    
    User clicked a button
    Losing one click isn't critical.

Configuration

Usually commit before processing.

    enable.auto.commit=true

or 

    manually commit before business logic.

Not common for critical systems.



2. At Least Once (Most Common)
   
    
       Read Message
       ↓
       Process Message
       ↓
       Commit Offset

Code:

    record = consumer.poll();
    process(record);
    consumer.commitSync();

What can go wrong?

Message: Offset 1 -> Pay ₹200

Consumer:

    Read Offset 1
    Process Payment

Then: Crash ❌

    before commit.
    Kafka still has:
    Committed Offset = 1
    
    After restart:
    
    Read Offset 1 again
    
    Payment processed twice.
    
    ₹200
    ₹200
    
    Total:
    
    ₹400

Duplicate.

Benefit

    No message loss.
    Every message eventually gets processed.
    That's why this is Kafka's default pattern.

How to Configure

Disable auto commit:

    enable.auto.commit=false

Then:

while(true){
records = consumer.poll(...);
process(records);
consumer.commitSync();
}

Commit only after successful processing.
How companies handle duplicates?
Make business logic idempotent.

Example:

    Payment ID = PAY123
    Before processing:
    
    SELECT * FROM payments
    WHERE payment_id='PAY123'
    
    If already processed:
    
    Ignore duplicate




3. Exactly Once

Scenario

    You have: orders-topic

Message:

    Offset 10 -> Order#100

Your service:
    
    Read Order
    ↓
    Create Payment Event
    ↓
    Send to payments-topic

At Least Once Problem

Code:

    ConsumerRecord record = ...
    
    process(record);
    
    producer.send(
    new ProducerRecord<>(
    "payments-topic",
    "PAYMENT_CREATED"
    )
    );
    
    consumer.commitSync();

Crash
Read Order#100
↓
Send PAYMENT_CREATED ✅
↓
CRASH ❌
↓
commitSync() never happens
        
        Kafka thinks: Offset 10 not processed
        Restart: Read Order#100 again
        
        Produces: PAYMENT_CREATED
        
        again. 
        
        Result:
        
        payments-topic
        
        PAYMENT_CREATED
        PAYMENT_CREATED
        
        Duplicate.
        
        Exactly Once Goal
        
        We want:
        
        Either:
        
        PAYMENT_CREATED written
        AND
        Offset committed
        
        OR
        
        Neither happens
        
        No half-finished state.
        
        How Kafka Does It

Kafka transaction groups these two actions:
    
    1. Write PAYMENT_CREATED
       2. Commit consumer offset
    
    into one atomic operation.

Step 1

Configure producer:

    Properties props = new Properties();
    
    props.put(
    ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
    "localhost:9092"
    );
    
    props.put(
    ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
    StringSerializer.class.getName()
    );
    
    props.put(
    ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
    StringSerializer.class.getName()
    );
    
    props.put(
    ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG,
    true
    );
    
    props.put(
    ProducerConfig.TRANSACTIONAL_ID_CONFIG,
    "payment-service"
    );
    
    KafkaProducer<String,String> producer =
    new KafkaProducer<>(props);

What do these mean?
    
    enable.idempotence=true
    
    Prevents duplicates caused by retries.
    
    transactional.id=payment-service
    
    Enables Kafka transactions.

Step 2
    
    Initialize transaction once:
    
    producer.initTransactions();
    
    Usually during application startup.

Step 3

Consumer
    
    props.put(
    ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,
    false
    );

We must control offset commits manually.

Main Logic
    
    while(true) {
    
        ConsumerRecords<String,String> records =
                consumer.poll(Duration.ofSeconds(1));
    
        producer.beginTransaction();
    
        try {
    
            for(ConsumerRecord<String,String> record : records) {
    
                String paymentEvent =
                    "PAYMENT_CREATED:" + record.value();
    
                producer.send(
                    new ProducerRecord<>(
                        "payments-topic",
                        paymentEvent
                    )
                );
            }
    
            producer.sendOffsetsToTransaction(
                getOffsets(records),
                consumer.groupMetadata()
            );
    
            producer.commitTransaction();
    
        } catch(Exception e) {
    
            producer.abortTransaction();
        }
    }
What happens internally?

Suppose: Order#100

arrives.
    
    Transaction starts
    producer.beginTransaction();
    
    Kafka creates:
    
    TX-1
    Produce event
    producer.send(...)
    
    Kafka stores:
    
    PAYMENT_CREATED
    
    but marks it:
    
    Uncommitted
    
    Other consumers can't see it yet.
    
    Add offset to transaction
    producer.sendOffsetsToTransaction(...)
    
    Kafka says:
    
    Offset 11 will be committed
    
    but not yet.
    
    Still inside TX-1.
    
    Commit transaction
    producer.commitTransaction();
    
    Now Kafka does BOTH:
    
    PAYMENT_CREATED visible
    Offset 11 committed
    
    together.

Crash Example

Suppose:
    
    beginTransaction()
    ↓
    send(PAYMENT_CREATED)
    ↓
    CRASH ❌
    
    Before:
    
    commitTransaction()
    
    Then:
    
    Kafka aborts TX-1
    
    Result:
    
    PAYMENT_CREATED invisible
    Offset not committed
    
    After restart:
    
    Order#100 read again
    
    Only one final payment event will ever become visible.
    
    What is sendOffsetsToTransaction?
    
    This is the magic.
    
    Normally:
    
    consumer.commitSync();
    
    commits offsets separately.
    
    For Exactly Once:
    
    producer.sendOffsetsToTransaction(...)
    
    puts the offset commit inside the Kafka transaction.
    
    So:
    
    Output Event
    +
    Offset Commit
    
    become one atomic unit.
    
    Simple Diagram
    
    Without EOS:
    
    Read Order
    ↓
    Write Payment Event
    ↓
    CRASH
    ↓
    Offset not committed
    
    Duplicate possible
    
    With EOS:
    
    BEGIN TX
    ↓
    Write Payment Event
    ↓
    Add Offset Commit
    ↓
    COMMIT TX
    
    Kafka guarantees:
    
    Both succeed
    OR
    Both fail
    In Spring Boot
    
    You rarely write beginTransaction() manually.

Configuration:
    
    spring.kafka.producer.transaction-id-prefix=tx-
    spring.kafka.producer.properties.enable.idempotence=true
    spring.kafka.consumer.properties.isolation.level=read_committed
    
    Then Spring Kafka can manage transactions for you.
    
    For interviews, the one sentence to remember is:
    
    Exactly Once is achieved by writing output records and committing consumer offsets in the same Kafka transaction, using an idempotent producer and transactional APIs.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


Min ISR

Very important.

Configuration:

    min.insync.replicas=2

Suppose:

    RF = 3
    ISR = [B1, B2]

Producer:

    acks=all
    Works.

Now:

    ISR = [B1]
    Kafka rejects writes.

Why?

    Because minimum safe replicas not available.