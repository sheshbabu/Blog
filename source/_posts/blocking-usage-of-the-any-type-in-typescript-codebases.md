---
title: Blocking usage of the any type in TypeScript codebases
description: Blocking usage of the any type in TypeScript codebases
keywords: TypeScript, any, ESLint, typescript-eslint
date: 2020-05-05 23:32:06
image: /images/2020-typescript-any/image-1.png
tags:
  - TypeScript
  - ESLint
  - Development
---

The `any` type in TypeScript denotes that the value can be anything - string, number, boolean etc. If widely used in a new codebase, it would defeat the purpose of using TypeScript over untyped JavaScript. So it's preferable to block the usage of this type. If you're incrementally migrating a JavaScript codebase to TypeScript, it would be better to use the new [unknown](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) type.

## Blocking implicit any

```javascript
function main() {
  frobulate("hello");
}

function frobulate(name) {
  console.log(name);
}
```

The `name` argument in the above `frobulate` function has an implicit `any` type. Usually, TypeScript can infer types based on the control flow, assignments etc but in the above case, it's hard to infer the type for the `name` argument so it would implicitly have the `any` type.

To block the implicit `any`, we can set `noImplicitAny` or `strict` to true in `tsconfig.json`.

## Blocking explicit any

Setting `noImplicitAny` or `strict` to true would throw errors for the above code but if you explicitly type the `name` argument as `any`, it wouldn't throw any errors.

```javascript
function main() {
  frobulate("hello");
}

// Notice the "any" type added to the "name" argument
function frobulate(name: any) {
  console.log(name);
}
```

To block explicit `any`, we need to use [ESLint](https://eslint.org) with the `typescript-eslint` plugin and turn on the `no-explicit-any` rule. Here's an example `.eslintrc` config:

```json
{
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint"],
  "rules": {
    "@typescript-eslint/no-explicit-any": ["error"]
  }
}
```
