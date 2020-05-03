---
title: How to prevent code reviews from slowing down your team
description: How to prevent code reviews from slowing down your team
keywords: Code Review, Self Review, Development Velocity, Coding Guidelines, Style Guide
date: 2020-05-03 18:36:12
image: /images/2020-code-review-slow/image-1.png
tags:
  - Code Reviews
  - Development
---

Code reviews when not implemented properly can seriously slow down your team’s ability to ship - with many changes stuck in review for multiple days (or weeks!) hurting your product’s time to market. Here are a few reasons why your code reviews might be taking a long time:

- Not having coding guidelines
- Not using automated checks
- Not doing self reviews
- Raising huge PRs
- Raising vague PRs
- Not having deadlines for finishing reviews

## Not having coding guidelines

Every team should have a set of coding guidelines which everyone in the team agrees to. This should contain things like naming conventions, folder structure, code formatting and best practices like unit testing, validations etc.

Not having clear guidelines and conventions would have every developer writing code the way they like, resulting in a lot of arguments during code review. If you notice a lot of comments about formatting, naming conventions etc, you need to start a discussion regarding coding guidelines.

Your team can either come up with your own set of guidelines or you can start with established guidelines from other companies. Here’s an example from [Google](http://google.github.io/styleguide/)

## Not using automated checks

Once you have the coding guidelines documented, explore ways you can use tools to check for compliance. Almost all languages have formatters, linters etc that you can use in pre-commit hooks and CI/CD.

These tools would go through the code, check for coding guideline violations and notify the author about them. The author gets to fix these issues before raising the PR which greatly reduces the noise in the PR. The more checks your team offloads to automation the more time you get as a reviewer to focus on bigger issues like design flaws, implementation gaps etc

## Not doing self reviews

Before asking others to review, the author needs to review their own changes. This is called “self review” and it’s similar to proofreading your email for typos and mistakes before sending it to others.

In practice, reviewing your own code is challenging as you’re mentally blind to its shortcomings. Here are some ways you can do better self reviews:

- Raise the PR without assigning any reviewers and come back to it after a couple of hours so you can see your changes with a fresh set of eyes
- Fight the natural tendency to skim through your changes instead, go over your changes line-by-line deliberately
- Follow a checklist for self reviews - “Is the code following the coding guidelines?”, “Have I covered all the business requirements by these changes?”, “Have I written tests for all the possible use cases?” etc

## Raising huge PRs

The number of review comments a PR gets is inversely proportional to the number of changes it contains. That is, large PRs => few comments, small PRs => many comments

> The number of review comments a PR gets is inversely proportional to the number of changes it contains

This because reviewers get overwhelmed with large PRs and they might skim through the changes to wrap it up quickly. This defeats the purpose of code reviews. Sometimes, the opposite happens where a PR sits without any comments for many days because the reviewers are intimidated to start reviewing it.

Break down the PR into smaller chunks that make sense in isolation. You would get the reviewers to review it properly and promptly.

## Raising vague PRs

It’s common to get asked to review PRs that have vague or no description about what it does. The reviewer then has to make sense of the changes by trying to remember the task from standups or issue tracker etc. Add details such as:

- What changes does this PR contain?
- Which files should the reviewer start from?
- Link to the task in the issue tracker
- Screenshots if it’s a visual change

Adding these would give more context to the reviewer and thereby helping them to review your PRs faster.

## Not having deadlines for finishing reviews

One way for PRs to take forever to merge is to not have any deadlines for the reviewers to finish reviewing. Have reasonable deadlines such as:

- Reviews should be done within 48hrs from the time of raising the PR
- Reviews for hotfixes need to be done within 30mins

Thanks for reading! :)
