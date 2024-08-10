---
title: "FastAPI without ORM: Testing with asyncpg"
image: /images/2024-fastapi-without-orm-testing-with-asyncpg/2024-fastapi-without-orm-testing-with-asyncpg.png
date: 2024-08-10 10:04:59
keywords: Python, FastAPI, asyncpg, Postgres
tags:
  - Testing
  - Python
  - FastAPI
  - asyncpg
  - Postgres
---

In our last two posts, we explored how to [setup asyncpg with FastAPI](https://www.sheshbabu.com/posts/fastapi-without-orm-getting-started-with-asyncpg/) and how to write [database schema migrations](https://www.sheshbabu.com/posts/demystifying-postgres-schema-migrations/). 

We'll write end-to-end integration tests that will exercise the code from API endpoints through business logic to database queries. We will create tests for two endpoints which can then be extended to more endpoints. We'll be following the [same codebase](https://github.com/sheshbabu/fastapi-asyncpg-demo) from the previous two posts.

### Setup and Teardown

To ensure that the tests are independent from each other, we must reset the database and apply migrations before and after each test. We can use `fixtures` in pytest to implement this in a clean manner:

```python
import pytest
from src.commons.postgres import database
from src.commons import migrate


@pytest.fixture()
async def setup_database():
    await database.connect()
    async with database.pool.acquire() as connection:
        await connection.execute("CREATE SCHEMA IF NOT EXISTS public;")
    await migrate.apply_pending_migrations()

    yield
    
    async with database.pool.acquire() as connection:
        await connection.execute("DROP SCHEMA IF EXISTS public CASCADE;")
    await database.disconnect()
```

### Testing GET /users endpoint

Now that we've the setup/teardown in place, let's start writing the first set of tests. We'll focus on the [`GET /users`](https://github.com/sheshbabu/fastapi-asyncpg-demo/blob/master/src/users/users_route.py) endpoint.

```python
from httpx import AsyncClient, ASGITransport
from src.main import app

@pytest.fixture
async def client_app():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        yield client


@pytest.mark.asyncio
async def test_get_users_returns_empty_list_when_no_users_exist(setup_database, client_app):
    response = await client_app.get("/users/")
    assert response.status_code == 200
    assert response.json() == []


@pytest.mark.asyncio
async def test_get_users_returns_users_list_when_users_exist(setup_database, client_app):
    async with database.pool.acquire() as connection:
        await connection.execute("INSERT INTO users (name, email) VALUES ($1, $2)", "Shesh", "sheshbabu@gmail.com")
    response = await client_app.get("/users/")
    assert response.status_code == 200
    assert response.json() == [{"email": "sheshbabu@gmail.com", "name": "Shesh"}]
```

Here, we added two scenarios - one with an empty table and another with a single record. We've used the `app` object to make the calls. This exercises the code paths all the way to database and back.

### Testing POST /users endpoint

We'll go through one more example so we know how the code is organized.

```python
@pytest.mark.asyncio
async def test_post_users_returns_success_status_when_valid_payload_is_provided(setup_database, client_app):
    payload = {"name": "Shesh", "email": "sheshbabu@gmail.com"}
    response = await client_app.post("/users/", json=payload)
    assert response.status_code == 200
```

That's it! We can now use this approach to add more test cases for this app.

Related posts:
* [FastAPI without ORM: Getting started with asyncpg](https://www.sheshbabu.com/posts/fastapi-without-orm-getting-started-with-asyncpg/)
* [Demystifying Postgres Schema Migrations: Building it from Scratch](https://www.sheshbabu.com/posts/demystifying-postgres-schema-migrations/)
* [Structured JSON Logging using FastAPI](https://www.sheshbabu.com/posts/fastapi-structured-json-logging/)