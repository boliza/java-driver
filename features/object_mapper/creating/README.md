## Definition of mapped classes

The object mapper allows to define mapped classes by using annotations.
 
### Creating a table entity
 
A table entity is generally meant to match the definition of an existing table in the *Cassandra* server.

Classes that will be mapped to tables are annotated via `@Table`:

```java
@Table(keyspace = "ks", name = "users",
       readConsistency = "QUORUM",
       writeConsistency = "QUORUM",
       caseSensitiveKeyspace = false,
       caseSensitiveTable = false)
public static class User {
    @PartitionKey
    @Column(name = "user_id")
    private UUID userId;
    private String name;
    // ... constructors / getters / setters
}
```

--------------

`@Table` allows to use options for each entity :

- `keyspace` : the keyspace for the table.
- `name` : the name of the table in the database.
- `caseSensitiveKeyspace` : whether the keyspace name is [case sensitive].
- `caseSensitiveTable` : whether the table name is [case sensitive].
- `readConsistency` : the [consistency level] that will be used on each get operation in the mapper. (if unspecified, it defaults to the cluster-wide setting)
- `writeConsistency` : the [consistency level] that will be used on each save, or delete operation in the mapper. (if unspecified, it defaults to the cluster-wide setting)

A class annotated with `@Table` must provide a default constructor.

[case sensitive]:http://docs.datastax.com/en/cql/3.0/cql/cql_reference/ucase-lcase_r.html
[consistency level]:http://docs.datastax.com/en/drivers/java/2.1/com/datastax/driver/core/ConsistencyLevel.html

--------------

`@Column` can be used on each field of a mapped object to specify the column name in the database. `caseSensitive` can be used for [case sensitive] columns. If `@Column` is not specified, the name of the field will be used.

--------------
 
`@PartitionKey` and `@ClusteringColumn` are used to indicate the [partition key][pks] and [clustering columns][pks]. In the case of a composite partition key or multiple clustering columns, the integer value indicates the position of the column in the key:
 
```java
@PartitionKey(0)
private String countryCode;
@PartitionKey(1)
private String areaCode;
```

[pks]:http://www.planetcassandra.org/blog/primary-keys-in-cql/

--------------

`@Enumerated` is used to reference a field typed as a Java enumeration into the database. Since *Cassandra* does not support enumerations, the object mapper defines to ways to persist a `enum` value :

- `EnumType.ORDINAL` : the value is persisted as a C\*'s `int` and its value is its position in the `enum`.
- `EnumType.STRING` : the value is persisted as a C\*'s `text` and its value is its name in the `enum`.

According to the chosen type, the matching database column must be the same type.

```java
@Enumerated(EnumType.ORDINAL)
MyEnum enum;
```

--------------

`@Computed` can be used on fields that are the result of a computation on the *Cassandra* side. Typically a function call. Native function in *Cassandra* like `writetime()` or [User Defined Functions] are supported.
 
```java
@Computed(formula = "ttl(name)")
Integer ttl;
```

Return types of function must match the type of the field, otherwise an exception will be thrown.

[User Defined Functions]:http://www.planetcassandra.org/blog/user-defined-functions-in-cassandra-3-0/

--------------

`@Transient` can be used to prevent a field from being mapped.

Each field in a class annotated with `@Table` (except ones annotated `@Transient`) must have setters and getters with the same name as the field name.


### Mapping User Types

[User Defined Types] are considered in the mapper as plain old java objects annotated with `@UDT` :

```java
@UDT(keyspace = "ks", name = "address")
class Address {
    private String street;
    @Field(name = "zip_code")
    private int zipCode;
    // ... constructors / getters / setters
}
```

As for `@Table` entities, some options are available with `@UDT` :

- `keyspace` : the keyspace for the UDT.
- `name` : the name of the UDT in the database.
- `caseSensitiveKeyspace` : whether the keyspace name is case sensitive.
- `caseSensitiveType` : whether the UDT name is case sensitive.

--------------

Fields in a mapped UDT class that don't have the same name in database, can be annotated with `@Field` :

```java
@Field(name = "zip_code")
private int zipCode;
```

The `@UDT` annotated class must provide getters and setters for all fields.

Then, to reference a UDT in a mapped class, it should just be put as a usual field, or can be included in a collection :

```java
public static class UserAd {
    @PartitionKey
    @Column(name = "user_id")
    private UUID userId;
    private String name;
    private Address ad;
}
```

[User Defined Types]:http://docs.datastax.com/en/developer/java-driver/2.1/java-driver/reference/userDefinedTypes.html

### Mapping collections

Java collections will be automatically mapped into corresponding *Cassandra* types, collections can contain, as in *Cassandra*, all native types and all [user types](#mapping-user-types) previously defined is the database.
Frozen collections can be mapped with `@Frozen`, frozen collection keys with `@FrozenKey` and frozen collection values by using `@FrozenValues` :

```java
private List<String> stringList; // Will be mapped as a 'list<text>'
@Frozen 
private List<String> frozenStringList; // Will be mapped as a 'frozen<list<text>>'
@FrozenKey
@FrozenValue
private Map<Address, List<String>> frozenKeyValueMap // Will be mapper as 'map<frozen<address>, frozen<list<text>>>'
@FrozenValue
private Map<String, List<Address>> frozenValueMap // Will be mapper as 'map<text, frozen<list<address>>>'
```