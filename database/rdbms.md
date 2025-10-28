---
description: Relational Database Management System 关系型数据库
---

# RDBMS

## Characteristics

* **Table structure**: each table consists of two dimension, row(record) and column(field)
* **Data integrity**: support data integrity convention, such as primary key, foreign key and unique convention
* **Transaction support**: support transaction processing, ensure ACID(Atomicity, Consistency, Isolation, Durability) features of data
* **Relationship between data**: support the establishment of relationship between data through mechanisms such as  foreign key, such as one-to-one, one-to-many, many-to-many relationships
* **Query Language**: SQL provides rich syntax and functionality to support many complex operations

## <mark style="color:green;">Pros</mark> vs <mark style="color:red;">Cons</mark>

* <mark style="color:green;">Data consistency and integrity</mark>: support ACID principle
* <mark style="color:green;">Mature and stable</mark>: after long-term development and practice, it has mature technology and stable performance
* <mark style="color:green;">Powerful query language</mark>: structural query language(SQL) provides powerful, supports complex queries and data operations
* <mark style="color:green;">Widely used</mark>: RDBMS is widely used in enterprise and traditional data processing scenarios , have rich tools and ecosystems
* <mark style="color:green;">Standardization</mark>: follow a unified data model and specification
* <mark style="color:green;">Security</mark>: provide data access control and authority management
* <mark style="color:red;">Limited scalability</mark>: In the case of large-scale data processing and high concurrent access, vertical scaling is expensive
* <mark style="color:red;">Data model is not flexible</mark>:  the data model is relatively fixed which is difficult to adopt the change of data structure and the storage requirement of unstructured data
* <mark style="color:red;">Performance problem</mark>: In the case of concurrent access massive data, the performance is not as good as NoSQL
* <mark style="color:red;">Complexity</mark>:  design and management are complex
* <mark style="color:red;">High cost</mark>: higher costs are usually required for licensing, hardware and maintenance
* <mark style="color:red;">Not suitable for large-scale distributed system</mark>:  the architecture is not adoptable to the needs of distributed environment

