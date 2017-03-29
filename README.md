# Composable SQL

This document explores how whitespace-sensitive SQL-like queries can be composed and transpiled to regular SQL. In addition, it proposes an application-side interface to compose and run these queries.

## Clause Ordering

Let's say we want to merge:

**category.sql**

```sql
select * from products p where p.category = "Swiss Morning Urine" order by popularity
```

And:

**cheerleaders.sql**

```sql
select * from products p 
left join cheerleader_endorsements e on (p.id = e.product_id)
left join cheerleaders c on (e.cheeleader_id = c.id)
where c.age > 99 order by c.hip_replacements
```

Into:

```sql
select * from products p
left join cheerleader_endorsements e on (p.id = e.product_id)
left join cheerleaders c on (e.cheeleader_id = c.id)
where p.category = 'Swiss Morning Urine'
where c.age > 99
order by popularity order by c.hip_replacements
```

Note that this is not simple concatenation. We removed one of the `select * from products p`s and we reordered the clauses, putting the joins first, the where clauses second, and the ordering clauses last. Especially the ordering requires a deep understanding of the SQL. This means that a solution for one SQL dialect would not work in another.

Composable SQL works around this by requiring all composable pieces to start on a new line:

**category.sql**

```sql
select * from products p
where p.category = 'Swiss Morning Urine'
order by popularity
```

Since these clauses will be ordered as a part of the composition, the order of these pieces is less strict than in typical SQL:

**cheerleaders.sql**

```sql
order by c.hip_replacements
where c.age > 99
select * from products p
left join cheerleader_endorsements e on (p.id = e.product_id)
left join cheerleaders c on (e.cheeleader_id = c.id)
```

We can still spread single pieces over multiple lines. If a line starts with a keyword that does not denote the start of an SQL clause, it is assumed to be part of the clause above:

```sql
select * from products p
where p.category = 'Swiss Morning Urine'
and p.price < 1000
order by popularity
```

These queries can be parsed by splitting them on newlines preceding SQL clause keywords, which is trivial in most programming languages.

## Composition

In the pseudo-code below, we will assume the queries to be represented as lists of two-tuples. The query above looks like this:

```
[
  {"select", "* from products p"},
  {"where", "p.category = 'Swiss Morning Urine'
  and p.price < 1000"},
  {"order by", "popularity"}
]
```
