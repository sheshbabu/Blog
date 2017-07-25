---
title: Disabling Bunyan in tests
description: Disable Bunyan logs when running tests
date: 2017-07-25 22:03:33
keywords: JavaScript, Nodejs, Bunyan, Logging
tags:
- JavaScript
- Testing
- Node.js
- Bunyan
- Logging
---

Bunyan (as of v1.8.10) doesn't provide an explicit api to mute logging. This is especially useful when you're running unit tests and don't want logs and test reports to be mixed. 

One workaround [suggested](https://github.com/trentm/node-bunyan/issues/456#issuecomment-259052991) by the author is to set the log level to a value above `FATAL`.

Set an environment variable when running tests:

```javascript
// package.json
...
...
  "scripts": {
    "test": "NODE_ENV=test mocha"
  }
...
...
```

Check for it in the logger module and if it matches, set the log level above `FATAL`.

```javascript
// logger.js
const bunyan = require("bunyan");
const logger = bunyan.createLogger({name: "myapp"});

if (process.env.NODE_ENV === "test") {
  logger.level(bunyan.FATAL + 1);
}

module.exports = logger;
```

 This disables logging when running tests.
