# Syetem Design
Reference: System Designer's Interview, An insider's guide by Alex Xu

### Single Server Setup
Considering a scenario for a system with 1 server
![single-server-architecture](assets/single-server-architecture.png)
- Users access websites through domain names, such as api.mysite.com. Usually, the Domain Name System (DNS) is a paid service provided by 3rd parties and not hosted by our servers.
- Internet Protocol (IP) address is returned to the browser or mobile app. In the example, IP address 15.125.23.214 is returned.
- Once the IP address is obtained, Hypertext Transfer Protocol (HTTP) [[1]](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) requests are sent directly to your web server.
- The web server returns HTML pages or JSON response for rendering.


Traffic Sources here can be the follows:
1. Web Application: It uses a combination of server-side languages (Java, Python, etc.) to handle business logic, storage, etc., and client-side languages (HTML and JavaScript) for presentation. 
2. Mobile Application: <u>HTTP protocol is the communication protocol between the mobile app and the web server.</u>  JavaScript Object Notation (JSON) is commonly used API response format to transfer data due to its simplicity. An example of the API response in JSON format is shown below

### Vertical scaling vs horizontal scaling
| Vertical Scaling | Horizontal Scaling |
|------------------|--------------------|
|Vertical scaling referred to as “scale up”, means the process of adding more power (CPU, RAM, etc.) to your servers.| Horizontal scaling, referred to as “scale-out”, allows you to scale by adding more servers into your pool of resources.|
|When traffic is low, vertical scaling is a great option, the simplicity of vertical scaling is its main advantage | Horizontal scaling is more desirable for large scale applications due to the limitations of vertical scaling. |

**Limitation of Vertical Scaling**
- Vertical scaling has a hard limit. It is impossible to add unlimited CPU and memory to a single server.
- Vertical scaling does not have failover and redundancy. If one server goes down, the website/app goes down with it completely.


## Components and Key Terms

### Database
With the growth of the user base, one server is not enough, we need multiple servers:
- one for web/mobile traffic
- other for database

Separating web/mobile traffic (web tier) and database (data tier) servers allows them to be scaled independently.

**Which Database to use?**
- Traditional Database(RDBMS)
  - Relational databases represent and store data in tables and rows.
  - You can perform join operations using SQL across different database tables.
  - The most popular ones are MySQL, Oracle database, PostgreSQL, etc. 
- Non-Relational Database
  - These databases are grouped into four categories: 
    - key-value stores
    - graph stores
    - column stores
    - document stores
  - Popular ones are CouchDB, Neo4j, Cassandra, HBase, Amazon DynamoDB, etc.

Non-relational databases might be the right choice if:
- Your application requires super-low latency.
- Your data are unstructured, or you do not have any relational data.
- You only need to serialize and deserialize data (JSON, XML, YAML, etc.). 
- You need to store a massive amount of data. 

```Latency: The delay before a transfer of data begins following an instruction for its transfer```


### Load balancer
If many users access the web server simultaneously and it reaches the web server’s load limit, users generally experience slower response or fail to connect to the server. A load balancer is the best technique to address these problems.

A load balancer evenly distributes incoming traffic among web servers that are defined in a load-balanced set.

![load-balancer](assets/load-balancer.png)

With this setup, web servers are unreachable directly by clients anymore. For better security, private IPs are used for communication between servers. *The load balancer communicates with web servers through private IPs.*


```A private IP is an IP address reachable only between servers in the same network; however, it is unreachable over the internet.```

Through the help of load balancer we can solve the issue of *no failover* and improved the availability of the web tier. This was achieved as follows:
- If server 1 goes offline, all the traffic will be routed to server 2. This prevents the website from going offline. We will also add a new healthy web server to the server pool to balance the load.
- If the website traffic grows rapidly, and two servers are not enough to handle the traffic, the load balancer can handle this problem gracefully. You only need to add more servers to the web server pool, and the load balancer automatically starts to send requests to them.

### Data Replication
The current design has one database, so it does not support failover and redundancy. Database replication is a common technique to address those problems.

Database replication can be used in many database management systems, usually with a master/slave relationship between the original (master) and the copies (slaves).
- A master database generally only supports write operations. 
- A slave database gets copies of the data from the master database and only supports read operations.
- All the data-modifying commands like insert, delete, or update must be sent to the master database.
- Most applications require a much higher ratio of reads to writes; thus, the number of slave databases in a system is usually larger than the number of master databases.

![data-replication](assets/data-replication.png)


Advantages of database replication:
- Better performance: 
  - All writes and updates happen in master nodes whereas, read operations are distributed across slave nodes. 
  - This model improves performance because it allows more queries to be processed in parallel. 
- Reliability:
	- If one of your database servers is destroyed by a natural disaster, such as a typhoon or an earthquake, data is still preserved. You do not need to worry about data loss because data is replicated across multiple locations.
- High availability:
  - By replicating data across different locations, your website remains in operation even if a database is offline as you can access data stored in another database server.

<br>

```
Q. What if one of the databases goes offline?

* Case #1: Only one slave database is available and it goes offline.
- Read operations will be directed to the master database temporarily. As soon as the issue is found, a new slave database will replace the old one.

* Case #2: Multiple slave databases are available.
- Read operations are redirected to other healthy slave databases. A new database server will replace the old one.

* Case #3: The master database goes offline
- A slave database will be promoted to be the new master
- All the database operations will be temporarily executed on the new master database.
- A new slave database will replace the old one for data replication immediately.
```

#### NOTES:
Complexities of promoting a new Master:
  - In production systems, promoting a new master is more complicated as the data in a slave database might not be up to date. 
  - The missing data needs to be updated by running data recovery scripts.
  - Although some other replication methods like [multi-masters](https://en.wikipedia.org/wiki/Multi-master_replication) and circular replication could help, those setups are more complicated

![single-server-architecture](assets/load-balancer-data-replication.png)
- A user gets the IP address of the load balancer from DNS. 
- A user connects the load balancer with this IP address. 
- The HTTP request is routed to either Server 1 or Server 2. 
- A web server reads user data from a slave database.
- A web server routes any data-modifying operations to the master database. This includes write, update, and delete operations.
