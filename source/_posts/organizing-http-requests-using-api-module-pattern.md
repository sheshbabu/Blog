---
title: Organizing HTTP requests using the API module pattern
description: Improve readability and testability by abstracting the HTTP code from the UI or business code
date: 2019-08-17 16:47:02
keywords: HTTP, Frontend, API, Axios, Fetch, Clean Code, Organization, Design Patterns
image: /images/2019-api/image-2.png
tags:
  - JavaScript
  - Maintainability
---

Let's say you're writing a frontend for an online store. You would have to make requests to get the shopping cart, add items to the cart, get product details, search for a product, list all products, etc.

If you're directly calling `fetch` or `axios` in your code, they would look something like this.

```js
// HomePage.js

async function getAllProducts() {
  const headers = { "x-secret-header": "ssshhh!" };
  const response = await fetch("/api/products", { headers });
  const body = await response.json();
  return body;
}
```

```js
// CartPage.js

async function addToCart(itemId, qty) {
  const headers = { "x-secret-header": "ssshhh!" };
  const payload = JSON.stringify({ itemId, qty });
  const response = await fetch("/api/cart", {
    method: "POST",
    headers,
    body: payload
  });
  const body = await response.json();
  return body;
}
```

![](/images/2019-api/image-1.png)

This is totally fine if you're only making a handful of requests, but if your codebase makes a lot of HTTP requests, it might be better to abstract them into their own modules instead of calling them directly.

## What is an API module?

An API module is just a [JS module](https://v8.dev/features/modules) that contains HTTP logic organized by business domain. For the online store example, the business domains would be `Cart`, `Search`, `Inventory`, `Product Catalog`, `Order`, etc. The respective API modules would be `CartApi`, `SearchApi`, `InventoryApi`, `CatalogApi` and `OrderApi`.

Let's rewrite the above code using the API module pattern by creating `CartApi` and `CatalogApi` modules.

```js
// CatalogApi.js

export async function getAllProducts() {
  const headers = { "x-secret-header": "ssshhh!" };
  const response = await fetch("/api/products", { headers });
  const body = await response.json();
  return body;
}
```

```js
// CartApi.js

export async function addToCart(itemId, qty) {
  const headers = { "x-secret-header": "ssshhh!" };
  const payload = JSON.stringify({ itemId, qty });
  const response = await fetch("/api/cart", {
    method: "POST",
    headers,
    body: payload
  });
  const body = await response.json();
  return body;
}
```

Now we can import these into the UI modules:

```js
// HomePage.js

import * as CatalogApi from "../api/CatalogApi";

CatalogApi.getAllProducts();
```

```js
// CartPage.js

import * as CartApi from "../api/CartApi";

CartApi.addToCart(itemId, qty);
```

![](/images/2019-api/image-3.png)

## Benefits

- Improve readability and testability by abstracting the HTTP code away from the UI or business code
- One place to make modifications like renaming payload structure, query param names etc
- One place to massage response into something that's useful for the other parts of the codebase
- Organizing HTTP code by business domain thereby improving code discoverability by new members of the team
- Colocate custom app status codes sent by the server

## HttpClient

If we see that there's a lot of common code used in API modules, we can add one more layer called `HttpClient` or `ApiClient` to keep them [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). The common code can be things like:

- Adding extra headers
- Logging
- Using right config for production, development etc — hostnames, headers, etc
- Session handling etc

```js
// HttpClient

export default async function request(path, method, body) {
  const headers = {}; // add custom headers here
  const url = ""; // add logic to get correct url for the environment

  const response = await fetch(url, { method, body, headers });

  // add response handling
  // - session management
  // - convert to correct data type - response.json(), etc
  // - handle common errors like 401, 403 etc

  return response;
}
```

We can now update our API modules to use HttpClient:

```js
// CatalogApi.js

import * as HttpClient from "./HttpClient";

export function getAllProducts() {
  return HttpClient.request("/api/products");
}
```

```js
// CartApi.js

import * as HttpClient from "./HttpClient";

export function addToCart(itemId, qty) {
  return HttpClient.request("/api/cart", "POST", { itemId, qty });
}
```

![](/images/2019-api/image-2.png)

Thanks for reading! :)
