> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sql-bits.com](https://sql-bits.com/the-difference-between-where-and-having/)

> SQL clauses WHERE and HAVING have different functions. But they both filter out rows, so many people ......

_SQL clauses_ `WHERE` _and_ `HAVING` _have different functions. But they both filter out rows, so many people don’t know the important difference between them. Let’s shred some light._

WHERE
-----

In a single-table query, `WHERE` comes in at the beginning of a query execution. We used to think that it determines which rows will be returned by the query, but this is not accurate:

**`WHERE` determines which rows will be processed by the query.**

The difference becomes clear when we use a `GROUP BY` clause.

HAVING
------

HAVING comes in at the end of a query execution.

**After all rows have been processed, `HAVING` determines which of rows will be sent to the client.**

The differences
---------------

The theory should be clear. But let’s see the differences between `WHERE` and `HAVING` in practice.

### GROUP BY

The difference between `WHERE` and `HAVING` becomes clear when we run a query with `GROUP BY`:

```
SELECT department_id, count(*) AS employees_no
    FROM employee
    WHERE gender = 'F'
    GROUP BY department_id
    HAVING employees_no < 10;
```

The query counts the number of female employees in each department, and only returns the departments where this number is less than 10.

*   `WHERE` excludes non-female employees. Those rows are not read at all by the query.
*   `GROUP BY` _groups_ (or aggregates) the found rows, producing only one row for each distinct `department_id`.
*   `HAVING` eliminates the _aggregated_ rows where `employees_no` is less than 10.

Note that:

*   `WHERE employees_no < 10` would fail with an error, because that value doesn’t exist before aggregation.
*   `HAVING gender = 'F'` would fail with an error, because the gender column doesn’t exist in the aggregated rows (or, if you prefer, in the `SELECT` clause).

### Query performance

Sometimes you may think that both `WHERE` and `HAVING` can be used, and that they’re equivalent. An example:

```
SELECT *
    FROM employee
    WHERE date_of_birth > '2000-01-01';
```

This query finds the employees that were born in this century.

Could we use `HAVING` instead of `WHERE`? In theory, yes. But in that case we’re telling the database to read all rows, and only return the ones that match the condition. This is unnecessarily slow.

With the `WHERE` clause, we’re asking the database to only read the rows we’re interested in. If there is an index that starts with the date_of_birth column, it will be used.

**Note that some DBMSs are smart enough to translate an unnecessary `HAVING` clause into a `WHERE` clause.** But not all DBMSs do so, and there may always be complex cases when a DBMS fails to apply this optimisation.