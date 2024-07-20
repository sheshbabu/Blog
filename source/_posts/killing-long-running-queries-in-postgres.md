---
title: Killing long running queries in Postgres
description: Finding and terminating long running queries in Postgres
keywords: Postgres
date: 2024-07-20 17:32:04
tags:
  - Postgres
---

To find all the queries that are currently running:

```sql
SELECT
	pid,
	AGE(NOW(), query_start),
	query
FROM
	pg_stat_activity
WHERE
	query_start IS NOT NULL
ORDER BY
	age DESC
```

The above will list all the running queries with its `pid`, `age` and `query`, where `age` is how long the query has been running.

To cancel a specific query, pass its `pid` to `pg_cancel_backend`:

```sql
SELECT pg_cancel_backend(pid)
```

For example, if `pid` is `29212`:

```sql
SELECT pg_cancel_backend(29212)
```

Note that sometimes `pg_cancel_backend` doesn't work. In such cases, you will need to wait for the query to finish.