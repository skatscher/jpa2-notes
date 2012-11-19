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
Important differnces to SQL - the domain is not th table but the entity.
 
