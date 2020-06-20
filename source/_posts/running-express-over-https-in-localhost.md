---
title: Running Express over HTTPS in localhost
description: Running Express over HTTPS in localhost
keywords: Express, HTTPS, mkcert, localhost
date: 2020-06-19 19:29:14
tags:
  - Express.js
  - Node.js
---

There are times when you want to expose your Express server via HTTPS for local development.

Let’s take a super simple Express code:

```javascript
const express = require("express");

const app = express();

app.get("/", (req, res, next) => {
  res.send("Hello World\n");
});

app.listen(3001);
```

This will return "Hello World" when you hit http://localhost:3001

```shell
$ curl http://localhost:3001
Hello World
```

To run this in HTTPS, we need to install a tool called [mkcert](https://github.com/FiloSottile/mkcert). Follow the [installation instructions](https://github.com/FiloSottile/mkcert/blob/v1.4.1/README.md#installation) or if you’re using macOS and Homebrew, run this command:

```shell
$ brew install mkcert
```

Create a local [CA](https://en.wikipedia.org/wiki/Certificate_authority) and create certificate for localhost:

```shell
$ mkcert -install
$ mkcert localhost
```

This will create a certificate file `localhost.pem` and key file `localhost-key.pem` in the current directory.

Update the Express code as follows:

```diff
+ const fs = require("fs");
+ const https = require("https");

const express = require("express");

+ const key = fs.readFileSync("localhost-key.pem", "utf-8");
+ const cert = fs.readFileSync("localhost.pem", "utf-8");

const app = express();

app.get("/", (req, res, next) => {
  res.send("Hello World\n");
});

- app.listen(3001);
+ https.createServer({ key, cert }, app).listen(3001);
```

And you’re done!

```shell
$ curl https://localhost:3001
Hello World
```

You can find the full codebase in [this repo](https://github.com/sheshbabu/express-https-localhost).
