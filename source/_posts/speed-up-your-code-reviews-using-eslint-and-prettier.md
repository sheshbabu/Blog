---
title: Speed up your code reviews using ESLint and Prettier
description: Speed up your code reviews using ESLint and Prettier
keywords: JavaScript, Development, ESLint, Prettier, Lint Staged, Husky, Code Review
date: 2017-09-07 20:00:00
tags:
- Code Reviews
- JavaScript
- ESLint
- Prettier
- Development
---

Code reviews are very important if you want to build great software. It's an effective way of sharing knowledge of the codebase to other members of the team, it's a good opportunity to learn as the reviewers might suggest better ways of solving a problem than your usual approach, it helps in identifying logical bugs or gaps in implementation, it helps in ensuring that the codebase stays readable, maintainable and follows your team's coding conventions etc

Code reviews are also time-consuming. For reviewers, it requires them to go through the changes to look for issues and opportunities for improvement. The more things they're checking for, the more time consuming and less focussed they are. For the authors, once the review is over, they need to refactor the code as per the review comments, do additional testing, do self-review etc. Rinse and repeat.

We should strive to make this process faster so we can deliver the software to our users as quickly as possible. We can lessen the time it takes for reviewers to review the code by automating the code review as much as possible and letting them focus on the non-automatable aspects. For the authors, we can give feedback on the code early so they can refactor much earlier in the development process.

Checking whether the changes follow the coding conventions, best practices and code formatting is something we can automate. These are also the ones that trigger the most nitpicks during a code review and thereby generating more noise in the review comments.

[ESLint](https://eslint.org/) and [Prettier](https://github.com/prettier/prettier) are two popular tools that can help us achieve this. Prettier [formats](https://github.com/prettier/prettier#what-does-prettier-do) code and ESLint helps enforce coding conventions and [find problematic patterns](https://eslint.org/docs/rules/#best-practices) in code. ESLint also has an [auto-fix](https://eslint.org/docs/user-guide/command-line-interface#--fix) mode that automatically fixes some of the rule violations. Both [have](https://github.com/prettier/prettier#editor-integration) [plugins](https://eslint.org/docs/user-guide/integrations#editors) for all popular editors which ensures that the violations are quickly shown to the developer. But if the developer is using an editor that doesn't have these plugins or is someone who sporadically contributes code and you don't want to add friction to their workflow by asking them to install or configure the plugins, we can use the git commit hooks so that the code gets automatically checked as it is committed. Git commit hooks are also useful in making sure that all the committed code adheres to the rules and there are no [broken windows](https://blog.codinghorror.com/the-broken-window-theory/) due to misconfigured editors or other reasons. You can use [lint-staged](https://github.com/okonet/lint-staged) for easily setting up git commit hooks. 

If you're newly setting up a project and don't want to spend time initially to pick the rules or config, Prettier comes with good defaults and ESLint can be initialized with popular style guides.

If you want to introduce this to an existing project, you can run all the files through Prettier and use ESLint auto-fix to change the existing code as per the new rules. For the rules that are not covered by auto-fix, you can [disable](https://eslint.org/docs/user-guide/configuring#configuring-rules) all the remaining non-auto-fixable rules initially and fix them manually in batches and re-enable them as they're fixed. If it's a very larger project, you might want to split your codebase into different sections and have [directory specific ESLint configs](https://eslint.org/docs/user-guide/configuring#configuration-cascading-and-hierarchy) and make changes on one section at a time.


