---
title: What Every Programmer Should Know About Load Testing
date: 2024-03-24 14:13:54
keywords: Load Testing, Performance, JMeter, Gatling, Locust, k6, Artillery
image: /images/2024-what-every-programmer-should-know-about-load-testing/2024-what-every-programmer-should-know-about-load-testing.png
tags:
  - Load Testing
  - Performance
---

Load testing is very important to prevent reputational or revenue loss. However, most developers struggle to write load tests. Even if they do load testing, they don’t know how to do it properly. So they confidently release the app to prod, and when it inevitably fails to support more than 7 concurrent users, they scratch their heads wondering why it wasn’t caught earlier.

I suspect the term “test” in “load test” is throwing developers off. Load tests in essence are about simulating production traffic and see how our application handles it. If it was named something like “production traffic simulation”, I feel people would be far more serious about it.

![](/images/2024-what-every-programmer-should-know-about-load-testing/2024-what-every-programmer-should-know-about-load-testing.png)

## What’s the plan?

Often the very first thing developers do is to pick their favourite load testing tool. It’s almost always written in the same language they’re familiar with - if they’re a Python developer, then [Locust](https://locust.io/), if they work with JVM languages, it’s usually [Gatling](https://docs.gatling.io/) or [JMeter](https://jmeter.apache.org/), if JavaScript, then [k6](https://k6.io/) or [Artillery](https://www.artillery.io/), and so on. Their very next step would be to hammer their endpoints with arbitrary number of concurrent users with no sense of pacing or think time. And this test would be done against an environment that doesn’t have the same resources (CPU, memory, scaling factor etc) as the final prod environment.

> Load tests in essence are about simulating production traffic and see how our application handles it

If you pause here and think about what’s happening, you’ll notice a couple of flaws. If load testing is simulating production traffic from real users, then we can’t just bombard our endpoints with random number of requests, can we? If we want to observe how our services deployed to production is going to handle the load, then shouldn’t the environment we’re testing against mirror the production in terms of resource (CPU, memory, scaling factor, warm cache, number of records in database etc) configs? The closer you get to modelling your expected user traffic and the resources you’ll have provisioned, the more accurate your test results are going to be. 

> The closer you get to modelling your expected user traffic and the resources you’ll have provisioned, the more accurate your test results are going to be.

Once you realize this, things start falling into place.

## How many users?

Sometimes you get lucky and you’ll know exactly how many people are going to use your application. For example, if you’re building a webapp for a conference, you can derive the max number of users from the max number of attendees, and then add some buffer on top.

Other times, you’ll have a sense based on past events. If you’re building an ecommerce application and targeting 11.11 or christmas sales, you’ll have an idea of traffic and usage patterns from past year. You’ll also have data on how well your company is doing and how much you’re spending on marketing compared to previous years. You can extrapolate your future traffic based on these numbers.

If you’re launching a new application or preparing for an event you’ve never done before, it’s useful to do a soft launch to a small number of internal users to get an idea of the usage patterns, how your infrastructure is handling this reduced load, and more importantly, getting a sense of the response latency your initial users are willing to put up with. You can also do a load test with a small number of users (5-10) if you’re not able to get internal users to use at the same time. This type of test is called “smoke test”. 

Another approach is to gradually increase the traffic in your load test until you reaching a breaking point. You start with a small number like 100 users and measure how your system is performing. Is the latency good ennough? Are you getting errors or crashes due to heavy load? How do the different services inside your system cope? Since different services scale differently, it’s useful to keep an eye on this. You then gradually increase this from 100 users to 1000 users and measure the same. You keep doing this until you start seeing degraded performance or errors. You’ll see some services/components being the bottleneck in your system. You can either tune them if possible or scale the resources available to that service (vertical or horizontal scaling). Rinse and repeat. The beauty of this approach is that you learn about the bottlenecks and the solutions that can fix these bottlenecks. When you ultimately release to prod and if there’s an unexpected amount of traffic, you’ll know exactly what services to scale instead of panicking.

## What scenarios?

Now that you’ve an idea of the number of (virtual) users, we need to come up with what they would be doing during the load test - pages they visit, actions performed, how long they linger in each page, etc.

Unless the application you’re load testing is a simple blog, you’d be aware of the different usage patterns of different types of users. For example, in an ecommerce app, anonymous users mainly search, look at deals and browse the products, compared to logged in users who might add items to cart, checkout or look at their past orders. You’ll need to look at your logs or analytics to find out more about the types of users and what actions they perform in your application.

![](/images/2024-what-every-programmer-should-know-about-load-testing/2024-what-every-programmer-should-know-about-load-testing-01.png)

After the above exercise, you’ll divide your virtual users into different user personas and create different user journeys so that your load test would closely simulate the real world. Once you’ve the different mapped out most of the user journeys (scenarios), you can prioritize few critical scenarios such as login, payment, or checkout, etc to write load tests for.

## What to measure?

After coming up with a realistic test plan and a traffic model, we need to come up with pass/fail metrics. 

- What is considered a good enough response time? Is it 300ms, 1s, 5s or more? This differs from application to application and even varies within an application.
- How many user sessions can your application support?
- What is throughput (requests per second) you’re aiming for?
- How many errors are you getting?
- How do the resources like CPU, Memory etc react to increased load?
- Does the auto scaling configuration work the way you want?
- If a component crashes during the load test, should that be considered a failure or is it a non-critical service that can be down during heavy load?

## Conclusion

To summarize, load testing needs a comprehensive view of your product. It’s something you need to discuss within your team and come up with a realistic plan and metrics to track. 

Aimlessly load testing helps no one, it actually gives your team false confidence that your application can indeed handle the load but would fail in production. If your application is critical for your company’s revenue or is high profile that it affects its reputation, then doing a proper load testing is a worthwhile undertaking.