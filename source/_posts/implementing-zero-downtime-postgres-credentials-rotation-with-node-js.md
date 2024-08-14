---
title: Zero-Downtime Postgres Credentials Rotation with Node.js
date: 2024-08-13 22:03:15
keywords: Zero-downtime database updates, Postgres credential rotation, Node.js database management, AWS Secrets Manager integration, Database connection pooling
tags:
  - Postgres
  - JavaScript
  - Node.js
---

In enterprise environments, it's common to store database credentials in a vault like AWS Secrets Manager. These credentials are usually rotated every few weeks based on the company's security policy. 

While this improves security, it adds complexity to the application. Instead of pulling these credentials from environment variables, the application code must now query AWS Secrets Manager and handle cases where database connections are dropped because of credential rotation.

How does one update the database credentials from vault if they are rotated while the application is running?

**Manual Approach:** We can note the date and time when the next credential rotation is scheduled. Before the rotation, we can take the application offline and then restart it after the rotation has completed. This approach is simple but involves downtime. If you're in an enterprise environment, you likely have many databases and applications, and this approach might not be feasible.

**Brute-force Approach:** Before each database query, retrieve the latest set of credentials from vault and then use that credentials to establish the connection to database. This approach works, but it's very wasteful and would degrade the performance. It will also cost a lot since AWS [charges for API calls](https://aws.amazon.com/secrets-manager/pricing/) to Secrets Manager.

## Better Approach

Is there a better approach? Instead of querying the vault for credentials before each database connection, wouldn't it be more efficient if we fetch the credentials from vault during application startup, store them in memory, and then refresh them after they get rotated? This would be ideal, but how would the application know that the credentials have rotated?

The key to solving this is to understand what happens to connections when credentials get rotated. Once a connection is established using a set of credentials, it will work until it gets terminated. So, if a connection is used to execute a query, and while it's under execution, the credentials got rotated, the connection will still work! It won't get cut-off in the middle. 

In complex applications, connections are managed in a pool. If the pool is empty and a query needs to be run, then a new connection is established and used to run the query. After running this query, the connection is released back into the pool. It's not terminated until the idle timeout has passed. So, the connections established using the old credentials can still be used if they're present in the pool.

However, establishing new connections with old credentials would fail. We can catch these connection errors, then fetch new credentials from the vault and reestablish the connection.

## Implementation

Let's start with a simple implementation using [node-postgres](https://node-postgres.com):

```js
// ./database.js

import pg from "pg";

const OLD_CONNECTION_STRING = "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable";
const NEW_CONNECTION_STRING = "postgres://postgres:new_password@localhost:5432/postgres?sslmode=disable";

let pool = null;

async function query(query, params) {
  if (pool === null) {
    console.log("creating new pool");
    pool = new pg.Pool({ connectionString: OLD_CONNECTION_STRING });
  }

  const client = await pool.connect();
  client = await pool.connect();

  try {
    return await client.query(query, params);
  } finally {
    client.release();
  }
}

export default { query };
```

This can be used as follows:

```js
// ./index.js

import Database from "./database.js";

await Database.query(`
    CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT NOT NULL
    )`);

for (let i = 0; i < 100; i++) {
  await Database.query("INSERT INTO users (name, email) VALUES ($1, $2)", [`user_${i}`, `user_${i}@email.com`]);
  await sleep(1000);
}

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

This creates a table and inserts 100 records in a loop. I've added a `sleep` so the loop takes 1-2 minutes to run.

Let's run this, and while it's running, we can change the database credentials by running this query:

```sql
ALTER USER postgres WITH PASSWORD 'new_password';
```

The script errors out and exits. If we look at the stack trace, we can see the error object has an attribute called `routine` with value `auth_failed`.

We can catch this error while connecting, fetch new credentials from vault and then re-create the pool with this new credentials:

```diff
  // ./database.js
  
  import pg from "pg";
  
  const OLD_CONNECTION_STRING = "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable";
  const NEW_CONNECTION_STRING = "postgres://postgres:new_password@localhost:5432/postgres?sslmode=disable";
  
  let pool = null;
  
  async function query(query, params) {
    if (pool === null) {
      console.log("creating new pool");
      pool = new pg.Pool({ connectionString: OLD_CONNECTION_STRING });
    }
  
-   const client = await pool.connect();
-   client = await pool.connect();

+   let client;
+   try {
+     client = await pool.connect();
+   } catch (e) {
+     if (e.routine !== "auth_failed") {
+       throw e;
+     }

+     console.log("auth failed, trying new password")
+     pool = new pg.Pool({ connectionString: NEW_CONNECTION_STRING });
+     client = await pool.connect();
+   }
  
    try {
      return await client.query(query, params);
    } finally {
      client.release();
    }
  }
  
  export default { query };
```

Now, if you reset the password to the old one, and then try the same flow again, you'll quickly see a `"auth failed, trying new password"` log and the script carries on without any interruptions.

I've used the `NEW_CONNECTION_STRING` from a variable, but you can update the above code so it fetches this from AWS Secrets Manager, Azure Key Vault, etc. It's also advisable to throttle the calls to these vault APIs so if there are multiple connections that are erroring out, there's only one call that retrieves the new credentials and creates a new pool.