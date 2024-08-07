---
title: "Demystifying Postgres Schema Migrations: Building it from Scratch"
image: /images/2024-demystifying-postgres-schema-migrations/2024-demystifying-postgres-schema-migrations.png
date: 2024-08-07 21:24:45
keywords: Postgres, Schema Migration, Python, asyncpg, CI/CD
tags:
  - Postgres
  - Python
  - asyncpg
---

Database schema migrations are rarely taught in universities, and if you're a self-taught developer, migrations are not something you would naturally pick up.

In this post, we’ll learn about what issues they solve and how they work. We’ll also build a simple implementation. I believe the best way to learn something is to build it from scratch. Even if you end up using a more mature library, by building from scratch you’ll learn about the internal concepts which will help you a lot in troubleshooting. 

![](/images/2024-demystifying-postgres-schema-migrations/2024-demystifying-postgres-schema-migrations-01.gif)

## Why do we need this?

If you work with databases but are unfamiliar with migrations, how do you make schema changes like adding new tables, altering columns, etc? There's a good chance you directly execute SQL commands in the database when the schema changes. 

If this works, then what's wrong with this approach? 
* Directly executing SQL commands is a manual process and it's easy to make mistakes. 
* If you have multiple environments like dev, staging, and prod, there's a chance some of the steps might get missed out.
* There's no record of what changes were applied and in which sequence.
* If two people on the team want to make changes to the schema, there's a chance of conflicts.

## How do migrations solve these issues?

Migrations are written as plaintext files, usually in SQL or using ORM methods. These files are stored along with the application code in version control systems like Git. When an application is being deployed, the CI/CD pipeline or the application startup code executes the migration files and then proceeds with running the application.

As the migrations are stored in version control, they can be deterministically executed in multiple environments. Code reviews can be used to notify other developers of changes and to seek feedback.

## Building it from scratch

Now that we know why they're important, let's learn how they work. The best way to learn something is to build a simple version from scratch. 

Based on the above, we need the following:
1. Every time there's a schema change, we write the change as an SQL file.
2. There should be a way for these files to indicate the order in which they need to be executed.
3. The changes that are previously applied are tracked somewhere so they're not applied again.

Points 1 & 2 can be achieved by creating an SQL file for each schema change and prefixing the file names with an auto-incrementing integer or timestamps:

```shell
src/
`-- migrations
    |-- 001_init_database.sql
    |-- 002_create_users_table.sql
    |-- 003_add_email_to_users_table.sql
    |-- 004_create_audit_table.sql
    `-- 005_create_users_index.sql
```

Point 3 can be solved by creating a table `schema_migrations` in the database to keep track of all the applied migrations:

```sql
CREATE TABLE IF NOT EXISTS schema_migrations (
    version     INTEGER   PRIMARY KEY,
    name        TEXT      NOT NULL,
    migrated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

```sql
+-------------+----------------------------------+----------------------------+
|   version   |               name               |         migrated_at        |
+-------------+----------------------------------+----------------------------+
|     1       | 001_init_database                | 2024-08-06 10:15:30.123456 |
|     2       | 002_create_users_table           | 2024-08-06 11:30:45.789012 |
|     3       | 003_add_email_to_users_table     | 2024-08-07 09:20:15.456789 |
|     4       | 004_create_audit_table           | 2024-08-07 10:05:00.234567 |
|     5       | 005_create_users_index           | 2024-08-08 14:45:30.901234 |
+-------------+----------------------------------+----------------------------+
```

So, during application startup, we can do the following:
1. Get the list of all applied migrations
2. Check the migrations dir to see if there are new unapplied migrations
3. Apply these pending migrations one by one

That's pretty much it for the simple version! Let's implement the above logic.

## Implementation

We'll use Python and [asyncpg](https://magicstack.github.io/asyncpg/current/) for this implementation.

Let's start with creating the `schema_migrations` table if it doesn't exist:

```python
import asyncpg

async def create_table():
    query = """
        CREATE TABLE IF NOT EXISTS schema_migrations (
            version     INTEGER   PRIMARY KEY,
            name        TEXT      NOT NULL,
            migrated_at TIMESTAMP NOT NULL DEFAULT NOW()
        );
    """
    connection = await asyncpg.connect(user='postgres')
    await connection.execute(query)
    await connection.close()
```

Next, we get all the pending migrations that are not applied yet. For this we need to get all the migration files, retrieve the applied versions from `schema_migrations` table, and then filter out the migration files that are already applied.

```python
from pathlib import Path

async def get_pending_migrations():
    # Get all migration files
    migrations = []
    for path in Path('./src/migrations').iterdir():
        if not path.is_file():
            continue
        migration = {}
        migration["name"] = path.name
        migration["content"] = path.read_text()
        migration["version"] = int(path.name.split("_")[0])
        migrations.append(migration)

    # Get all applied versions
    query = "SELECT version from schema_migrations ORDER BY version ASC"
    connection = await asyncpg.connect(user='postgres')
    records = await connection.fetch(query)
    applied_versions = [r["version"] for r in records]
    await connection.close()

    # Filter out applied migrations
    migrations = [m for m in migrations if m["version"] not in applied_versions]

    # Sort migrations by version
    migrations = sorted(migrations, key=lambda m: m['version'])
    return migrations
```

Finally, we apply these pending migrations one by one inside transactions.

```python
async def apply_pending_migrations():
    await create_table()
    migrations = await get_pending_migrations()

    connection = await asyncpg.connect(user='postgres')
    try:
        async with connection.transaction():
            for migration in migrations:
                await connection.execute(migration["content"])
                await connection.execute("INSERT INTO schema_migrations (version, name) VALUES ($1, $2)", migration["version"], migration["name"])
    finally:
        await connection.close()
```

That’s it! This is a simple implementation of a database schema migration script. This does forward-only migration, and applies all the pending migrations. This can be updated to perform rollbacks and migration to a specific version if needed. I’ve also omitted validation and error handling logic for sake of simplicity. These can be added based on your needs. 

## Handling breaking changes
One common issue with migrations is when you perform a breaking schema change. You have updated the application code to accommodate the new schema change, but the code that’s currently running in production would break if you apply this migration. 

There are two ways to fix this:

**Downtime:** This approach is simplest and straightforward. If your users are accommodating, you can take down your application, run the schema migrations and then restart the application with new code.

**Expand-Contract:** If you can't have downtime, then you need to use the "expand-contract" approach. Here you perform migrations multiple times. First you do a backwards compatible schema migration so that the application code running in production won’t break. The new code that is able to handle the new schema changes is deployed. We then perform another schema migration to remove the remnants of the old schema. This requires careful planning and can go wrong if it’s not tested properly before execution.

## Further Reading
* [GitLab Migration Style Guide](https://docs.gitlab.com/ee/development/migration_style_guide.html)
* [Common DB schema change mistakes](https://postgres.ai/blog/20220525-common-db-schema-change-mistakes)
* [Schema changes and the Postgres lock queue](https://xata.io/blog/migrations-and-exclusive-locks)