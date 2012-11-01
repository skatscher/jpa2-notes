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

Filtering results using the WHERE clause:

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
```java
    em.createQuery(queryString);

Translated queries are often cached. To exploit the caching, use parametrized queries. If using simply concatenated queries, they will be have to be translated again each time.

When concatentaing queries from usen input, be aware that users could alter the queries maliciously.

Use parametrized queries to avoid such attacks. Here, the quotes used in the parameter are escaped and the whole parameter is treated as a string - the malicious SQL will not be executed.

In general, static named queries are preferrable for queries that are exacted often.

### Named Queries
```java
    @NamedQuery(name="findSalaryForNameAndDepartment", query="SELECT e.salary FROM employee e WHERE e.department.name = :deptName AND e.name = :empName")

Named queries are typically annotated on the entitiy class. Here, string concatenation for formatting is acceptable because of it's low and one-time cost. The name of the query msut be unique across the complete PU. So a common practice is to prefix the queries with the name of the entity: `name=Employee.findSalaryByName`

For multiple queries, use the `@NamedQueries` annnotation:

```java
    @NamedQueries({@NamedQuery(...), @NamedQuery(...)})

    em.createNamedQuery("Employee.findByName", Employee.class).setParameter("name", name);

### Parameter types

Parameters are set using the `setParameter(name/number, value)`. `Date` and `Calendar` types require a third method parameter that specifies the type (`java.sql.Date` or `.Datetime` or `.Timesamp`).

Entity types amy be used as parameters as well. When translating to SQL, the PK columns of the entities are used.

    setParameter("start", currentDate, TemporalType.DATE)

The parameter may be used several times in the query, but needs to be set only once. 

### Executing queries

3 ways to execute a query:

* getSingleResult
* getResultList
* executeUpdate - for buld updates, no result is returned

If the query executed by `getResultList` dies not find results, the list is empty. The return type is specified as `List` to support sorting (using `ORDER BY`, queries are unordered by default).

If the query executed by `getSingleResult` finds no result, it throws a `NoResultExcepion`.
If multiple results are available, it wil throw a `NonUniqueResultException`.

These exceptions, unlike other exceptions thrown by the EM will not cause the TX to rollback.

Both `get*` queries may specifiy locking for the affected rows using the `setLockMode` method (more in ch. 11)

`Query` and `TypedQuery` objects may be reused as long as the same PC is active. For TX-scoped EM, the lifetime of a `Query` is limited to the TX. Other EM types may use the Query isntances until the EM is closed or removed.



 


 




 
 








