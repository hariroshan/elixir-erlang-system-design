# Elixir / Erlang System Design

Organising the system design and scaleability knowledge while building Elixir / Erlang systems. Feel free to correct statements as it is meant to serve the community for future reference.

## Things to remember

- Atoms are not garbage collected. Make sure you don't create atoms during runtime as it can crash BEAM. Use `String.to_existing_atom/1` to create atoms from strings as it crashes the process when it encounters a new atom which doesn't exist in the atom table.

- ETS table needs to be managed manually by the process (i.e. creation and deletion of objects) Since it is not garbase collected as well.


- To achieve high performance, Send only the data the process needs rather than sending the whole struct as message.
    ```elixir

    # Example
    data = %Data{value: 12, ...otherFields} # large struct with multiple fields

    # Instead of this 
    GenServer.cast(__MODULE__, {:process_value, data})
    
    # Use this
    GenServer.cast(__MODULE__, {:process_value, data.value})

    ```
## Best Practices

1. Start by writing a module that solves business needs and logics. And a module to handle the storage needs such as writing data in a database / disk.

2. Use the module to build Elixir/Erlang Processes (GenServer, GenStage, Flow etc.). If needed, use ETS table for read / write faster access to minimize database requests. Clean the ETS table when the data is not used.

3. Connect each process to a supervisor. Then to context level supervisor. Then connect all context supervisors to application supervisor.

![](/Images/new-supervisor.png)

### Factors that degrade system performance

1. Disk I/O. Performing frequent disk read or write operations can be slow. It can be avoided / optimized by using ETS table or by using disk with high Disk I/O operation.

2. Network I/O. This can be unavoidable at times. So, having fallbacks incase of failure is advised.


### Using Mnesia for Database:

Erlang has its own database called [Mnesia](https://www.erlang.org/doc/apps/mnesia/mnesia_chap1) ([Docs](https://www.erlang.org/doc/man/mnesia.html)). 

#### Features of Mnesia
- A relational/object hybrid data model that is suitable for telecommunications applications.
- A DBMS query language, Query List Comprehension (QLC) as an add-on library.
- Persistence. Tables can be coherently kept on disc and in the main memory.
- Replication. Tables can be replicated at several nodes.
- Atomic transactions. A series of table manipulation operations can be grouped into a single atomic transaction.
- Location transparency. Programs can be written without knowledge of the actual data location.
- Extremely fast real-time data searches.
- Schema manipulation routines. The DBMS can be reconfigured at runtime without stopping the system.

Use it after reading the docs and knowing the advantages and disadvantages of it. Mnesia has a size limitation that table data can't be more than 2GB. If it exceeds, you need to fragment the table. 

> A concept of table fragmentation has been introduced to cope with large tables. The idea is to split a table into several manageable fragments. Each fragment is implemented as a first class Mnesia table and can be replicated, have indexes, and so on, as any other table. But the tables cannot have local_content or have the snmp connection activated.

It is suitable if you need to store Elixir / Erlang terms in the database and don't want to use SQL as data storage mechanism. It supports both transactions for consistency and dirty operation for inconsistent but faster read/write.

### Using SQL - based databases:

This is the approach the elixir community encourages as it has better consistency, support and reliability than Mnesia. 

### Release
(to be updated soon)


## 1. Single Server

> Suitable for blogs, services with less availability and hosting webpages.


![](/Images/single-server.png)

This architechure can take you long way if you can upgrade to multi-cores and high memory as Elixir/Erlang programs runs faster in multi-core systems.

Staying with single server can be expensive because the cost of a multi-core server will be much higher than having multiple average servers.
But distributed architechure has its own pros and cons.
