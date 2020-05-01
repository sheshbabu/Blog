---
title: Ways to manage config in frontend and their tradeoffs
description: Ways to manage config in frontend and their tradeoffs
keywords: JavaScript, Frontend, Environment Variables, Configuration, Config Management
date: 2020-05-01 11:43:36
image: /images/2020-config/image-1.png
tags:
  - JavaScript
---

Let’s say our app has 3 environments - development, staging and production. We might have different configs for each of these environments like backend endpoints, analytics api keys, error reporting api keys, settings like turning off debug logs in production environment etc.

![](/images/2020-config/image-1.png)

There are three common ways to manage these configs:

1. Buildtime injection - For each environment, create a separate bundle and inject environment specific config into code.
2. Runtime resolution - Store the configs for all environments in the same bundle and dynamically resolve the right config in runtime.
3. Remote fetching - Fetch the config from backend during runtime

Let's talk about each of these approaches and their pros and cons.

## Buildtime injection

[SPA](https://en.wikipedia.org/wiki/Single-page_application) apps are usually built using a bundler and deployed to web servers or copied into a nginx docker image and deployed to container platforms.

In this approach, a separate bundle (or docker image) is created for each environment and the config values for that environment is injected into the bundle using "environmental variables". Of course, there’s no such thing as environment variables in frontend so it’s usually mimicked using global variables.

If you’re using [webpack](https://webpack.js.org/plugins/define-plugin/), [create-react-app](https://create-react-app.dev/docs/adding-custom-environment-variables/) or [vue-cli](https://cli.vuejs.org/guide/mode-and-env.html#environment-variables) etc, you can access the config value as `process.env.YOUR_CONFIG_NAME` etc

Drawbacks:

- Since your environment specific config is baked into your bundle, you need to build separate bundles for each environment.
- The way to access the config feels hacky - you need to get it from the `process.env` object which isn’t a part of the browser apis.
- You cannot use this if you’re not using a bundler or a build tool.
- Since the config values would be injected from CI/CD, the values would most likely not be version controlled.

Advantages:

- Since each bundle contains only the config for a specific environment, attackers won’t be able to learn about the existence of other environments and their configs like backend endpoints etc by looking at the bundle. This provides a small safety in form of [security through obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity) but shouldn't be seriously relied upon.

## Runtime resolution

Alternatively, we can store the configs for all environments in the source code and dynamically load the appropriate config for each environment. Usually, different environments would have different urls and we can use this to load the correct config.

Let’s say we have 3 environments - development, staging and production, which have the following urls - `dev.myapp.com`, `staging.myapp.com` and `www.myapp.com` respectively.

The configs for these 3 environments can be stored in `dev.config.json`, `staging.config.json` and `prod.config.json`

```json
// dev.config.json

{
  "API_URL": "dev.myapp.com/api",
  "ANALYTICS_KEY": "xxxaaa",
  "ERROR_REPORTING_KEY": "xxxeee",
  "IS_LOGGING_ENABLED": true
}
```

```json
// staging.config.json

{
  "API_URL": "staging.myapp.com/api",
  "ANALYTICS_KEY": "xxxaaa",
  "ERROR_REPORTING_KEY": "xxxeee",
  "IS_LOGGING_ENABLED": true
}
```

```json
// prod.config.json

{
  "API_URL": "www.myapp.com/api",
  "ANALYTICS_KEY": "yyyaaa",
  "ERROR_REPORTING_KEY": "yyyeee",
  "IS_LOGGING_ENABLED": false
}
```

We can use `window.location.hostname` to decide which config to load in runtime:

```javascript
// Config.js

import prod from "./prod.config.json";
import staging from "./staging.config.json";
import dev from "./dev.config.json";

let config = {};

switch (window.location.hostname) {
  case "www.myapp.com":
    config = prod;
    break;
  case "staging.myapp.com":
    config = staging;
    break;
  case "dev.myapp.com":
    config = dev;
    break;
  default:
    config = dev;
}

export default config;
```

This can be consumed in the application as

```javascript
// ApiClient.js

import axios from "axios";
import Config from "../Config";

const ApiClient = axios.create({ baseURL: Config.API_URL });

export default ApiClient;
```

Usually some config values would be repeated for different environments. For example, you might decide to use the same analytics api key for all non-production environments. We can make the configs [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) by extracting these common config values to common.config.json and merging the configs in Config.js

```json
// common.config.json

{
  "ANALYTICS_KEY": "xxxaaa",
  "ERROR_REPORTING_KEY": "xxxeee",
  "IS_LOGGING_ENABLED": true
}
```

```json
// dev.config.json

{
  "API_URL": "dev.myapp.com/api"
}
```

```json
// staging.config.json

{
  "API_URL": "staging.myapp.com/api"
}
```

```json
// prod.config.json

{
  "API_URL": "www.myapp.com/api",
  "ANALYTICS_KEY": "yyyaaa",
  "ERROR_REPORTING_KEY": "yyyeee",
  "IS_LOGGING_ENABLED": false
}
```

```javascript
// Config.js

import common from "./common.config.json";
import prod from "./prod.config.json";
import staging from "./staging.config.json";
import dev from "./dev.config.json";

let config = {};

switch (window.location.hostname) {
  case "www.myapp.com":
    config = { ...common, ...prod };
    break;
  case "staging.myapp.com":
    config = { ...common, ...staging };
    break;
  case "dev.myapp.com":
    config = { ...common, ...dev };
    break;
  default:
    config = { ...common, ...dev };
}

export default config;
```

Advantages

- Same bundle (or docker image) can be used for all environments
- Works for projects that don’t use bundlers or build tools
- The config values are version controlled and easy to reference as they’re in the same codebase

Drawbacks

- You would be exposing the existence of other environments to attackers. Might be an issue if you're not confident of the security of your undelying infra.

## Remote fetching

Generally, frontend builds don’t take much time so it’s easy to deploy config changes to users within minutes. But sometimes you might need to change the config from backend:

- Use a different set of configs for different user segments - A/B testing, phased rollouts, feature flags etc
- Bypass processes - some enterprises have a lot of processes to go through to deploy a single change to a production site. It’s useful to fetch some configs from backend in those cases.

Despite these advantages, the fact that we’re fetching these from backend complicates things - we need to design the code/interface with latency in mind and implement caching in a way that it doesn’t erode the advantages mentioned above. This approach is commonly used alongside either of the first 2 approaches.

Thanks for reading! :)
