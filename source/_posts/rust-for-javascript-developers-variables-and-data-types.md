---
title: Rust for JavaScript Developers - Variables and Data Types
description: Rust for JavaScript Developers - Variables and Data Types
keywords: Rust, JavaScript, String, &str, Option, Vector
date: 2020-07-02 22:39:59
tags:
  - Rust
  - JavaScript
---

This is the second part in a series about introducing the Rust language to JavaScript developers. Here's the previous chapter:

- [Tooling Ecosystem Overview](http://www.sheshbabu.com/posts/rust-for-javascript-developers-tooling-ecosystem-overview/)

## Variables

JavaScript has three ways to declare variables - `var`, `const` and `let`. Rust also has `const` and `let` but they work very differently compared to JavaScript.

### let

In Rust, we declare variables using `let`.

```rust
let a = 123;
```

As expected, this assigns the value `123` to the variable `a`. But Rust by default makes the variable immutable. You won’t be able to change the value after it’s assigned.

```rust
let a = 123;
a = 456; // Error! :(
```

At first glance, this might look similar to JavaScript’s const but JavaScript’s const doesn’t make variables immutable, it just makes the [binding immutable](https://ponyfoo.com/articles/const-variables-not-immutable).

If you want to make variable mutable, you need to explicitly mention it using the `mut` keyword

```rust
let mut a = 123;
a = 456; // No Error :)
```

### const

Rust’s `const` is also very different from JavaScript’s const - it’s better to think of Rust’s const as a "label" to a constant value. During compile time they get replaced by their actual values in all the places they are used. It’s usually used for constants like port numbers, timeout values, error codes etc.

You can also define const outside functions at global level which can’t be done using let.

## Data Types

### Numbers

In JavaScript, we’ve the [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) type for both integers (numbers without decimal point) and floats (numbers with decimal point). In Rust, there’s a ton of options for [integers](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types) and [floats](https://doc.rust-lang.org/book/ch03-02-data-types.html#floating-point-types) but by default we can use `i32` for integers and `f64` for floats.

```rust
let x = 123; // i32
let y = 4.5; // f64
```

### Booleans

Pretty straightforward - both JavaScript and Rust have booleans with true/false values

```rust
let x = false; // bool
```

### Strings

We usually don’t think much about strings when working with JavaScript - they "just work". In Rust, there are many types of strings but let’s focus on the widely used ones - `String` and `&str`.

`String` is growable whereas `&str` is immutable and fixed size.

When you create a string using a string literal, it creates a `&str` type:

```rust
let name = "Saitama"; // &str
```

You need to use `String::from` or `to_string` methods to create a `String` type:

```rust
let name  = String::from("Genos"); // String
let name2 = "King".to_string();    // String
```

You can convert from `String` to `&str` using the `as_str` function

```rust
let name2 = "King".to_string(); // String
let name3 = name2.as_str();     // &str
```

We’ll learn more about strings in future posts.

### Optionals

JavaScript has two types for empty values - `undefined` and `null`. Undefined is used when a variable, property etc is not defined and null is used when something is intentionally empty.

Rust has neither of these - it doesn’t even have a dedicated null data type. Instead it has something call `Option`. When we have a situation where a value can be empty or initially undefined, this `Option` type is used.

People who have worked with TypeScript/Flow might see some similarities here but it’s quite different in terms of how the optionals are created and used.

Say we want to write a function that takes in a file path and returns its contents. Let’s say for whatever reason, we want to return a "null" value when empty string is passed as file path.

Here’s how we would write this in JavaScript/TypeScript:

```typescript
function read_file(path: string): string | null {
  const contents = "hello";

  if (path !== "") {
    return contents;
  }

  return null;
}
```

Implementing the same using Rust's `Option`:

```rust
fn read_file(path: &str) -> Option<&str> {
  let contents = "hello";

  if path != "" {
    return Some(contents);
  }

  return None;
}
```

You can see that we return `None` for null value but for non-null value, we don’t return the `contents` as it is, but rather we "wrap" it inside `Some` and return that. The return type is also not "string or null" as per the TypeScript example but "Option that contains &str" type.

Here’s how you would call this function in JavaScript/TypeScript:

```javascript
function main() {
  const file_contents = read_file("/path/to/file");

  if (file_contents !== null) {
    console.log(file_contents); // file_contents is refined to string type
  } else {
    console.log("Empty!");
  }
}
```

Calling the function in Rust:

```rust
fn main() {
  let file = read_file("path/to/file");

  if file.is_some() {
    let contents = file.unwrap();
    println!("{}", contents);
  } else {
    println!("Empty!");
  }
}
```

As you can see, we need to manually "unwrap" the Option to get the contents inside.

### Arrays

Similar to strings, there are two types of arrays - one with fixed size (simply referred to as "Array") and other that can grow/shrink in size (called "Vectors").

Arrays:

```rust
fn main() {
  let list = [1, 2, 3];
  println!("{:?}", list);
}
```

Vectors:

```rust
fn main() {
  let mut list = vec![1, 2, 3];
  list.push(4);
  println!("{:?}", list);
}
```

### Objects

Technically, all non-primitive types are "objects" in JavaScript but we commonly use the term "object" for two things - bag of data or hash map.

**Bag of data:**
Unlike other languages, you need not go through a lot of ceremony to create an object and this is one of the coolest things about JavaScript.

To create an employee object in JavaScript:

```javascript
function main() {
  const employee = {
    name: "Saitama",
    age: 25,
    occupation: "Hero",
  };
}
```

To create the same object in Rust, we can use structs:

```rust
struct Employee {
  name: String,
  age: i32,
  occupation: String,
}

fn main() {
  let employee = Employee {
    name: "Saitama".to_string(),
    age: 25,
    occupation: "Hero".to_string(),
  };
}
```

**HashMap:**

In JavaScript, to create an object with arbitrary key value pairs, we can either use normal object literals or the Map object:

```javascript
function main() {
  const colors = new Map();

  colors.set("white", "#fff");
  colors.set("black", "#000");

  console.log(colors.get("white")); // #fff
}
```

In Rust, you can do the same using the HashMap type:

```rust
use std::collections::HashMap;

fn main() {
  let mut colors = HashMap::new();

  colors.insert("white".to_string(), "#fff");
  colors.insert("black".to_string(), "#000");

  println!("{:?}", colors.get("white").unwrap()); // #fff
}
```

Notice the usage of unwrap above. HashMap’s get method returns an Option type which we need to unwrap to get the value inside.

Thanks for reading! :)
