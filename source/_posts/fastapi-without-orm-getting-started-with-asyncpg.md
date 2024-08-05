---
title: "FastAPI without ORM: Getting started with asyncpg"
image: /images/2024-fastapi-without-orm-getting-started-with-asyncpg/2024-fastapi-without-orm-getting-started-with-asyncpg.png
date: 2024-08-05 22:38:35
keywords: Python, FastAPI, asyncpg, Postgres
tags:
  - Python
  - FastAPI
  - asyncpg
  - Postgres
---

Let's start with a simple FastAPI example:

```python
# src/main.py

import uvicorn
from fastapi import FastAPI

app = FastAPI()

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0")
```

Now, we want to add database logic to this app. We don't want to use ORM, so we choose [asyncpg](https://magicstack.github.io/asyncpg/current/). Let's see how to integrate asyncpg with the above app.

We first create a `postgres.py` module that encapsulates the database connection, pool, termination, and other logic:

```python
# src/commons/postgres.py

import asyncpg

DATABASE_URL = "postgresql://postgres@localhost/postgres"

class Postgres:
    def __init__(self, database_url: str):
        self.database_url = database_url

    async def connect(self):
        self.pool = await asyncpg.create_pool(self.database_url)

    async def disconnect(self):
        self.pool.close()

database = Postgres(DATABASE_URL)
```

I've hardcoded the database connection string here, but you can update this to pull from environment variables instead. 

You'll also notice that I've instantiated the `Postgres` class as `database` variable. This is intentional, and this module-scoped variable can be imported into other modules without the need for dependency injection.

Let's update `main.py` to connect to the database:

```diff
  # src/main.py
  
  import uvicorn
  from fastapi import FastAPI
+ from contextlib import asynccontextmanager
  
+ from src.commons.postgres import database
  
  
+ @asynccontextmanager
+ async def lifespan(app: FastAPI):
+     await database.connect()
+     yield
+     await database.disconnect()
  
- app = FastAPI()
+ app = FastAPI(lifespan=lifespan)
  
  if __name__ == "__main__":
      uvicorn.run(app, host="0.0.0.0")
```

We've added a [lifespan](https://fastapi.tiangolo.com/advanced/events/) function that connects to the database during application startup and then terminates the connection during shutdown.

Now that the setup is complete, let's create an example table:

```sql
CREATE TABLE IF NOT EXISTS users (
	name  TEXT  NOT NULL,
	email TEXT  NOT NULL
);
```

We then create an equivalent `pydantic` model:

```python
# src/users/users_schema.py

from pydantic import BaseModel

class User(BaseModel):
    name: str
    email: str
```

Let's proceed to write logic for interacting with the above `users` table:

```python
# src/users/users_model.py

from typing import List, Optional
from src.commons.postgres import database
from src.users.users_schema import User

async def get_user_by_email(email: str) -> Optional[User]:
    query = "SELECT name, email FROM users WHERE email = $1"
    async with database.pool.acquire() as connection:
        row = await connection.fetchrow(query, email)
        if row is not None:
            user = User(name=row["name"], email=row["email"]) 
            return user
        return None


async def get_all_users(limit: int, offset: int) -> List[User]:
    query = "SELECT name, email FROM users LIMIT $1 OFFSET $2"
    async with database.pool.acquire() as connection:
        rows = await connection.fetch(query, limit, offset)
        users = [User(name=record["name"], email=record["email"]) for record in rows]
        return users


async def insert_user(user: User):
    query = "INSERT INTO users (name, email) VALUES ($1, $2)"
    async with database.pool.acquire() as connection:
        await connection.execute(query, user.name, user.email)


async def bulk_insert_users(users: List[User]):
    query = "INSERT INTO users (name, email) VALUES ($1, $2)"
    user_tuples = [(user.name, user.email) for user in users]
    async with database.pool.acquire() as connection:
        await connection.executemany(query, user_tuples)
```

Okay, that's a lot of code, but each function should be self-explanatory. As mentioned before, we've used the module-scoped variable `database` from `postgres.py` module. 

We first acquire the connection and then use the specialized methods within it:
* `fetchrow` is for querying a single row.
* `fetch` is for multiple rows.
* `execute` is when you want to execute multiple queries without returning rows.
* `executemany` is same as above and also accepts a list of arguments.

One thing to note is that we need to perform "type conversion" before and after talking to asyncpg. You can see that we're converting the `User` object in `insert_user` and `bulk_insert_users` to primitives or list of tuples before inserting them. We're also converting the return value of asyncpg, which is a [dict-like object](https://magicstack.github.io/asyncpg/current/api/index.html#asyncpg.Record) to `User` type.

We can tie this together with the rest of the application by creating a router:

```python
# src/users/users_route.py

from typing import Optional, List
from fastapi import APIRouter
from src.users import users_model
from src.users.users_schema import User

users_router = APIRouter(prefix="/users")

@users_router.get("/")
async def get_all_users(limit: Optional[int] = 10, offset: Optional[int] = 0):
    return await users_model.get_all_users(limit, offset)

@users_router.get("/{email}")
async def get_user_by_email(email: str):
    return await users_model.get_user_by_email(email)

@users_router.post("/")
async def insert_user(user: User):
    return await users_model.insert_user(user)

@users_router.post("/bulk")
async def bulk_insert_users(users: List[User]):
    return await users_model.bulk_insert_users(users)
```

And then adding it to `main.py`:

```diff
  # src/main.py
  
  import uvicorn
  from fastapi import FastAPI
  from contextlib import asynccontextmanager
  
  from src.commons.postgres import database
+ from src.users.users_route import users_router
  
  @asynccontextmanager
  async def lifespan(app: FastAPI):
      await database.connect()
      yield
      await database.disconnect()
  
  app = FastAPI(lifespan=lifespan)
  
+ app.include_router(users_router)
  
  if __name__ == "__main__":
      uvicorn.run(app=app, host="0.0.0.0")
```

You can find the full codebase in this [repo](https://github.com/sheshbabu/fastapi-asyncpg-demo).


Related posts:
* [Structured JSON Logging using FastAPI](https://www.sheshbabu.com/posts/fastapi-structured-json-logging/)