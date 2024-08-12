---
title: Implementing SAML SSO in Node.js with Microsoft Entra ID
description: Authenticate with Microsoft Entra ID (Azure Active Directory) using Node.js, Express and Passport
date: 2024-08-11 22:11:30
keywords: SAML Authentication Flow, SAML Session, SSO, Passport, Express, Azure Active Directory, Microsoft Entra ID
tags:
  - SAML
  - Authentication
  - JavaScript
  - Node.js
---

SAML is one of the commonly used standards for implementing SSO in enterprise environments. Even though OIDC is rapidly gaining traction, not everyone supports it, or there are compliance requirements that mandate SAML.

![](/images/2020-saml/image-2.png)

For a conceptual overview of how the SAML flow works, please refer to [this post](https://www.sheshbabu.com/posts/visual-explanation-of-saml-authentication/).

We'll use [Microsoft Entra ID](https://learn.microsoft.com/en-sg/entra/fundamentals/new-name?culture=en-sg&country=sg) (Azure Active Directory) as the Identity Provider (IdP).

Let's start with a simple Express example:

```js
// index.js

import express from "express";

const app = express();

const router = express.Router();

router.get("/public", (req, res) => {
  res.send("Please login");
});

router.get("/private", (req, res) => {
  res.send("Welcome home!");
});

app.use("/api", router);

app.listen(3000, () => {
  console.log("Server is running on port 3000");
});
```

We've two routes: `/api/public` is public and can be seen by anyone, `/api/private` is private and should only be visible to logged-in users. If the IdP requires the server to use HTTPS, please refer to [this post](https://www.sheshbabu.com/posts/running-express-over-https-in-localhost/) on how to implement HTTPS in Express.

We'll add the SAML auth logic in a separate middleware module to keep things clean. Let's start with a dummy implementation:

```js
// saml.js

import express from "express";

const router = express.Router();

router.get("/login", (req, res, next) => {
  res.redirect("/api/private");
});

router.post("/login/callback", (req, res, next) => {
  res.redirect("/api/private");
});

export default router;
```

Update `index.js` to use the SAML middleware.

```diff
  // index.js
  
  import express from "express";
+ import SamlAuth from "./saml.js";
  
  const app = express();
  
+ app.use(SamlAuth);
  
  const router = express.Router();
  
  router.get("/public", (req, res) => {
    res.send("Please login");
  });
  
+ router.use(ensureLoggedIn);
  
  router.get("/private", (req, res) => {
    res.send("Welcome home!");
  });
  
  app.use("/api", router);
  
+ function ensureLoggedIn(req, res, next) {
+   if (!req.isAuthenticated()) {
+     return res.redirect("/login");
+   } else {
+     next();
+   }
+ }
  
  app.listen(3000, () => {
    console.log("Server is running on port 3000");
  });
```

We've also added an `ensureLoggedIn` middleware to validate whether an user is logged-in for the private routes and redirect them to login page if they're not logged-in.

Next, we will configure `passport` and `passport-saml` in `saml.js`:

```diff
  // saml.js

  import express from "express";
+ import passport from "passport";
+ import { Strategy } from "passport-saml";

+ const SAML_ISSUER = "xxx";
+ const SAML_CERT = "xxx";

+ const config = {
+   entryPoint: "https://login.microsoftonline.com/{tenantId}/saml2", // replace 'tenant' before using
+   callbackUrl: "http://localhost:3000/login/callback",
+   issuer: SAML_ISSUER,
+   cert: SAML_CERT,
+   logoutUrl: "/logout"
+ };

  const router = express.Router();

+ passport.use(
+   new Strategy(config, (profile, cb) => {
+     const email = profile["http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"];
+     const name = profile["http://schemas.microsoft.com/identity/claims/displayname"];
+     return cb(null, { email, name });
+   })
+ );

  router.get("/login", (req, res, next) => {
    res.redirect("/api/private");
  });
  
  router.post("/login/callback", (req, res, next) => {
    res.redirect("/api/private");
  });

  export default router;
```

The values for `tenantId`, `SAML_ISSUER`, and `SAML_CERT` would be available while registering your application in Microsoft Entra ID.

We'll then [configure the session](https://www.passportjs.org/tutorials/password/session/) so the users don't need to keep re-logging-in. I've used SQLite as session store, but you can replace it with Redis, Postgres etc.

```diff
  // saml.js

  import express from "express";
  import passport from "passport";
  import { Strategy } from "passport-saml";
+ import session from "express-session";
+ import connectSqlite3 from "connect-sqlite3";
+ import cookieParser from "cookie-parser";

+ const SQLiteStore = connectSqlite3(session);
+ const SESSION_SECRET = "xxx";

  const SAML_ISSUER = "xxx";
  const SAML_CERT = "xxx";

  const config = {
    entryPoint: "https://login.microsoftonline.com/{tenantId}/saml2", // replace 'tenant' before using
    callbackUrl: "http://localhost:3000/login/callback",
    issuer: SAML_ISSUER,
    cert: SAML_CERT,
    logoutUrl: "/logout"
  };

  const router = express.Router();

  passport.use(
    new Strategy(config, (profile, cb) => {
      const email = profile["http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"];
      const name = profile["http://schemas.microsoft.com/identity/claims/displayname"];
      return cb(null, { email, name });
    })
  );

+ passport.serializeUser((user, cb) => {
+   cb(null, user);
+ });

+ passport.deserializeUser((user, cb) => {
+   cb(null, user);
+ });

+ router.use(cookieParser());

+ router.use(
+   session({
+     secret: SESSION_SECRET,
+     resave: false,
+     saveUninitialized: false,
+     store: new SQLiteStore({ db: "sessions.db" })
+   })
+ );

+ router.use(passport.initialize());
  
+ router.use(passport.session());

  router.get("/login", (req, res, next) => {
    res.redirect("/api/private");
  });
  
  router.post("/login/callback", (req, res, next) => {
    res.redirect("/api/private");
  });

  export default router;
```

We'll update the `/login/*` endpoints and also add a logout endpoint

```diff
  // saml.js

  import express from "express";
  import passport from "passport";
  import { Strategy } from "passport-saml";
  import session from "express-session";
  import connectSqlite3 from "connect-sqlite3";
  import cookieParser from "cookie-parser";
  
  const SQLiteStore = connectSqlite3(session);
  const SESSION_SECRET = "xxx";
  
  const SAML_ISSUER = "xxx";
  const SAML_CERT = "xxx";
  
  const config = {
    entryPoint: "https://login.microsoftonline.com/{tenantId}/saml2", // replace 'tenant' before using
    callbackUrl: "http://localhost:3000/login/callback",
    issuer: SAML_ISSUER,
    cert: SAML_CERT,
    logoutUrl: "/logout"
  };
  
  const router = express.Router();
  
  passport.use(
    new Strategy(config, (profile, cb) => {
      const email = profile["http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"];
      const name = profile["http://schemas.microsoft.com/identity/claims/displayname"];
      return cb(null, { email, name });
    })
  );
  
  passport.serializeUser((user, cb) => {
    cb(null, user);
  });
  
  passport.deserializeUser((user, cb) => {
    cb(null, user);
  });
  
  router.use(cookieParser());
  
  router.use(
    session({
      secret: SESSION_SECRET,
      resave: false,
      saveUninitialized: false,
      store: new SQLiteStore({ db: "sessions.db" }),
    })
  );
  
  router.use(passport.initialize());
  
  router.use(passport.session());

- router.get("/login", (req, res, next) => {
-   res.redirect("/api/private");
- });

+ router.get("/login", passport.authenticate("saml"));

- router.post("/login/callback", (req, res, next) => {
-   res.redirect("/api/private");
- });

+ router.post("/login/callback", express.urlencoded({ extended: false }), passport.authenticate("saml"), (req, res) => {
+   res.redirect("/api/private");
+ });

+ router.post("/logout", (req, res, next) => {
+   req.logout((err) => {
+     if (err) {
+       return next(err);
+     }
+     req.session.destroy(() => {
+       res.clearCookie("connect.sid");
+       res.redirect("/api/public");
+     });
+   });
+ });

  export default router;
```

Please note the `express.urlencoded` middleware added to `/login/callback`. If this is skipped, you'll face infinite login loops.

Related posts:
* [Visual explanation of SAML authentication](https://www.sheshbabu.com/posts/visual-explanation-of-saml-authentication/)
* [Running Express over HTTPS in localhost](https://www.sheshbabu.com/posts/running-express-over-https-in-localhost/)