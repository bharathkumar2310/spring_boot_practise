**_ACID :_**

        ---> Atomicity, Consistency, Isolation, Durability
        ---> ACID properties are a set of 4 properties that guarantees DB transactions are processed reliably, consistently
             and maintains data integrity even during failures

        --->A transaction is a group of operations treated as a single logical unit of work.
             Either all operations succeed or none of them are applied.


----> Most of the DB implement ACID principles by themselves for reliable transaction
----> SpringBoot acts as transactional manager telling DB when to start a transaction when to roll back when to commit



1. Atomicity : 

        ---> Ensures every operation within a transaction is successfull. If any operation fails the entire transaction(all operations are rolledback)
        --->There should never be a partially executed transaction.

Example :

```
      BEGIN;
      
      UPDATE accounts SET balance = balance - 100 WHERE id = 1;
      UPDATE accounts SET balance = balance + 100 WHERE id = 2;
      
      COMMIT;
```



Step 1️⃣ Transaction Starts

      DB assigns a Transaction ID (T1).

Step 2️⃣ First UPDATE Happens

      First Buffer checks whether it has the data else it will fetch it from disk


      👉 DB writes OLD value into Undo Log(Buffer Undo Log)--> RAM as well as redo Logd(RAM)
       In undo logs it stores the previous value , in redo logs it stores the update value
       Then Buffer Pool updates value to 900.

Step 3 : Same way 2nd update happens

      First Buffer checks whether it has the data else it will fetch it from disk


      👉 DB writes OLD value into Undo Log(Buffer Undo Log)--> RAM
       In undo logs it stores the previous value , in redo logs it stores the update value
       Then Buffer Pool updates value to 600.

Suppose error happens before commit

DB uses Undo Log:

      Restore Account B → 500
      Restore Account A → 1000

Undo is applied in reverse order.

After rollback:

      System is exactly like before transaction.

There is no work on disk itself

Atomicity preserved ✅



During commit

1. The redo  logs form buffer is flushed to redo  logs in disk
2. undo logs can be flushed during checkpoint or any other time depending on dbms
3. Redolog flush is required for  Write-Ahead Logging (WAL).
4. The status of transaction is changed to committed


The value change in disk data happens periodically
DBMS periodically runs a background process called:
👉 Checkpoint
which will flush buffer dat to disk data



**_2. CONSISTENCY :_**

---> Consistency ensures that a transaction changes DB from one valid state to another valid state

After a transaction, the database rules are never violated.

Examples of rules include:

            Foreign keys
            Primary keys
            Unique constraints
            Check constraints
            Application-level rules

      --->Consistency is guaranteed only if the transaction itself is correct:
      
      --->ACID ensures atomicity + isolation + durability
      --->These properties help transactions not leave partial updates, which could violate consistency.
      
So Consistency is a logical property, enforced by:
      
            Database constraints
            Correct transaction code



**_3. ISOLATION :_** 

      ---> Isolation ensures that concurrent transactions do not interfer with each other
      ---> Each transaction is exceuted as if it is the only transaction
      
      
      --->Intermediate results of a transaction are not visible to other transactions until it commits.
      ---->This prevents dirty reads, non-repeatable reads, and phantom reads.


Without isolation:

      Two transactions could update the same data at the same time → data corruption
      One transaction could see partial updates from another → inconsistent reads

DBMS acheives this by 

1. Row level locks 
2. Table level locks

Row-level locks → Only one transaction can modify a row at a time
Table-level locks → Prevent access to entire table during certain operations


3. Multi-Version Concurrency Control

         Instead of locking, some DBs (like PostgreSQL, MySQL InnoDB) use versioned rows
         Each transaction sees a snapshot of data at the time it started
         Updates create new versions; old versions are kept in undo logs
         While writing it sees if the old version matches current version if yes do update else roll back




**_4. DURABILITY :_**


      ---> Durability guarantees that once a transaction is committed, its changes will survive permanently, 
           even in case of system crashes, power failures, or hardware errors.


In other words:

      If you see COMMIT succeed → DB promises that the change won’t be lost.
      The data may still be in RAM (buffer pool), but mechanisms ensure recovery in case of crash.

2️⃣ How DBMS Achieves Durability

Durability is primarily implemented using Write-Ahead Logging (WAL) and redo logs.

🔹 Step 1 — Redo Log

      Every change in a transaction is first written to the redo log in RAM
      At commit: redo log is flushed to disk
      WAL Rule: Redo log must reach disk before marking transaction committed
               :The DB must write the log entry to disk before it writes the actual data page to disk.

This ensures durability even if data pages in the buffer pool are not yet written to disk.

🔹 Step 2 — Buffer Pool & Data Pages

      Updated pages in buffer pool are dirty pages
      Flushing to data file on disk happens asynchronously:
      During checkpoint
      Eviction
      Shutdown

Even if a crash occurs before page flush, redo log allows recovery.

🔹 Step 3 — Crash Recovery

If crash occurs after commit:

      DBMS reads redo logs on restart(from disk redo logs it changes the disk data)
      Reapplies all committed changes to the data files
      Data file is now consistent with the committed transactions
      
      Undo logs are used only for uncommitted transactions (rollback)
      Redo logs are used for committed transactions (durability)