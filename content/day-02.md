---
title: "Day 02: Functions, Return Values & Expressions"
date: 2026-06-07
tags: [til, rust, functions, expressions, return-values]
---

# Day 02: Functions, Return Values & Expressions

## What I Explored Today

Yesterday I learned that `fn main()` is the entry point of every Rust program. Today I dove deeper into Rust's function system — how to define functions with parameters, return values, and the crucial distinction between **statements** (which do things) and **expressions** (which produce values). This distinction is one of Rust's most important concepts, and it took me a few examples to really understand.

## Anatomy of a Rust Function

Functions in Rust follow a clean, consistent pattern. Let me break it down with a heavily annotated example:

```rust
// `fn` keyword, function name, parameters with types, return type (->)
fn greet(name: &str) -> String {
    // Parameters must have their types annotated — Rust won't infer them here
    let greeting = format!("Hello, {}!", name);
    greeting // <-- no semicolon! This is an expression, it's returned
    // If I put a semicolon here, it becomes a statement and won't return
}

fn main() {
    let message = greet("Rust");
    println!("{}", message); // Prints: Hello, Rust!
}
```

Key observations:
- **Parameters**: Each parameter needs a type annotation (`name: &str`)
- **Return type**: Written with `->` followed by the type
- **Return value**: The last expression in the function body (no semicolon!) is returned automatically
- You can also use `return` keyword, but idiomatic Rust prefers the expression style

## Statements vs. Expressions — The Big Distinction

This is where Rust differs from many languages I've used before. The difference is subtle but powerful:

**Statements** perform actions and don't produce a value. They end with semicolons.
**Expressions** evaluate to a value. They don't end with semicolons (usually).

```rust
fn main() {
    // STATEMENT: assigns, doesn't produce a value
    let x = 5;
    
    // EXPRESSION: 5 + 3 evaluates to 8
    let y = {
        let a = 2;
        let b = 3;
        a + b // <-- no semicolon, this block is an expression evaluating to 5
    };
    println!("y = {}", y); // y = 5
    
    // EXPRESSION: if/else can be used as expressions!
    let condition = true;
    let number = if condition { 10 } else { 20 };
    println!("number = {}", number); // number = 10
}
```

Notice how the block `{ ... }` with `a + b` (no semicolon) evaluates to 5. This is incredibly powerful — blocks, if/else, and match statements can all be expressions.

## Multiple Return Paths

Functions can have multiple return points, but Rust's type system ensures they all return the same type:

```rust
fn describe_temperature(temp: f64) -> &'static str {
    // Note: &'static str is a string literal type — more on lifetimes later
    if temp > 30.0 {
        "Hot 🔥"
    } else if temp > 15.0 {
        "Warm ☀️"
    } else if temp > 0.0 {
        "Cool ❄️"
    } else {
        "Freezing 🥶"
    }
    // Every branch returns a &str — consistent types!
}

fn main() {
    println!("{}", describe_temperature(35.0)); // Hot 🔥
    println!("{}", describe_temperature(20.0)); // Warm ☀️
}
```

## Early Returns with `return`

Sometimes you need to exit early. The `return` keyword is there for that:

```rust
fn check_age(age: u32) -> String {
    // u32 means unsigned 32-bit integer (non-negative)
    
    if age == 0 {
        return "Not born yet!".to_string(); // Early exit
    }
    
    if age >= 21 {
        "Adult — welcome!".to_string() // Implicit return (expression)
    } else {
        "Too young!".to_string()
    }
}

fn main() {
    println!("{}", check_age(0));  // Not born yet!
    println!("{}", check_age(25)); // Adult — welcome!
    println!("{}", check_age(15)); // Too young!
}
```

## The Compiler Fight: Missing Return Type

Here's where the compiler caught me off guard:

```rust
// ❌ THIS WON'T COMPILE
fn add(x: i32, y: i32) {
    // I forgot to specify a return type!
    x + y
}

// The compiler says:
// error: expected `()`, found integer `i32`
// help: try adding a return type: `-> i32`
```

The fix is straightforward — specify the return type:

```rust
// ✅ THIS COMPILES
fn add(x: i32, y: i32) -> i32 {
    x + y
}
```

Another common mistake: accidentally making an expression into a statement:

```rust
// ❌ THIS WON'T COMPILE
fn double(x: i32) -> i32 {
    x * 2; // <-- semicolon! This is now a statement returning ()
}

// The compiler says:
// error: expected `i32`, found `()`
// help: remove this semicolon to return this value
```

## Functions Calling Functions

Functions can call other functions naturally. This is where Rust's function system starts to shine:

```rust
fn square(x: i32) -> i32 {
    x * x
}

fn sum_of_squares(a: i32, b: i32) -> i32 {
    square(a) + square(b) // Calling square twice!
}

fn main() {
    let result = sum_of_squares(3, 4);
    println!("3² + 4² = {}", result); // 9 + 16 = 25
}
```

## Try It Yourself

Write a function that takes three numbers and returns the largest one. Use an if/else expression inside the function. Here's a starting scaffold:

```rust
fn max_of_three(a: i32, b: i32, c: i32) -> i32 {
    // Your code here — use if/else as an expression
    // Hint: you might need nested if/else or combine conditions
}

fn main() {
    println!("Max of 5, 12, 3 is: {}", max_of_three(5, 12, 3));
    // Should print: Max of 5, 12, 3 is: 12
}
```

**Challenge**: Try doing it with just one expression (no semicolons inside the function body)!

---
*Next up → **Day 03:** Control Flow: if, loop, while, for*