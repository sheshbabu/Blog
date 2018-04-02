---
title: Packaging Node.js code into cross platform executables
description: Cross compile Node.js code into static binaries for easier distribution
date: 2018-03-31 02:54:23
keywords: JavaScript, Nodejs
tags:
- JavaScript
- Node.js
---

If you're writing command line tools in Node.js, it can be hard to distribute them since users need to install Node.js in their machines before being able to use your tool. It would be a lot easier for users if we can package our app into a single executable file that they can download and run without installing anything extra.

We can use [pkg](https://github.com/zeit/pkg) to compile our code into a single executable file for multiple target platforms (Windows, Linux, Mac etc). 

Let's start with a simple example:

```js
// index.js
console.log("hello world");
```

This file prints "hello world" and exits. We can package this by running:

```shell
$ npx pkg index.js
```

This by default builds executables for three platforms - Windows, Linux and Mac:

```shell
$ ls -1
index-linux
index-macos
index-win.exe
index.js
```

The target platforms can be customized by using the `--targets` [flag](https://github.com/zeit/pkg#targets).

I was curious how much space these take up:

```shell
$ ls -lh
total 183496
-rwxr-xr-x  1 sheshbabu  staff    33M Mar 31 01:40 index-linux
-rwxr-xr-x  1 sheshbabu  staff    34M Mar 31 01:40 index-macos
-rw-r--r--  1 sheshbabu  staff    22M Mar 31 01:40 index-win.exe
-rw-r--r--  1 sheshbabu  staff    28B Mar 31 00:46 index.js
```

22-34MB feels like a bit too much for something that just prints "hello world". Looking around the internet, it seems we can use tools like [upx](https://upx.github.io/) to reduce the file size.
