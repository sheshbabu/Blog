---
title: Measuring response times of Express route handlers
description: Measuring response times of Express route handlers
date: 2018-01-27 18:57:03
keywords: JavaScript, Nodejs, Expressjs
tags:
- JavaScript
- Node.js
- Express.js
---

If we want to measure the total time taken for each route in an Express server, we can do so by adding a common middleware that tracks the time elapsed between the moment a request comes and before the response is sent back.

```javascript
// index.js
const express = require("express");
const logResponseTime = require("./response-time-logger");
const app = express();

app.use(logResponseTime);

app.get("/", (req, res) => {
  res.send("hello");
});

app.get("/slow", (req, res) => {
  for (let i = 0; i < 1e10; i++) {}
  res.send("hello");
});

app.listen(3000);
```

```javascript
// response-time-logger.js
function logResponseTime(req, res, next) {
  const startHrTime = process.hrtime();

  res.on("finish", () => {
    const elapsedHrTime = process.hrtime(startHrTime);
    const elapsedTimeInMs = elapsedHrTime[0] * 1000 + elapsedHrTime[1] / 1e6;
    console.log("%s : %fms", req.path, elapsedTimeInMs);
  });

  next();
}

module.exports = logResponseTime;
```

Calling the above two endpoints would log
```
/ : 1.791791ms
/slow : 18541.045675ms
```

The `index.js` is the entrypoint to our server and that's usually the place where all the common middlewares are added. The `logResponseTime` middleware needs to be at the top because we want to be as accurate as possible when we measure the time taken for each route in our Express server. The [process.hrtime](https://nodejs.org/api/process.html#process_process_hrtime_time) api is used to measure the elapsed time as it is more accurate than just using the `Date` api.