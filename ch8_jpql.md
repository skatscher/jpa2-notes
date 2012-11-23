JPQL
====

## Into

### Terminology
Queries fall into 4 categories

* select
* aggergate - variation of select, summarizing the select result
* update
* delete

Aggregate and select queries are sometimes called report queries. THe set of entities and embeddables the queries operate on are called "abstact persistence schema". 

Entities may be named using the `name` attribute of the `@Entity` annotation. The unqualified class name is the default entity name. Smple properties of entities are called "state fields", properties that are relations are called "association fields".

Queries are not case-sensitive except for entity and propery names - they must be written exactly as in teh onject.

## Select queries

```(sql)
SELECT
FROM
[WHERE]
[ORDER BY]
```
Important differnces to SQL - the domain is not the table but the entity.

In `SELECT e FROM Employee e`, the `e` (called alias in SQL) is called **identification variable**. Unlike SQL, the id var. is mandatory. The generated SQL from the above query may look like this : `SELECT id, name, salary, manager_id, dept_id, address_id FROM emp`

### Path expressions

The building blocks of queries. They are used to navigate along relationships or to properties. 

Navigations to a property is called a **state field path**.
Navigation to a singe entity is called  **single-valued association path**
Navigation to a collection of entities is called  **collection-valued association path** 

The navigation is performed using the dot(**.**) operator. The navigation stops at state field paths and collection-valued association paths. 

The keyword `OBJECT` has no implct on the query but may be used as a visual clue to sgnify that an entity is expected. Unfortunately, `OBJECT` is restricted to id vars (though e.department would return an entity, you cannot use `OBJECT` - only `e`) and thus not very useful.

The `DISTINCT` operator is used to exclude duplicates:
`SELECT DISTINCT e.department FROM Employee e`

The result type of a select query may not include collection-valued paths:
`SELECT d.employees FROM Department d` is illegal. This restriction prevent the provider from combining successive DB rows to a single result (and why would that be bad?) -> SO

When a select query returns an embeddable , like in `SELECT e.address FROM Employee e`, it's important to note that the embeddable **will not be managed**. In order to obtain managed embeddables, retrieve the managed embedding entities and navigate from there.

### combining expressions (Projection queries)
### Ctor expressions
### Ingeritance and polymorphism 

Restrivting queries to a particular class of a hierarchy:

`SELECT p FROM Project p WHERE TYPE(p)=DesignProject` -- there are no quotes around the type name, but string params can be used as params.

## FROM clause

Consists from id vars and join clause declarations. Every query must have at least one id var in the fROM clause, corresponding to an Entity. When the id var is not a path but a single entity - e.g. `e` - it is called a **range variable declaration** (confusing, but it comes from the set theory). An optional `AS` keyword max be used. The id must follow standard java naming rules (case insensititve)

### Joins

Joins are queries combining results from ultiple entities. They occur whenever:

* more than one range var decls are listed in the `FROM` and appear in the `SELECT`
* `JOIN` operator is used to extend the entitiy through a pth expression (whaa?)
* a path expr. navigates along an association field to the same or different entity. 
*  `WHERE` conditions compare attributes of different id vars.

Joins may be specified explicitly using the `JOIN` keyword, or implicitly as result of path navigation.

**Inner joins** return entites satisfying all join conditions. Path navigation from on eentity to another is aform of inner join. An **outer join** is a set of all entities satisfying a ll join condtions plus all instances of one of the entities (called the **left** type). Absence of join conditions will produca a cartesian product (all possible combinations of the entities -> M*N results). Cartesian products are rarely used in JPQL. 

 More on the `JOIN` operator wiht examples: 217-222:
#### Inner joins
##### Join op. on collection association fields

`SELECT p FROM Employee e JOIN e.phones p` eqivalent to 
`SELECT p FROM Employee e, Phone p WHERE  e.emp_id = e.id`
 And once more using the old `IN()` operator from EJBQL:
`SELECT DISTINCT p FROM Employee e, IN(e.phones) p`

Of the above example, using the `JOIN` op is recommended.

##### Join op and single valued association fields

`SELECT d FROM Employee e JOIN e.department d` 
// what woudl be the difference to `SELECT d FROM Employee e JOIN Department d`? would hte association be used?
identical query w/o explicit joins: `SELECT e.department FROM Employee e`
identical query with join conditions in the `WHERE` clause: `SELECT d FROM Employee e, Department d WHERE d = e.department`


As we see, there is not much to gain in preferring explicit joins for single valued association fields to regular navigation. Path navigation is equivalent to inner joins. An exception are outer joins:

`SELECT DISTINCT e.department FROM Project p JOIN p.employees e WHERE p.name='Release1' AND e.address.state='CA'` 

Selecting the departments of all employees from California who work on the project Release1.
This query conatins 4 logical joins (and could involve even more tables if relations with join tables are involved). +p219

#### Join consitions in WHERE clause

These are often used if there is no explicit relation between the entities. E.g.: Selecting all departments and their managers (the employees who direct other employees):
`SELECT d, m FROM Department d, Employee m WHERE d=m.department AND m.manager IS NOT EMPTY`

#### Multiple joins

`SELECT DISTINCT p FROM Department d JOIN d.employees e JOIN e.projects p`

#### Map joins

Use `KEY` and `VALUE` operators to access the enties of a map. `VALUE` is the default. See p.221.
To access the whole entry, use the `ENTRY` keyword. It can be only used in `SELECT` clauses. `KEY` and `VALUE` can also be used in `WHERE` and `HAVING` clauses. JPA is not able to join the source entitiy against map values.

#### Outer joins p.221

#### Fetch joins

Used to optimize DB access and prepare query results to detachment. Fetch joins can be useed to enforce eager loading of lazily loading properties (assuming `address` is lazy):

`SELECT e FROM Employee e JOIN FETCH e.address`

This query will be replaced by the provider wiht a query with a regular join, where one of the returned entities would then be dropped:

` SELECT e, a FROM Employee e JOIN e.address a` 

Selecting all departments and eagerly fetching the employees requires an outer join, as departments with no employees should be returned as well (and would not be included due to a join).

`SELECT d, FROM Department d LEFT JOIN FETCH d.employees` would be turned by the provider into:
`SELECT d, e FROM Department d LEFT JOIN d.employees e` ...it' a bit more complicated in fact, as we would like to select disctinct departments, so there is no SQL equivalent to a fetch query, thats why the employees are fetched too and then dropped.
This "extra" work w´may result in a performance issue for large result sets. Due to this implications, if reations require eager fetching, consider marking them as such.

## WHERE clause

### Input params (already tried in ch.7)
### Basic expression form












