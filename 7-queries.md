Using Queries
============

## Overview

JPQL is msotly similar to pure SQL - the first difference is that instead of selecting a table, we selct an entity:

    SELECT e FROM Employee e

#### . operator
We can navigate entity relationshis using the dot operator

    SELECT e.name FROM Employee e

We dont have to list an entity to navigate to it:
    
    SELECT e.department FROM Employee e

### Projections
We can retrieve a projection, using only portions of found data:
    
    SELECT e.name, e.salary FROM Employee e

//TODO : how do we work with projected results - how are they returned from a query?

### Filtering results 
using the WHERE clause:

    SELECT e FROM Employee e WHERE e.name = "John Smith"

#### Joins

Collection return types are illegal in JPQL. To return the collection we msut use Joins (p. 181).
I have foudn collection queries to work, despite the warning - see _BasicQueriesTest.testCollectionQuery_ in the _jpa2-ch7-queries_ project.

Joins may be implicit and explicit. Explicit joins are used for eager prefetching or for different types of joins (more in ch.8)

#### Aggregate Queries

These five aggregate function are supported: **AVG, COUNT, MIN, MAX, SUM**. The results may be grouped using the **GROUP BY** and filtered with the **HAVING** clause.

    SELECT COUNT(e), MAX(e.salary), AVG(e.salary) FROM Department d JOIN d.employees e
    GROUP BY d HAVING COUNT(e) > 3

#### Query Params

There are 2 types of parameter binding syntax - positional and named:

    SELECT e FROM Employee e WHERE e.department = ?1 AND e.salary > ?2

    SELECT e FROM Employee e WHERE e.department = :dept AND e.salary > :base

## Defining Queries

**Query** & **TypedQuery**. `TypeQuery` extends `Query`. Queries are created using the 4 factory methods of the EM. Thsi chapter handles only the SPQL queries. SQL -> ch.11, criteria -> ch.9

2 Approaches to deifne an SPQL query: 

* dynamically specified at runtime
* configured in teh PU metadata (annotation or XML) and referenced by name.

Dynamic queries are simple strings, named queries are staic and more efficient as they are precompiled once.

### Dynamic Queries


    em.createQuery(queryString);

Translated queries are often cached. To exploit the caching, use parametrized queries. If using simply concatenated queries, they will be have to be translated again each time.

When concatentaing queries from usen input, be aware that users could alter the queries maliciously.

Use parametrized queries to avoid such attacks. Here, the quotes used in the parameter are escaped and the whole parameter is treated as a string - the malicious SQL will not be executed.

In general, static named queries are preferrable for queries that are exacted often.

### Named Queries

```java
    @NamedQuery(name="findSalaryForNameAndDepartment", query="SELECT e.salary FROM employee e WHERE e.department.name = :deptName AND e.name = :empName")
```

Named queries are typically annotated on the entitiy class. Here, string concatenation for formatting is acceptable because of it's low and one-time cost. The name of the query msut be unique across the complete PU. So a common practice is to prefix the queries with the name of the entity: `name=Employee.findSalaryByName`

For multiple queries, use the `@NamedQueries` annnotation:

```java
    @NamedQueries({@NamedQuery(...), @NamedQuery(...)})

    em.createNamedQuery("Employee.findByName", Employee.class).setParameter("name", name);
```

### Parameter types

Parameters are set using the `setParameter(name/number, value)`. `Date` and `Calendar` types require a third method parameter that specifies the type (`java.sql.Date` or `.Datetime` or `.Timesamp`).

Entity types amy be used as parameters as well. When translating to SQL, the PK columns of the entities are used.

    setParameter("start", currentDate, TemporalType.DATE)

The parameter may be used several times in the query, but needs to be set only once. 

### Executing queries

3 ways to execute a query:

* getSingleResult
* getResultList
* executeUpdate - for bulk updates, no result is returned

If the query executed by `getResultList` dies not find results, the list is empty. The return type is specified as `List` to support sorting (using `ORDER BY`, queries are unordered by default).

If the query executed by `getSingleResult` finds no result, it throws a `NoResultExcepion`.
If multiple results are available, it wil throw a `NonUniqueResultException`.

These exceptions, unlike other exceptions thrown by the EM will not cause the TX to rollback.

Both `get*` queries may specifiy locking for the affected rows using the `setLockMode` method (more in ch. 11)

`Query` and `TypedQuery` objects may be reused as long as the same PC is active. For TX-scoped EM, the lifetime of a `Query` is limited to the TX. Other EM types may use the Query isntances until the EM is closed or removed.

## Working with query results

The entities returned from a query are managed. The only exception is when using an EM outside any TX - then the entities are retuned in detached state.

### Untyped results

If the query is not parametrized by the return type, `getResultList` will return an untyped `List` and `getSingleResult` will return an `Object`.

### Optimizing read-only queries

Using TX-scoped EM outside of TX ffor read-only wueries may be more efficient. For managed antites, a copy is created to compare the initial state with the one on TX commit. In cases where the entities are detached immediately, the provider may be able to optimize this overhead away. This trick does not work for app-managed or extended EMs, as their PC is not discarded on TX commit. To disable transactions, use `TransactionAttribute.NOT_SUPPORTED`

### Special result types

When returning multiple results (projection and aggregate queries), the return type will be a `List` of `Object[]`. This can be used in **Constructor expressions** to create custom objects with the JPQL `NEW` operator. -> see tests

### Pagination

JPA supports pagination with the `setFirstResult` and `setMaxResults` methods of the `Query`. These values can be accessed via `getFirstResult` and `getMaxResults`.

Do not use pagination for queries that use joins across collection relationships (1-* and *-*) as these queris may return duplicate values. The duplicates make it impossible ot use a position.
-> example?

### Queries and uncommited changes

Queries are eecuted directly on the DB, so the provider cannot participate and use the PC. If the PC has not been flushed, the query may return stale data.

This is not a concern when using the `find()` method - it always checks the PC first.

The provider will attempt to mke query results consistent, regardless of the flush state - either flushing the PC before the Query or using the PC to modify the results. Ensuring the integrity is not easy and may not always be desired. 






 


 




 
 








