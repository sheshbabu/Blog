---
title: Unit testing Express route handlers
description: Using node-mocks-http to write unit tests for Express route handlers
date: 2017-07-08 10:22:49
keywords: JavaScript, Nodejs, Expressjs, Mocha
tags:
- JavaScript
- Testing
- Node.js
- Express.js
---

We can unit test [Express.js](https://expressjs.com/) route handler functions using a mocking library called [node-mocks-http](https://github.com/howardabrams/node-mocks-http)

Let's say we've a simple express app

```javascript
// index.js
const express = require("express");
const exampleRouter = require("./example-router");
const app = express();

app.use("/example", exampleRouter);

app.listen(3000);
```

With route handler defined separately as

```javascript
// example-router.js
function exampleRouteHandler(req, res) {
  res.send("hello world!");
}

module.exports = exampleRouteHandler;
```

For unit testing, we should be able to pass various inputs and see if we get correct outputs. Here, we should be able to pass valid request (`req`) and response (`res`) objects as inputs and since this function doesn't return anything, we should be able to make assertions on the response object (`res`).


We can do this by using node-mocks-http's [createRequest](https://github.com/howardabrams/node-mocks-http#createrequest) and [createResponse](https://github.com/howardabrams/node-mocks-http#createresponse) apis.

Let's write a simple test for this using [mocha](https://github.com/mochajs/mocha)

```javascript
// example-router.test.js
const assert = require("assert");
const httpMocks = require("node-mocks-http");
const exampleRouteHandler = require("./example-router");

describe("Example Router", () => {

  it("should return 'hello world' for GET /example", () => {
    const mockRequest = httpMocks.createRequest({
      method: "GET",
      url: "/example"
    });
    const mockResponse = httpMocks.createResponse();

    exampleRouteHandler(mockRequest, mockResponse);

    const actualResponseBody = mockResponse._getData();
    const expectedResponseBody = "hello world!";
    assert(actualResponseBody, expectedResponseBody);
  });

});
```

Checkout the [node-mocks-http repo](https://github.com/howardabrams/node-mocks-http) for more info.
