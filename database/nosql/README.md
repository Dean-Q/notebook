---
description: Not only SQL 非关系型数据库
---

# NoSQL

## Characteristics

* **Flexible data model**: support for multiple data models, such as key-value pairs, docs, column, graph, and better handling semi-structured data and unstructured data(such as docs, logs, images, videos)
* **Weak consistency**: For high availability and scalability, Nosql uses eventual consistency or flexible transaction, which allows data to have certain time difference between different nodes
* **Decentration**: emphasis on scalability and fault tolerance in distribute environment and reduce the risk of single point of failure
* **High performanc**e: performs well in large-scale distribute system, can quickly process large amounts of data and complex queries

## <mark style="color:green;">Pros</mark> and <mark style="color:red;">Cons</mark>

* <mark style="color:green;">Flexible data model</mark>
* <mark style="color:green;">High performance</mark>
* <mark style="color:green;">High scalability</mark>: Horizontally scaling, more node can be added to increase the processing power and capacity of system
* <mark style="color:green;">Distributed architecture</mark>
* <mark style="color:green;">Low cost:</mark> Open source Nosql is free
* <mark style="color:red;">Weak consistency</mark>: for some scenarios, data inconsistency occurs
* <mark style="color:red;">Query capacity limitation</mark>: less suitable for complex query and transaction processing, Nosql lacks complex query operations and aggregation functions, and usually only support basic query
* <mark style="color:red;">Lack of standardization</mark>: no unified standard among Nosql databases, and each database has its own API and query language, which can lead to increase development and maintenance costs
* <mark style="color:red;">Security and Privacy</mark>: Nosql doesn't have a well-established ecosystem and is inferior to RDBMS in terms of  data security and privacy protection
