---
title: Guidelines to choose a JavaScript library
date: 2016-06-12 13:38:45
tags:
- JavaScript
- Development
---

### How important is this?

Picking the right JavaScript library is very important, especially during the beginning of a project. If we're not careful with our decisions during the beginning, we will end up spending a lot of time later cleaning up the mess. The more tightly coupled the codebase and the dependency, the more careful we must be in selecting the right one. Even more so for frameworks - as our code practically lives inside them. Here are some of the things I look for in an external dependency:

### 1) How invested are the core contributors?
* Every opensource project has a couple of core contributors and sometimes a company behind it.
* Finding out how they're using the project would be a good indicator of their commitment. Are they using it in production and on revenue generating parts of the business? Example: Facebook is using React for Newsfeed, Instagram etc
* This does not always apply as not all opensource projects have a commercial entity backing them.

### 2) How widely is it used?
* If it is widely used by others, then you would have access to a lot of tutorials, discussions about best practices, how-tos, StackOverflow answers etc.
* Edge cases and bugs would have been detected early and bugfixes made.
* Widely used libraries/frameworks would also help in hiring as there would be a good number of developers with that experience. Also, a good number of developers would be interested in joining your company to gain experience.
* This can be found out by keeping your ear to the ground for what's going on in the JavaScript community.

### 3) How are breaking changes introduced?
* The Web moves very fast and breaking changes are inevitable. Breaking changes might be done for deprecating bad ways of doing things, remedying poor architectural decisions made in the past, performance optimizations, availability of new browser features etc.
* How they're introduced makes a lot of difference - is this done gradually and incrementally? Is this communicated in advance by showing deprecation warnings?
* Are any migration tools provided? Example: jQuery provides [jQuery migrate](https://github.com/jquery/jquery-migrate), React provides [React codemod](https://github.com/reactjs/react-codemod) scripts for migrating to newer versions etc.
* This ties into the "How invested are the core contributors?" question. If they're using it for important projects then they would be very careful and systematic with breaking changes.

### 4) How is the documentation?
* A good documentation makes the library easy to use and helps avoid wasting time.
* This depends on the project - libraries with simple and intuitive APIs can get away with minimal documentation whereas complicated libraries with hard to understand concepts need extensive documentation, examples, and tutorials.
* Depending on how tightly coupled the library and the codebase is going to be, go through the library's documentation to get a feel of it and try to build a quick proof-of-concept to see if all the current and future requirements could be easily implemented using it.

### 5) How actively is it being developed?
* Actively developed projects ensure that any new bugs are quickly fixed, new functionality added and PRs merged (all depending on priority and validity).
* This also depends on the project as some projects are just "done" with nothing more to be added or fixed.

### Conclusion

I understand that this is a very broad topic and there might be a lot more factors that need to be considered. I formed these guidelines based on my personal experience and learning from others.
