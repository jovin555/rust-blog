---
title: "Day 09: Enums & Pattern Matching with match"
date: 2026-06-13
tags: [til, rust, enums, pattern-matching, control-flow]
---

# Day 09: Enums & Pattern Matching with `match`

## What I Explored Today

Today I unlocked one of Rust's most powerful features: **enums** (enumerations) combined with **pattern matching** via `match`. This is where Rust starts to feel _different_ from other languages I've used. In Python or JavaScript, I'd use `if/elif` chains or `switch` statements. In Rust, `match` is a first-class citizen that enforces exhaustiveness and can bind to the data inside an enum variant.

I learned that enums in Rust aren't just simple lists of values — each variant can carry its own data, making them a form of **algebraic data type** (like Haskell's or Scala's enums). Combined with `match`, they let me model complex states concisely and safely.

## Defining Enums with Data

Let's start with a simple enum that represents different kinds of IP addresses:

```rust
// Each variant can hold different types of data
enum IpAddr {
    V4(u8, u8, u8, u8),  // four octets
    V6(String),          // a single string
}
```

Here, `V4` holds four `u8` values, and `V6` holds a `String`. This is _way_ more ergonomic than creating separate structs for each variant.

Let's use it:

```rust
fn main() {
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));

    // We can't just print an enum — we need to pattern match!
    describe_address(&home);
    describe_address(&loopback);
}

fn describe_address(ip: &IpAddr) {
    // The `match` expression must be exhaustive — every variant must be handled
    match ip {
        IpAddr::V4(a, b, c, d) => {
            println!("IPv4: {}.{}.{}.{}", a, b, c, d);
        }
        IpAddr::V6(s) => {
            println!("IPv6: {}", s);
        }
    }
}
```

**Key insight**: The `match` arms destructure the enum variants, binding the inner data to variables (`a`, `b`, `c`, `d` for V4; `s` for V6). The compiler checks that every variant is covered — if I add a third variant to `IpAddr` later, the compiler will refuse to compile this code until I update the `match`.

## A More Realistic Example: Message System

Let's model a messaging system where different message types carry different data:

```rust
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },    // Named fields (like a struct)
    Write(String),              // A single string
    ChangeColor(i32, i32, i32), // Three integers
}
```

Now let's process each message type:

```rust
impl Message {
    fn process(&self) {
        match self {
            Message::Quit => {
                println!("Quitting...");
            }
            Message::Move { x, y } => {
                println!("Moving to ({}, {})", x, y);
            }
            Message::Write(text) => {
                println!("Writing message: {}", text);
            }
            Message::ChangeColor(r, g, b) => {
                println!("Changing color to RGB({}, {}, {})", r, g, b);
            }
        }
    }
}

fn main() {
    let messages = vec![
        Message::Write(String::from("Hello, world!")),
        Message::Move { x: 10, y: 20 },
        Message::ChangeColor(255, 0, 0),
        Message::Quit,
    ];

    for msg in &messages {
        msg.process();
    }
}
```

Each `match` arm pattern-matches the variant and extracts the data. Notice how `Move` uses curly braces (because it has named fields) while `Write` and `ChangeColor` use parentheses (tuple-like).

## The Compiler Fight: Non-Exhaustive Match

Here's the error I triggered when I first tried `match`:

```rust
// BAD: This won't compile!
fn process_incomplete(msg: &Message) {
    match msg {
        Message::Quit => println!("Quit"),
        // Missing: Move, Write, ChangeColor
    }
}
```

**Compiler says:**
```
error[E0004]: non-exhaustive patterns: `Move` not covered
  --> src/main.rs:10:11
   |
10 |     match msg {
   |           ^^^ pattern `Move` not covered
   |
   = help: ensure that all possible cases are being handled
```

The fix is either to handle every variant or use a **catch-all** pattern:

```rust
fn process_with_catch_all(msg: &Message) {
    match msg {
        Message::Quit => println!("Quit"),
        // `other` matches everything else — but we lose the variant's data
        other => println!("Some other message: {:?}", other),
    }
}
```

But wait — this won't compile either because `Message` doesn't implement `Debug`. Let's add `#[derive(Debug)]`:

```rust
#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

Now it works. But there's a subtlety: `other` binds the _entire_ enum value, not the inner data. If I only care about one specific variant and want to ignore the rest, I should use `_` (underscore):

```rust
fn process_ignore_rest(msg: &Message) {
    match msg {
        Message::Quit => println!("Quit"),
        _ => println!("Ignoring this message type"),
    }
}
```

The `_` pattern matches anything and doesn't bind the value. It's the idiomatic way to say "I don't care about the other variants."

## Pattern Matching with `if let`

Sometimes `match` feels verbose when I only care about one variant. Enter `if let` — a syntactic sugar for single-arm matches:

```rust
fn main() {
    let msg = Message::Write(String::from("Hello"));

    // Verbose match
    match &msg {
        Message::Write(text) => println!("Matched: {}", text),
        _ => (),  // unit — do nothing
    }

    // Concise if let
    if let Message::Write(text) = &msg {
        println!("Matched: {}", text);
    }
}
```

`if let` is perfect when you want to handle one case and ignore the rest. It's less safe than `match` (the compiler won't warn you if you forget variants), but much more readable for simple cases.

## Matching on Literals and Ranges

`match` isn't just for enums — it works with any expression:

```rust
fn classify_number(n: u32) -> &'static str {
    match n {
        0 => "zero",
        1 | 2 => "small",        // `|` means OR
        3..=5 => "medium",       // `..=` is an inclusive range
        6..=10 => "large",
        _ => "huge",             // catch-all
    }
}

fn main() {
    for i in 0..=12 {
        println!("{} is {}", i, classify_number(i));
    }
}
```

This prints:
```
0 is zero
1 is small
2 is small
3 is medium
4 is medium
5 is medium
6 is large
7 is large
8 is large
9 is large
10 is large
11 is huge
12 is huge
```

## Matching with Guards

For more complex conditions, I can add **match guards** — additional `if` conditions:

```rust
fn describe_point(x: i32, y: i32) {
    match (x, y) {
        (0, 0) => println!("Origin"),
        (0, y) => println!("On y-axis at y = {}", y),
        (x, 0) => println!("On x-axis at x = {}", x),
        (x, y) if x == y => println!("On diagonal: ({}, {})", x, y),
        (x, y) if x.abs() == y.abs() => println!("On anti-diagonal"),
        (x, y) => println!("At ({}, {})", x, y),
    }
}

fn main() {
    describe_point(0, 0);
    describe_point(3, 3);
    describe_point(1, -1);
    describe_point(5, 7);
}
```

Match guards let me express complex conditions without nesting `if` statements inside arms.

## Try It Yourself

Create an enum `TrafficLight` with variants `Red`, `Yellow`, and `Green`. Implement a method `next_light()` that returns the next light in the cycle (Red → Green → Yellow → Red). Then write a `match` expression that prints what a driver should do for each light color.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

impl TrafficLight {
    fn next_light(&self) -> TrafficLight {
        // Your code here
    }

    fn action(&self) -> &'static str {
        // Your code here
    }
}

fn main() {
    let mut light = TrafficLight::Red;
    for _ in 0..6 {
        println!("Light: {:?} — {}", /* your code */);
        light = light.next_light();
    }
}
```

**Bonus challenge**: Add a `Flashing` variant that takes a `Color` enum (with `Red` and `Yellow` variants). Update `next_light()` and `action()` to handle it.

---
*Next up → **Day 10:** `Option<T>` — Handling the Absence of a Value*