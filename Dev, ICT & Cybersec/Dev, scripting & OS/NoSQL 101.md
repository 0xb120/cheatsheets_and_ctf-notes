---
aliases:
  - NoSQL
---

NoSQL databases store and retrieve data in a format other than traditional SQL relational tables. They are designed to handle large volumes of unstructured or semi-structured data. As such they typically have fewer relational constraints and consistency checks than SQL.

Differently from SQL, NoSQL applications **use a wide range of query languages instead of a universal standard**.

## NoSQL database models

Some common types of NoSQL databases include [^1] :
- **Document stores**: data are stored in flexible, semi-structured documents. Tipically used formats are JSON, BSON and XML and are queried using API or query language. Examples are MongoDB and Couchbase
- **Key-value stores**: data are stored in a key-value format. Each data field is associated with a unique key string. Values are retrieved based on the unique key. Examples include Redis and Amazon DynamoDB
- **Wide-columns stores**: organize related data into flexible column families rather than traditional rows. Examples include Apache Cassandra and Apache HBase
- **Graph databases**: These use nodes to store data entities, and edges to store relationships between entities. Examples include Neo4j and Amazon Neptune

[^1]: [NoSQLi: A Complete Guide to Exploiting Advanced NoSQL Injection Vulnerabilities](../../Readwise/Articles/blackbird-eu%20-%20NoSQLi%20A%20Complete%20Guide%20to%20Exploiting%20Advanced%20NoSQL%20Injection%20Vulnerabilities.md)
