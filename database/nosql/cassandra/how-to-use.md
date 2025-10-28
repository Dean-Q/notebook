---
description: Some terms, read and write logic, simple cql queries
---

# How to use

## Basic Terms

* **Node**: The basic building block of cassandra, a node can be a physical server or a virtual server.
* **Datacenter**:  A group of related nodes that are configured together within a cluster for replication and workload segregation purposes. Not necessarily a separate location or physical data center. Datacenter names are case sensitive and cannot be changed.
* **Keyspace**: A namespace container that defines how data is replicated on nodes in each datacenter, the outermost container for data in Cassandra, it's familiar to traditional database, used to organize and manage tables, each keyspace has its own replica strategy and related tables.
* **Column Family**: A container for rows, similar to the table in a relational system. Called a table in CQL 3.
* **Partition Key**: A unique identifier for each row that is used to locate the location of the data in the cluster.
* **Replication Factor**: It’s the number of machines in the cluster that will receive copies of the same data.
* **Consistency Level**: A setting in which a read or write must enter several replicas before it is considered successful
* **Token**: Location identifier used to allocate data in Cassandra. Each node is responsible for a range of tokens, which ensures that the data is evenly distributed throughout the cluster

## Keyspace

### Create

```sql
CREATE KEYSPACE IF NOT EXISTS school 
   WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'}  
   AND durable_writes = true;

# or

CREATE KEYSPACE school
   WITH replication = {'class': 'NetworkTopologyStrategy', 'DC1' : 1, 'DC2' : 3}
   AND durable_writes = false;
```

### Read

```sql
# show all keyspaces
DESC KEYSPACES;
# select one specified keyspace
USE school;
# read the detailed info of the keyspace
DESC school;
```

### Set

```sql
ALTER KEYSPACE school
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 4};
    
# or

ALTER KEYSPACE IF EXISTS school
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 4};
```

### Delete

```sql
DROP KEYSPACE school;

# or

DROP KEYSPACE IF EXISTS school;
```

### Replication (Commands parameter)

The `replication` property is mandatory and must contain the `'class'` sub-option that defines the desired replication strategy class. The rest of the sub-options depend on which replication strategy is used

SimpleStrategy: defines a replication factor for data to be spread across the entire cluster, it does not respect datacenter layouts and can lead to wildly varying query latency

