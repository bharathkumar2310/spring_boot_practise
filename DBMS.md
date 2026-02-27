

**_DISK :**_ 

    ---> A disk is a hardware storage device which stores data permanenty
    --->It is where your:
    
            Operating System
            Applications
            Databases
            Files
            Photos
            Videos
    are stored.
    
    --->When you turn off your laptop, disk keeps the data safe.


Types of Disk :

    ---> HDD(Old)
    ---> SSD(Modern)


C: drive
D: drive
these are the partitions of disk

If we add a new ssd we can create a partition drive for it



**_RAM :_**

    ---> Temporary working memory used by the CPU to run programs.
    --->It stores:
    
            Running applications
            Open browser tabs
            Active database queries
            Variables in your program
    
    When power goes off → RAM is erased.
    
    ---> RAM is a physical chip installed on the motherboard looks like a long green stick
    ---> You normally cannot see RAM stored data directly.


| RAM                                 | Disk                      |
| ----------------------------------- | ------------------------- |
| Temporary                           | Permanent                 |
| Very fast                           | Slower                    |
| Volatile (loses data without power) | Non-volatile              |
| Used for running programs           | Used for storing programs |
| Small size (8GB, 16GB)              | Large size (256GB, 1TB)   |



**_FILE :**_ 

    ---> File is a named collection of data stored in a storage device

**_DATABASE :_**

    ---> An organized collection of data stored in a structured way so it can be easily accessed, managed, and updated.
    ---> It is data stored in one or more structured files, managed by a DBMS.


A Relational Database (RDBMS) stores data in:

        Tables
        Rows
        Columns
        With relationships between tables

Non-Relational Database?

        Also called NoSQL database.
        It does NOT strictly store data in tables with fixed schema.
It can store data as:

        Documents (JSON-like)
        Key-value pairs
        Graphs
        Wide-column stores

A DBMS (Database Management System) is:

        Software that manages databases.

That’s the core idea.

🔵 What Does “Manages” Mean?

A DBMS:

        ✔ Stores data on disk
        ✔ Organizes data into tables
        ✔ Allows querying (SELECT, INSERT, UPDATE)
        ✔ Handles multiple users
        ✔ Ensures security
        ✔ Maintains consistency
        ✔ Recovers data after crash

Without DBMS, database is just raw files.

🔵 Real Examples of DBMS

        MySQL
        PostgreSQL
        Microsoft SQL Server
        Oracle Database
        MongoDB

These are programs installed on your system or server.

**_OS :**_

    An Operating System is the main software that controls your entire computer.
    
    It sits between:
    
        🧠 Hardware (CPU, RAM, Disk, Keyboard, etc.)
        👨‍💻 Applications (Chrome, VS Code, Database, etc.)


What happens when we create a database?

        CREATE DATABASE college;

The DBMS:

        Parses the SQL
        Checks permissions
        Decides where to create files

The DBMS tells the OS:

    👉 “Create new storage files for this database.”
        Create a data file → .mdf
        Create a log file → .ldf

At OS level:

    New files are created on disk with some default storage space, later if needed it will allocate more


1️⃣ Data File (.mdf)

Stores:

    Tables
    Rows
    Indexes
    Metadata
    System pages

This is the actual database content.


2️⃣ Log File (.ldf)

Stores:

    Every change made to the database
    Before/after values
    Transaction begin/commit info

This is used for:

    Crash recovery
    Rollback
    Ensuring ACID properties




============================================================================================================



Whenwver we start a DBMS , OS allocates it some RAM Storage to DBMS 

DBMS splits and divides this memory 


1. 1️⃣ Buffer Pool (Largest Part)

Contains:

        Data pages
        Index pages
        Dirty pages
        Cached table metadata

Purpose:
👉 Avoid disk reads
👉 Improve performance

This is usually the biggest chunk of memory (often 60–80% of total DB memory).

2️⃣ redo Log Buffer

3. Undo Log Buffer


2. When we create a DB , disk memory is divided into this 

1️⃣ Data files ----> seperate for each db
2️⃣ Redo log files ----> shared per dbms
3️⃣ Undo log files ----> shared per dbms
4️⃣ System catalog (metadata)
5️⃣ Temporary files
6️⃣ Configuration files
7️⃣ Control files



