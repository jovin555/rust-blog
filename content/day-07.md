---
title: "Day 07: Structs — Defining & Instantiating"
date: 2026-06-11
tags: [til, rust, structs, data-structures]
---

# Day 07: Structs — Defining & Instantiating

## What I Explored Today

Today I learned about **structs** (short for "structures"), Rust's way of creating custom data types. If you've used classes in other languages, structs will feel familiar — but with Rust's ownership rules baked in. Structs let me bundle related data together into a single, named type. Think of them as blueprints for creating objects that hold multiple pieces of information.

## Defining a Struct

A struct definition is just a blueprint. It says "every value of this type will have these fields with these types." Here's how to declare one:

```rust
// Define a struct named `User` with three fields
struct User {
    username: String,  // owned String, not &str
    email: String,     // also owned — more on why in a moment
    sign_in_count: u64,
    active: bool,
}
```

Notice I used `String` (owned) instead of `&str` (borrowed string slice). That's intentional: I want each `User` to *own* its data. If I used `&str`, Rust would force me to manage lifetimes (a topic for later), which adds complexity. For now, owned types inside structs are simpler and safer.

## Instantiating a Struct

To create an actual value from a struct blueprint, I specify field names and values. The order doesn't matter — Rust matches by name, not position:

```rust
fn main() {
    // Create an instance of User
    let user1 = User {
        email: String::from("alice@example.com"),
        username: String::from("alice123"),
        active: true,
        sign_in_count: 1,
    };

    // Access fields with dot notation
    println!("Email: {}", user1.email);
    println!("Username: {}", user1.username);
    println!("Active: {}", user1.active);
    println!("Sign-in count: {}", user1.sign_in_count);
}
```

Key insight: after creating `user1`, I can read its fields freely. But if I tried to *move* one of its fields (like `user1.email`), the whole struct would become partially invalid. Rust won't let me use `user1` after moving a field unless the field type implements `Copy`.

## Mutable Structs

If I want to change a field after creation, the entire struct instance must be declared `mut`. Rust doesn't support per-field mutability:

```rust
fn main() {
    let mut user2 = User {
        email: String::from("bob@example.com"),
        username: String::from("bob456"),
        active: false,
        sign_in_count: 0,
    };

    // This works because user2 is mutable
    user2.email = String::from("bob_new@example.com");
    user2.sign_in_count = 5;

    println!("Updated email: {}", user2.email); // bob_new@example.com
}
```

## Field Init Shorthand

When my variable names match struct field names, Rust lets me use a shorthand. This is a small but welcome ergonomic win:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,    // shorthand for email: email
        username, // shorthand for username: username
        active: true,
        sign_in_count: 1,
    }
}

fn main() {
    let user3 = build_user(
        String::from("carol@example.com"),
        String::from("carol789"),
    );
    println!("Built user: {}", user3.username);
}
```

The `email` and `username` fields automatically get the value of the variables with the same names. This saves typing and reduces visual noise.

## Struct Update Syntax

Often I want to create a new struct instance that's almost identical to an existing one, with just a few fields changed. Rust's `..` syntax (called struct update syntax) copies all remaining fields from another instance:

```rust
fn main() {
    let user1 = User {
        email: String::from("alice@example.com"),
        username: String::from("alice123"),
        active: true,
        sign_in_count: 1,
    };

    // Create user4 with a new email, but everything else from user1
    let user4 = User {
        email: String::from("alice_work@example.com"),
        ..user1  // fill remaining fields from user1
    };

    // user1 is now partially moved — we can't use user1.email or user1.username
    // But we CAN still use user1.active and user1.sign_in_count (they're Copy types)
    println!("user1 active: {}", user1.active); // OK — bool is Copy
    // println!("user1 email: {}", user1.email); // ERROR — String moved
}
```

This is where ownership gets interesting. The `..user1` syntax *moves* the `username` and `email` fields (owned `String`s) into `user4`. The `active` and `sign_in_count` fields are `Copy` types, so they're copied. After this, `user1` is partially invalid — I can still access its `Copy` fields, but not the moved ones.

## Tuple Structs

Rust also supports tuple structs — structs with unnamed fields that behave like tuples but have their own type:

```rust
// Tuple structs — fields have no names, just types
struct Color(u8, u8, u8);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);

    // Access by index like a tuple
    println!("Red channel: {}", black.0);
    println!("X coordinate: {}", origin.1);

    // Even though Color and Point have the same field types,
    // they are DIFFERENT types. This won't compile:
    // let mixed: Point = black; // ERROR: expected `Point`, found `Color`
}
```

The key benefit: `Color` and `Point` are distinct types even though they both contain three `u8` values. This prevents accidentally mixing them up.

## Unit-Like Structs

Finally, there are unit-like structs — structs with no fields at all. They're useful for implementing traits on types that don't need any data:

```rust
struct AlwaysEqual; // no fields — just a marker

fn main() {
    let marker = AlwaysEqual;
    // Useful for implementing traits (we'll see this later)
}
```

## Compiler Fight: Missing Fields

One of Rust's best features is that it won't let me create a struct instance without providing all required fields:

```rust
struct Book {
    title: String,
    author: String,
    pages: u32,
}

fn main() {
    // This will NOT compile
    let incomplete = Book {
        title: String::from("Rust in Action"),
        // forgot author and pages!
    };
}
```

The compiler error is crystal clear:

```
error[E0063]: missing fields `author` and `pages` in initializer of `Book`
 --> src/main.rs:10:22
  |
10 |     let incomplete = Book {
  |                      ^^^^ missing `author`, `pages`
```

This is fantastic — it catches incomplete data at compile time instead of letting `null` or undefined values sneak through at runtime.

## Try It Yourself

Create a struct called `Rectangle` with `width` and `height` fields (both `f64`). Write a function that takes a `Rectangle` and returns its area. Then create a few rectangles and print their areas.

```rust
struct Rectangle {
    width: f64,
    height: f64,
}

fn area(rect: &Rectangle) -> f64 {
    rect.width * rect.height
}

fn main() {
    let rect1 = Rectangle { width: 5.0, height: 3.0 };
    let rect2 = Rectangle { width: 10.0, height: 2.5 };

    println!("Area of rect1: {}", area(&rect1));
    println!("Area of rect2: {}", area(&rect2));
}
```

Notice I passed `&Rectangle` (a reference) to `area()` — this borrows the struct without taking ownership, so I can use `rect1` and `rect2` again after calling the function.

---
*Next up → **Day 08:** Methods on Structs — attaching behavior to data*