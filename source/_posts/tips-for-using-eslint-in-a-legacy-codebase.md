---
title: Tips for using ESLint in a legacy codebase
description: Tips for using ESLint in a legacy codebase
date: 2018-02-25 02:28:07
keywords: JavaScript, ESLint
tags:
- JavaScript
- ESLint
- Development
---

ESLint is a fantastic tool that helps in detecting problematic patterns in codebase and enforcing coding conventions. It's a must-have tool for any team trying to maintain high quality JavaScript codebases. 

However, when you're introducing ESLint to a legacy codebase for the first time, it's hard not to feel overwhelmed by the bajillion errors it throws. 

```shell
$ eslint .

✖ 20983 problems (20983 errors, 0 warnings)
  19032 errors, 0 warnings potentially fixable with the `--fix` option.
```

It's understandable that the old code was written without the ESLint rules in mind but it becomes a problem when errors in any new code written gets drowned in all that noise. In this blog post, I'll go through some techniques that can help you significantly reduce the amount of errors you see.

## Autofix

ESLint has this very useful [utility](https://eslint.org/docs/user-guide/command-line-interface#--fix) that automatically fixes most of the errors. What can be and cannot be fixed depends on the rules themselves. The impact of this on reducing the number of errors depends on the codebase and the config used. If you notice in the above shell snippet, out of `20983` errors, `19032` can be automatically fixed! Let's do that:

```shell
$ eslint --fix .

✖ 1700 problems (1700 errors, 0 warnings)
```

`1700` errors sounds a lot more manageable than `20983`!

## Specifying environments and globals

Each JS environment (Browser, Nodejs, etc) has it's own set of host/global objects (window, process, setTimeout etc) in addition to native JS objects (Date, parseInt etc). ESLint doesn't assume an environment, so you might see errors like `'window' is not defined` or `'setTimeout' is not defined `. By [specifying environments](https://eslint.org/docs/user-guide/configuring#specifying-environments), these errors can be fixed.

Up until a few years back, the most common way of adding dependencies to a web codebase was to use `<script>` tags and they expose their apis using global variables like `$` for jQuery, `_` for underscore etc. You might see errors like `'$' is not defined` or `'moment' is not defined`. These errors can be fixed by [specifying globals](https://eslint.org/docs/user-guide/configuring#specifying-globals).

## Disabling rules

This should be the last resort and before we start disabling rules left and right, let's go through the different levels in which we can disable rules:

#### Line level
This disables rules for a line
```js
alert('foo'); // eslint-disable-line no-alert, quotes, semi

// eslint-disable-next-line no-alert, quotes, semi
alert('foo');
```
#### Block level
This disables rules for multiple lines
```js
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */
```
#### File level
This disables rules for the whole file
```js
/* eslint-disable no-alert */

alert('foo');
```
#### Directory level
This disables rules for a directory. This is done by creating a new config file inside that directory. There can be multiple ESLint config files inside a project. The closest config file would override the config files defined in outer directories.

This feature can be useful in situations where you need to disable some rules only a particular directory but still need those rules to be enabled for the rest of the codebase. For example, if you have [no-magic-numbers](https://eslint.org/docs/rules/no-magic-numbers) rule enabled and see a lot of errors from `tests/` directory because you're using magic numbers in assertions, you can just disable this rule in a config file created inside the `tests/` directory.

Learn more about this feature [here](https://eslint.org/docs/user-guide/configuring#configuration-cascading-and-hierarchy).
#### Project level
You can disable rules for the whole codebase if you disable them in the root config file. Usually this approach is used when you're using a [sharable config](https://eslint.org/docs/user-guide/configuring#using-a-shareable-configuration-package) like [eslint-config-airbnb](https://www.npmjs.com/package/eslint-config-airbnb) or [eslint-config-standard](https://www.npmjs.com/package/eslint-config-standard) etc and want to add your own overrides.

#### Thoughts on disabling rules

It's very easy to disable rules at the project level but doing so would mean that any new code written won't be checked against those rules. 

Disabling at line level is best as only those lines are affected but this can be very tedious as we might need to do this for thousands of lines.

Depending on the number of files that have errors, it's better to disable at a file or directory level. This is easier than disabling at line level and the impact on new code is also smaller than disabling at a project level. 

It would be nice to have a tool that goes through the codebase and adds the `eslint-disable-line` comment for the affected lines, but I don't know if such tool exists. 

## Conclusion

Using the steps above, all the errors in the codebase can be fixed or disabled. Any new errors would be only because of new code written and they can be fixed as soon as we see them. We can allocate some time depending on the bandwidth available to go through the disabled errors and fix them incrementally.

Thanks for reading! If you liked this, you might also like: [Speed up your code reviews using ESLint and Prettier](http://www.sheshbabu.com/posts/speed-up-your-code-reviews-using-eslint-and-prettier/)

