---
title: Rust for JavaScript Developers - Pattern Matching and Enums
description: Part 4 covers pattern matching, destructuring, and enums
keywords: Rust, JavaScript, Pattern Matching, Destructuring, Enum, Option
image: /images/2020-rust-for-javascript-developers-4/image-1.png
date: 2020-07-12 13:48:03
tags:
  - Rust
  - JavaScript
  - Rust Beginners
  - Rust for JavaScript Developers
---

This is the fourth part in a series about introducing the Rust language to JavaScript developers. Here are all the chapters:

1. [Tooling Ecosystem Overview](http://www.sheshbabu.com/posts/rust-for-javascript-developers-tooling-ecosystem-overview/)
2. [Variables and Data Types](http://www.sheshbabu.com/posts/rust-for-javascript-developers-variables-and-data-types/)
3. [Functions and Control Flow](http://www.sheshbabu.com/posts/rust-for-javascript-developers-functions-and-control-flow/)
4. [Pattern Matching and Enums](http://www.sheshbabu.com/posts/rust-for-javascript-developers-pattern-matching-and-enums/)

## Pattern Matching

To understand Pattern Matching, let’s start with something familiar in JavaScript - Switch Case.

Here’s a simple example that uses `switch case` in JavaScript:

```javascript
function print_color(color) {
  switch (color) {
    case "rose":
      console.log("roses are red,");
      break;
    case "violet":
      console.log("violets are blue,");
      break;
    default:
      console.log("sugar is sweet, and so are you.");
  }
}

print_color("rose"); // roses are red,
print_color("violet"); // violets are blue,
print_color("you"); // sugar is sweet, and so are you.
```

Here’s the equivalent Rust code:

```rust
fn print_color(color: &str) {
  match color {
    "rose" => println!("roses are red,"),
    "violet" => println!("violets are blue,"),
    _ => println!("sugar is sweet, and so are you."),
  }
}

fn main() {
  print_color("rose"); // roses are red,
  print_color("violet"); // violets are blue,
  print_color("you"); // sugar is sweet, and so are you.
}
```

Most of the code should be immediately understandable. The `match` expression has the following signature:

```rust
match VALUE {
  PATTERN1 => EXPRESSION1,
  PATTERN2 => EXPRESSION2,
  PATTERN3 => EXPRESSION3,
}
```

The fat arrow `=>` syntax might trip us up because of the similarities with JavaScript arrow functions but they’re unrelated. The last pattern that uses underscore `_` is called the catchall pattern and is similar to the default case in switch case. Each `PATTERN => EXPRESSION` combination is called a `match arm`.

The above example doesn’t really convey how useful pattern matching is - it just looks like switch case with a different syntax and a fancy name. Let’s talk about destructuring and enums to understand why pattern matching is useful.

## Destructuring

Destructuring is the process of extracting the inner fields of an array or struct into separate variables. If you have used destructuring in JavaScript, it is very similar in Rust.

Here’s an example in JavaScript:

```javascript
let rgb = [96, 172, 57];
let [red, green, blue] = rgb;
console.log(red); // 96
console.log(green); // 172
console.log(blue); // 57

let person = { name: "shesh", city: "singapore" };
let { name, city } = person;
console.log(name); // name
console.log(city); // city
```

Here’s the same example in Rust:

```rust
struct Person {
  name: String,
  city: String,
}

fn main() {
  let rgb = [96, 172, 57];
  let [red, green, blue] = rgb;
  println!("{}", red); // 96
  println!("{}", green); // 172
  println!("{}", blue); // 57

  let person = Person {
    name: "shesh".to_string(),
    city: "singapore".to_string(),
  };
  let Person { name, city } = person;
  println!("{}", name); // name
  println!("{}", city); // city
}
```

## Comparing Structs

It’s very common to write "if this then that" type of code. Combining destructuring and pattern matching allows us to write these type of logic in a very concise way.

Let’s take this following example in JavaScript. It’s contrived but you have probably written something like this sometime in your career:

```javascript
const point = { x: 0, y: 30 };
const { x, y } = point;

if (x === 0 && y === 0) {
  console.log("both are zero");
} else if (x === 0) {
  console.log(`x is zero and y is ${y}`);
} else if (y === 0) {
  console.log(`x is ${x} and y is zero`);
} else {
  console.log(`x is ${x} and y is ${y}`);
}
```

Let’s write the same code in Rust using pattern matching:

```rust
struct Point {
  x: i32,
  y: i32,
}

fn main() {
  let point = Point { x: 10, y: 0 };

  match point {
    Point { x: 0, y: 0 } => println!("both are zero"),
    Point { x: 0, y } => println!("x is zero and y is {}", y),
    Point { x, y: 0 } => println!("x is {} and y is zero", x),
    Point { x, y } => println!("x is {} and y is {}", x, y),
  }
}
```

It’s a bit concise compared to the `if else` logic but also might be confusing as we’re performing comparison, destructuring and variable binding at the same time.

This is how it looks like visually:

![](/images/2020-rust-for-javascript-developers-4/pattern-matching-rust-1.png)
![](/images/2020-rust-for-javascript-developers-4/pattern-matching-rust-2.png)

We begin to see why it’s named as "pattern matching" - we take an input and see which pattern in the match arms "fits" better - It’s like the [shape sorter](https://www.google.com/search?q=shape+sorter) toys that kids play with. Apart from comparison, we also do variable binding in the 2nd, 3rd and 4th match arms. We pass variables x or y or both to their respective expressions.

Pattern matching is also `exhaustive` - that is, it forces you to handle all the possible cases. Try removing the last match arm and Rust won’t let you compile the code.

## Enum

JavaScript doesn’t have Enums but if you’ve used TypeScript, you can think of Rust’s Enums as a combination of TypeScript’s [Enums](https://www.typescriptlang.org/docs/handbook/enums.html) and TypeScript’s [Discriminated Unions](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions)

In the simplest case, Enums can be used as a group of constants.

For example, even though JavaScript doesn’t have Enums, you might have used this pattern:

```javascript
const DIRECTION = {
  FORWARD: "FORWARD",
  BACKWARD: "BACKWARD",
  LEFT: "LEFT",
  RIGHT: "RIGHT",
};

function move_drone(direction) {
  switch (direction) {
    case DIRECTION.FORWARD:
      console.log("Move Forward");
      break;
    case DIRECTION.BACKWARD:
      console.log("Move Backward");
      break;
    case DIRECTION.LEFT:
      console.log("Move Left");
      break;
    case DIRECTION.RIGHT:
      console.log("Move Right");
      break;
  }
}

move_drone(DIRECTION.FORWARD); // "Move Forward"
```

Here, we could’ve defined the FORWARD, BACKWARD, LEFT and RIGHT as separate constants, yet grouping it inside the DIRECTION object has the following benefits:

- The names FORWARD, BACKWARD, LEFT and RIGHT are namespaced under DIRECTION so naming conflicts can be avoided
- It is self-documenting as we can quickly see all the valid directions available in the codebase

However, there are some problems with this approach:

- What if someone passes NORTH or UP as an argument to move_drone? To fix this, we can add a validation to make sure that only values present in the DIRECTION object is allowed in the move function.
- What if we decide to support UP and DOWN in future or rename LEFT/RIGHT to PORT/STARBOARD? We need to find all the places where similar switch-case or if-else is used. There’s a chance that we might miss out a few places which would cause issues in production.

Enums in strongly typed languages like Rust are more powerful as they solve these problems without us writing extra code.

- If a function can take in only a small set of valid inputs, Enums can be used to enforce this constraint
- Enums with pattern matching force you to cover all cases. Useful when you’re updating Enums in future

Here’s the equivalent Rust example:

```rust
enum Direction {
  Forward,
  Backward,
  Left,
  Right,
}

fn move_drone(direction: Direction) {
  match direction {
    Direction::Forward => println!("Move Forward"),
    Direction::Backward => println!("Move Backward"),
    Direction::Left => println!("Move Left"),
    Direction::Right => println!("Move Right"),
  }
}

fn main() {
  move_drone(Direction::Forward);
}
```

We access the `variants` inside Enum using the `::` notation. Try editing this code by calling "move_drone(Direction::Up)" or adding "Down" as a new item in the Direction enum. In the first case, the compiler will throw an error saying that "Up" is not found in "Direction" and in the second case, the compiler will complain that we haven’t covered "Down" in the match block.

Rust Enums can do much more than act as a group of constants - we can also associate data with an Enum variant.

```rust
enum Direction {
  Forward,
  Backward,
  Left,
  Right,
}

enum Operation {
  PowerOn,
  PowerOff,
  Move(Direction),
  Rotate,
  TakePhoto { is_landscape: bool, zoom_level: i32 },
}

fn operate_drone(operation: Operation) {
  match operation {
    Operation::PowerOn => println!("Power On"),
    Operation::PowerOff => println!("Power Off"),
    Operation::Move(direction) => move_drone(direction),
    Operation::Rotate => println!("Rotate"),
    Operation::TakePhoto {
      is_landscape,
      zoom_level,
    } => println!("TakePhoto {}, {}", is_landscape, zoom_level),
  }
}

fn move_drone(direction: Direction) {
  match direction {
    Direction::Forward => println!("Move Forward"),
    Direction::Backward => println!("Move Backward"),
    Direction::Left => println!("Move Left"),
    Direction::Right => println!("Move Right"),
  }
}

fn main() {
  operate_drone(Operation::Move(Direction::Forward));
  operate_drone(Operation::TakePhoto {
    is_landscape: true,
    zoom_level: 10,
  })
}
```

Here, we’ve added one more Enum called Operation that contains "unit like" variants (PowerOn, PowerOff, Rotate) and "struct like" variants (Move, TakePhoto). Notice how we've used pattern matching with destructuring and variable binding.

If you’ve used TypeScript or Flow, this is similar to `discriminated unions` or `sum types`:

```typescript
interface PowerOn {
  kind: "PowerOn";
}

interface PowerOff {
  kind: "PowerOff";
}

type Direction = "Forward" | "Backward" | "Left" | "Right";

interface Move {
  kind: "Move";
  direction: Direction;
}

interface Rotate {
  kind: "Rotate";
}

interface TakePhoto {
  kind: "TakePhoto";
  is_landscape: boolean;
  zoom_level: number;
}

type Operation = PowerOn | PowerOff | Move | Rotate | TakePhoto;

function operate_drone(operation: Operation) {
  switch (operation.kind) {
    case "PowerOn":
      console.log("Power On");
      break;
    case "PowerOff":
      console.log("Power Off");
      break;
    case "Move":
      move_drone(operation.direction);
      break;
    case "Rotate":
      console.log("Rotate");
      break;
    case "TakePhoto":
      console.log(`TakePhoto ${operation.is_landscape}, ${operation.zoom_level}`);
      break;
  }
}

function move_drone(direction: Direction) {
  switch (direction) {
    case "Forward":
      console.log("Move Forward");
      break;
    case "Backward":
      console.log("Move Backward");
      break;
    case "Left":
      console.log("Move Left");
      break;
    case "Right":
      console.log("Move Right");
      break;
  }
}

operate_drone({
  kind: "Move",
  direction: "Forward",
});

operate_drone({
  kind: "TakePhoto",
  is_landscape: true,
  zoom_level: 10,
});
```

## Option

We learnt about the `Option` type in [chapter 2](http://www.sheshbabu.com/posts/rust-for-javascript-developers-variables-and-data-types/). Option is actually an Enum with two variants - `Some` and `None`:

```rust
enum Option<T> {
  Some(T),
  None,
}
```

This is how we handled the Option value in that chapter:

```rust
fn read_file(path: &str) -> Option<&str> {
  let contents = "hello";

  if path != "" {
    return Some(contents);
  }

  return None;
}

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

Using pattern matching, we can refactor the logic as follows:

```rust
fn main() {
  let file = read_file("path/to/file");

  match file {
    Some(contents) => println!("{}", contents),
    None => println!("Empty!"),
  }
}
```

Thanks for reading! Feel free to follow me in [Twitter](https://twitter.com/sheshbabu) for more posts like this :)
