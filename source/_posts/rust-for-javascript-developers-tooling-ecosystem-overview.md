---
title: Rust for JavaScript Developers - Tooling Ecosystem Overview
description: Rust for JavaScript Developers - Tooling Ecosystem Overview
keywords: Rust, JavaScript, Cargo, NPM
date: 2020-06-28 21:35:17
tags:
  - Rust
  - JavaScript
  - Rust Beginners
  - Rust for JavaScript Developers
---

This is the first part in a series about introducing the Rust language to JavaScript developers. Here are all the chapters:

1. [Tooling Ecosystem Overview](http://www.sheshbabu.com/posts/rust-for-javascript-developers-tooling-ecosystem-overview/)
2. [Variables and Data Types](http://www.sheshbabu.com/posts/rust-for-javascript-developers-variables-and-data-types/)
3. [Functions and Control Flow](http://www.sheshbabu.com/posts/rust-for-javascript-developers-functions-and-control-flow/)
4. [Pattern Matching and Enums](http://www.sheshbabu.com/posts/rust-for-javascript-developers-pattern-matching-and-enums/)

I find it easier to understand something new if it was explained in terms of something I already know - I thought there might be others like me :)

Here's the tl;dr version:

| Tool                             | JavaScript             | Rust             |
| -------------------------------- | ---------------------- | ---------------- |
| Version Manager                  | nvm                    | rustup           |
| Package Manager                  | npm                    | Cargo            |
| Package Registry                 | npmjs.com              | crates.io        |
| Package Manifest                 | package.json           | Cargo.toml       |
| Dependency Lockfile              | package-lock.json      | Cargo.lock       |
| Task Runner                      | npm scripts, gulp etc  | make, cargo-make |
| Live Reload                      | nodemon                | cargo-watch      |
| Linter                           | ESLint, TSLint, JSLint | Clippy           |
| Formatter                        | Prettier               | rustfmt          |
| Dependency Vulnerability Checker | npm audit              | cargo-audit      |

## Setup

Rust is installed using the [rustup](https://rustup.rs) command. rustup is similar to [nvm](https://github.com/nvm-sh/nvm) in Node.js. You can use it to install and manage multiple versions of Rust and more.

## Cargo

Installing Rust using rustup also installs Cargo similar to how installing Node.js also installs NPM. Cargo is Rust’s package manager and would feel very familiar if you’ve used NPM before.

Rust’s packages are called "crates", and they’re downloaded from the [crates.io](https://crates.io) registry similar to how NPM packages are downloaded from [npmjs.com](https://www.npmjs.com).

NPM while primarily a package manager, is also used as a task runner using the [npm scripts](https://docs.npmjs.com/misc/scripts) feature. Cargo has builtin support for common tasks like running code, building code etc. Cargo has features like [workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html) (similar to [lerna](https://lerna.js.org)), [dependency overrides](https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html) (similar to [patch-package](https://www.npmjs.com/package/patch-package)) out of the box. It is also a test runner (similar to mocha, jest etc), benchmark runner etc.

So basically, Cargo is NPM on steroids!

## Project Setup

Creating a new project is done by running

```shell
$ cargo new hello_rust
```

This is somewhat similar to `npm init`. This creates a directory called "hello_rust" with the following files:

```shell
hello_rust
├── .git
├── .gitignore
├── Cargo.toml
└─┬ src
  └── main.rs
```

## Cargo.toml

This is the package manifest file similar to package.json. The lock file (package-lock.json equivalent) is named Cargo.lock. Opening the Cargo.toml, you’ll see a familiar layout:

```toml
[package]
name = "hello_rust"
version = "0.1.0"
authors = ["sheshbabu"]
edition = "2018"

[dependencies]
```

The `[package]` [table](https://toml.io/en/v0.5.0#section-16) contains metadata like the crate name, author, keywords etc. The `[dependencies]` table is similar to the [dependencies object in package.json](https://docs.npmjs.com/files/package.json#dependencies). Cargo.toml also supports `[dev-dependencies]` similar to [devDependencies](https://docs.npmjs.com/files/package.json#devdependencies).

## Dependency Management

Installing a new dependency is done by manually editing the Cargo.toml file, adding the dependency under `[dependencies]` and running `cargo build`. For example, if we want to install the "serde" crate, we need to edit the Cargo.toml file as follows:

```diff
[package]
name = "hello_rust"
version = "0.1.0"
authors = ["sheshbabu"]
edition = "2018"

[dependencies]
+ serde = "1.0.106"
```

and run

```shell
$ cargo build
```

Similarly, to remove or update a dependency, we need to manually edit the Cargo.toml file and run `cargo build`. I was initially confused by the existence of the `cargo install` command but it turned out to be an equivalent of `npm install -g`.

If you want something similar to `npm install`, `npm update` or `npm uninstall`, you can install [cargo-edit](https://crates.io/crates/cargo-edit) which enhances Cargo with `cargo add`, `cargo rm` and `cargo upgrade` subcommands.

You can also specify the [dependency version using patterns](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#caret-requirements) similar to NPM.

## Development tools

### Task Runner

Cargo supports running common tasks like build, run, test etc. But if you want something similar to NPM scripts, you can use make or [cargo-make](https://crates.io/crates/cargo-make)

### Live Reload

Nodemon is an essential tool for Node.js development - it watches for changes to files and automatically restarts the application. [cargo-watch](https://crates.io/crates/cargo-watch) is the equivalent in Rust world.

### Linter and Formatter

Rust has builtin linter called [Clippy](https://github.com/rust-lang/rust-clippy) and formatter called [rustfmt](https://github.com/rust-lang/rustfmt). They’re equivalent to ESLint and Prettier in JS ecosystem. Precommit hooks can be managed using [cargo-husky](https://crates.io/crates/cargo-husky).

### Vulnerability Checking

Scanning for vulnerabilities in dependencies is done using [cargo-audit](https://crates.io/crates/cargo-audit) and is very similar to [npm audit](https://docs.npmjs.com/cli/audit).

Thanks for reading! Feel free to follow me in [Twitter](https://twitter.com/sheshbabu) for more posts like this :)
