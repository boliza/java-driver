# Object Mapper

Version 2.1 of the driver introduces a simple object mapper, which avoids most of the boilerplate when converting your domain classes to and from query results.

The object mapper handles basic CRUD operations in Cassandra tables containing UDTs, collections and all native CQL types.

This documentation is organized in subsections defining all functionalities of the object mapper.

The examples in this section are based on the following table structure:

```java
CREATE TABLE user (user_id uuid PRIMARY KEY, name text);
CREATE TYPE address (street text, zip_code int);
CREATE TABLE userAd (user_id uuid PRIMARY KEY, name text, ad address);
```