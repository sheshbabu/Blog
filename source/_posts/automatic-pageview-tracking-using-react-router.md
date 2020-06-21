---
title: Automatic PageView Tracking using React Router
description: Automatic PageView Tracking using React Router
keywords: Google Analytics, Amplitude, React, PageView
date: 2020-06-20 14:27:51
tags:
  - React
  - Analytics
---

Analytics providers like Google Analytics, Amplitude etc work really well with server rendered pages for tracking pageviews but fall short when you want to use them in Single Page Applications (SPA).

You would need to manually call their pageview tracking apis on every navigation.

```javascript
// Google Analytics
ga("set", "page", "/account");
ga("send", "pageview");

// Amplitude
amplitude.logEvent("PageView", { path: "/account" });

// Freshlytics
window.__freshlytics__.sendPageView();
```

For example, let's take a simple React app that has 3 routes:

```javascript
import React from "react";
import { BrowserRouter, Switch, Route, Link } from "react-router-dom";

export default function App() {
  return (
    <BrowserRouter>
      <NavBar />
      <Routes />
    </BrowserRouter>
  );
}

function NavBar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/users">Users</Link>
      <Link to="/about">About</Link>
    </nav>
  );
}

function Routes() {
  return (
    <Switch>
      <Route exact path="/" component={HomePage} />
      <Route exact path="/users" component={UsersPage} />
      <Route exact path="/about" component={AboutPage} />
    </Switch>
  );
}

function HomePage() {
  return <h2>Home</h2>;
}

function UsersPage() {
  return <h2>Users</h2>;
}

function AboutPage() {
  return <h2>About</h2>;
}
```

To track the pageviews, we need to call the tracking api during [mount](https://reactjs.org/docs/react-component.html#mounting) in each of these pages

```diff
function HomePage() {
+ React.useEffect(() => {
+   ga("set", "page", "/");
+   ga("send", "pageview");
+ }, []);

  return <h2>Home</h2>;
}

function UsersPage() {
+ React.useEffect(() => {
+   ga("set", "page", "/users");
+   ga("send", "pageview");
+ }, []);

  return <h2>Users</h2>;
}

function AboutPage() {
+ React.useEffect(() => {
+   ga("set", "page", "/about");
+   ga("send", "pageview");
+ }, []);

  return <h2>About</h2>;
}
```

Not terrible, but there are some problems:

- The url paths ("/", "/users", "/about") should be kept in sync with the router
- The same code is repeated in multiple page components
- Developers might forget to add this when creating new pages

To avoid hardcoding the url paths, we can use the `window.location.pathname` api

```diff
function HomePage() {
  React.useEffect(() => {
-   ga("set", "page", "/");
+   ga("set", "page", window.location.pathname);
    ga("send", "pageview");
  }, []);

  return <h2>Home</h2>;
}

function UsersPage() {
  React.useEffect(() => {
-   ga("set", "page", "/users");
+   ga("set", "page", window.location.pathname);
    ga("send", "pageview");
  }, []);

  return <h2>Users</h2>;
}

function AboutPage() {
  React.useEffect(() => {
-   ga("set", "page", "/about");
+   ga("set", "page", window.location.pathname);
    ga("send", "pageview");
  }, []);

  return <h2>About</h2>;
}
```

To fix code duplication and centralize the pageview tracking, we can use the `history` object and listen to it for navigation events. When navigation occurs, we can send the pageview event.

```diff
import React from "react";
- import { BrowserRouter, Switch, Route, Link } from "react-router-dom";
+ import { BrowserRouter, Switch, Route, Link, useHistory } from "react-router-dom";

export default function App() {
  return (
    <BrowserRouter>
      <NavBar />
      <Routes />
    </BrowserRouter>
  );
}

function NavBar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/users">Users</Link>
      <Link to="/about">About</Link>
    </nav>
  );
}

function Routes() {
+  const history = useHistory();
+
+  React.useEffect(() => {
+    trackPageView(); // To track the first pageview upon load
+    history.listen(trackPageView); // To track the subsequent pageviews
+  }, [history]);
+
+  function trackPageView() {
+    ga("set", "page", window.location.pathname);
+    ga("send", "pageview");
+  }

  return (
    <Switch>
      <Route exact path="/" component={HomePage} />
      <Route exact path="/users" component={UsersPage} />
      <Route exact path="/about" component={AboutPage} />
    </Switch>
  );
}

function HomePage() {
-  React.useEffect(() => {
-    ga("set", "page", window.location.pathname);
-    ga("send", "pageview");
-  }, []);

  return <h2>Home</h2>;
}

function UsersPage() {
-  React.useEffect(() => {
-    ga("set", "page", window.location.pathname);
-    ga("send", "pageview");
-  }, []);

  return <h2>Users</h2>;
}

function AboutPage() {
-  React.useEffect(() => {
-    ga("set", "page", window.location.pathname);
-    ga("send", "pageview");
-  }, []);

  return <h2>About</h2>;
}
```

Now the pageviews for all routes can be tracked automatically without adding any tracking code in the page components! We can refine this further and extract the tracking logic in `Routes` component to a [custom hook](https://reactjs.org/docs/hooks-custom.html).

You can find a complete example in [this repo](https://github.com/sheshbabu/react-pageview-tracking-demo) and play with the [demo here](https://react-pageview-tracking-demo.netlify.app).

Note: The examples in this post use React Router 5.2.0
