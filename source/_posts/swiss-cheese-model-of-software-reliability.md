---
title: Swiss Cheese Model of Software Reliability
date: 2024-08-17 15:47:18
image: /images/2024-swiss-cheese-model-of-software-reliability/2024-swiss-cheese-model-of-software-reliability.png
keywords: software reliability, Swiss Cheese Model, layered bug prevention, software testing strategies, no silver bullet
tags:
  - Mental Model
  - Reliability
  - Development
  - Testing
  - Code Reviews
  - Maintainability
---

Software surrounds us. It's in the planes we fly and the cars we drive. It handles our finances, helps us learn, and keeps us connected. Despite being an integral part of our everyday lives, we still encounter software failures with regular frequency. Why is this? How do we eliminate software bugs?

In any complex system like the weather, ecology, or even the human body, there are countless variables, triggers, and participants that are interconnected. It's hard to accurately predict the future state of such a system. Most software, while not as complex as weather patterns or ecosystems, still presents similar challenges. There are so many ways code can behave that the more complex the software system becomes, the harder it is to ensure it's 100% bug-free.

How do we deal with this then? Should we just give up, saying that 100% bug-free software is not possible? Of course not! We try our best and use different approaches to reduce the number of bugs. But which approaches should we use?

![](/images/2024-swiss-cheese-model-of-software-reliability/2024-swiss-cheese-model-of-software-reliability.png)

## Silver Bullet Model

Here's where it gets confusing. Every developer, team, or devtools vendor would propose a different list of approaches. If you've been in this profession long enough, you might have encountered the heated debates between statically-typed and dynamically-typed languages, functional vs object-oriented programming models, and so on. 

As engineers, we think of ourselves as objective, but most of us wear the tools we use as our identities. If you work with statically-typed languages like Java, Go, and C, you criticize developers using JavaScript or Python, saying that using dynamically-typed languages is bad for reliability. Similarly, advocates of TDD say that if you're not following the TDD approach, your software is not good. 

Does that mean that software written in statically-typed languages or following TDD result in no bugs? Far from it. 

## Swiss Cheese Model
Fred Brooks, the author of *The Mythical Man-Month*, wrote a paper titled [No Silver Bullet — Essence and Accident in Software Engineering](https://www.cs.unc.edu/techreports/86-020.pdf), in which he says:

> There is no single development, in either technology or management technique, which by itself promises even one order of magnitude improvement in productivity, in reliability, in simplicity.

This was written almost four decades back, and it's still true! So, if there's no silver bullet, how do we make software more reliable? 

What we can do instead is to use a layered strategy, instead of relying on a single approach to increase software reliability.

This strategy stacks multiple approaches one after another. As the software passes through these layers, some bugs are caught in the initial layers, while the remaining bugs are trapped in subsequent layers. By the time the software reaches the end user, the number of bugs is significantly reduced.

This layered approach is commonly known as [Defense in Depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)) in cybersecurity or as [Swiss Cheese Model](https://en.wikipedia.org/wiki/Swiss_cheese_model) in disease prevention.

I particularly like the [Swiss cheese imagery](https://www.google.com/search?q=swiss+cheese+slice) because it's not only fun, but the holes in the cheese represent the ways in which these individual layers fail. 

Approaches like static typing, unit tests, linting, code reviews, canary deployment, or code coverage, are not 100% effective at catching bugs on their own. However, if we stack them together, bugs would be caught in one of these layers and would greatly improve the reliability of software that users use. 

## Conclusion

As we can see, relying on a single approach to prevent bugs is not a good idea. Adopting a layered strategy ensures that bugs are caught in any one of these layers and it also makes the process future-proof. If there are new innovations in future, like *AI Code Review* tools, we can add them as one of the layers, and thereby increasing the reliability even more.