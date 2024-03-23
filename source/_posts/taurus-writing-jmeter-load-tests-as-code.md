---
title: "Taurus: Writing JMeter Load Tests As Code"
date: 2024-03-23 08:11:01
keywords: Load Testing, Performance, JMeter, Taurus
image: /images/2024-taurus-writing-jmeter-load-tests-as-code/2024-taurus-writing-jmeter-load-tests-as-code.png
tags:
  - Load Testing
  - Performance
---

JMeter is one of the [oldest](https://en.wikipedia.org/wiki/Apache_JMeter#Releases) and widely used tool for load testing applications. However it’s GUI based, and JMX the storage format it uses for test plans is not user friendly ([example file](https://jmeter.apache.org/demos/ForEachTest2.jmx)). This makes it hard to collaborate with other team members.

[Taurus](https://gettaurus.org/) is an open source tool that provides a friendly abstraction over JMeter. It allows one to write the test plans in a lightweight YAML format which are easier to read compared to JMX.

![](/images/2024-taurus-writing-jmeter-load-tests-as-code/2024-taurus-writing-jmeter-load-tests-as-code.png)

## Installation

Follow the installation steps from Taurus [here](https://gettaurus.org/install/Installation/)

## Simple test plan

Let’s come up with a very simple test plan to familiarize ourselves with Taurus.

```yaml
scenarios:
  home:
    requests:
      - url: http://localhost:8000/home

execution:
  - scenario: home
    concurrency: 1
    iterations: 10

```
This will make a HTTP GET request to the endpoint `http://localhost:8000/home` for 10 times.

Save the above as `main.yaml` and run using `bzt main.yaml`. For learning purposes, you can create a dummy server by running `python3 -m http.server` and create dummy "endpoints" using `touch home account`

In real world applications, we'll obviously make more than one request. We can specify sequential requests as follows:

```diff
  scenarios:
    home:
      requests:
        - url: http://localhost:8000/home
+       - url: http://localhost:8000/account

  execution:
    - scenario: home
      concurrency: 1
      iterations: 10

```

Let's also add a POST request:

```diff
  scenarios:
    home:
      requests:
        - url: http://localhost:8000/home
        - url: http://localhost:8000/account
+       - url: http://localhost:8000/login
+         method: POST
+         body: '{ username:"user", password:"hunter2" }'

  execution:
    - scenario: home
      concurrency: 1
      iterations: 10

```

Then add some headers and cookies:

```diff
  scenarios:
    home:
+     headers:
+       Cookie: 'connect.sid=abc'
+       authorization: Bearer 123
      requests:
        - url: http://localhost:8000/home
        - url: http://localhost:8000/account
        - url: http://localhost:8000/login
          method: POST
          body: '{ username:"user", password:"hunter2" }'

  execution:
    - scenario: home
      concurrency: 1
      iterations: 10
```

More examples [here](https://gettaurus.org/docs/JMeter/#Requests).

## Parameterizing values

We can pass values from command line to the test plan using "JMeter properties". Say we want to configure the `iterations` from command line, we can run the test using:

```shell
bzt main.yaml -o modules.jmeter.properties.iterations=20
```

and change our script to use this parameter:

```diff
  scenarios:
    home:
      requests:
        - url: http://localhost:8000/home
        - url: http://localhost:8000/account

  execution:
    - scenario: home
      concurrency: 1
-     iterations: 10
+     iterations: ${__P(iterations)}

```

Multiple parameters can be passed this way. More examples and configuration options [here](https://gettaurus.org/docs/JMeter/#JMeter-Properties-and-Variables).

## Splitting scenarios into multiple files

As your application becomes more complex, your scenarios would also become longer. At some point, you’d need to split your scenarios into multiple files. Let’s move the `home` scenario to a new file `home.yaml` and import it into `main.yaml`

```diff
+ included-configs:
+   - home.yaml

- scenarios:
-   home:
-     requests:
-       - url: http://localhost:8000/home
-       - url: http://localhost:8000/account

  execution:
    - scenario: home
      concurrency: 1
      iterations: 10
```

```yaml
# home.yaml

scenarios:
  home:
    requests:
      - url: http://localhost:8000/home
      - url: http://localhost:8000/account
```

## Using data from CSV file

Sometimes you would need to test your endpoints with multiple inputs. The `data sources` feature comes in handy those times. You can save your inputs as a CSV file and refer them in test file.

```diff
  # home.yaml

  scenarios:
    home:
+     data-sources:
+       - path: customers.csv
+         random-order: false
      requests:
        - url: http://localhost:8000/home
-       - url: http://localhost:8000/account
+       - url: http://localhost:8000/account?customer_id=${customer_id}&customer_name=${customer_name}
```
Contents of `customers.csv`:

```csv
customer_id,customer_name
1,John Doe
2,Jane Smith
3,Michael Johnson
```

As you can see above, the columns in the CSV file get converted to variables. These variables are initialized with values from one row of the file for each request and can be used in urls, headers etc. More options [here](https://gettaurus.org/docs/DataSources/).

## Using scripts for processing requests

Sometimes you need to pass the output of an endpoint into the input of a subsequent endpoint. This can’t be done using YAML, so we need to rely on scripting languages like JavaScript, Groovy etc. 

```yaml
# home.yaml

scenarios:
  home:
    requests:
      - url: http://localhost:8000/home
      - url: http://localhost:8000/account
      - url: http://localhost:8000/user
        jsr223:
          - language: javascript
            execute: after
            script-text: |
              var response = JSON.parse(prev.getResponseDataAsString());
              var userId = response.userId;
              print(userId);
              vars.put("user-id", userId);
      - url: http://localhost:8000/log
        method: POST
        headers:
          Content-Type: application/json
        jsr223:
          - language: javascript
            execute: before
            script-text: |
              var requestBody = {
                "userId": vars.get("user-id")
              };
              vars.put("request-body", JSON.stringify(requestBody));
        body: ${request-body}
```

In this example, we add a script that executes after the `/user` request to extract the `userId` and use it before the `/log` request to construct a payload.

Notice how I used `print()` instead of `console.log()` for debugging. This is because the [JavaScript used in JMeter](https://www.oracle.com/technical-resources/articles/java/jf14-nashorn.html) is different from the ones used in Web Browsers or NodeJS, so some keywords or APIs would not work. Also, JMeter developers recommend using [Groovy instead of JavaScript](https://jmeter.apache.org/usermanual/best-practices.html#jsr223) if you’re doing heavy load tests.