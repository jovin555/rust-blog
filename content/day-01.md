---
title: "Day 01: Variables, Mutability & Basic Types"
date: 2026-06-07
tags: [til, basics]
---

# Day 01: Variables, Mutability & Basic Types

## What I Explored Today

Starting from scratch with Rust. Before you can do anything interesting, you need to understand how Rust thinks about variables — and it's already different from most languages.

---

## The Core Concept: Variables Are Immutable by Default

In most languages, variables are mutable by default. In Rust, it's the opposite.

```rust
fn main() {
    let x = 5;
    x = 6; // ❌ ERROR
}
```

**The Compiler Error:**

```
error[E0384]: cannot assign twice to immutable variable `x`
  --> src/main.rs:3:5
   |
2  |     let x = 5;
   |         - first assignment to `x`
3  |     x = 6;
   |     ^^^^^ cannot assign twice to immutable variable
```

**Why:** Rust enforces immutability by default so the compiler can guarantee that a value won't change unexpectedly. This eliminates an entire category of bugs — especially in concurrent code — before your program even runs.

**The Fix:** If you *want* a variable to change, you must explicitly say so with `mut`:

```rust
fn main() {
    let mut x = 5;
    println!("x is: {}", x);

    x = 6; // ✅ Works now
    println!("x is now: {}", x);
}
```

---

## Shadowing vs. Mutation

Rust has a second trick: **shadowing**. You can redeclare a variable with the same name using `let` again. This is *not* the same as mutation.

```rust
fn main() {
    let x = 5;
    let x = x + 1; // shadows the previous x
    let x = x * 2; // shadows again

    println!("x is: {}", x); // prints: x is 12
}
```

**Why this is useful:** Shadowing lets you reuse a name but *change the type*, which `mut` cannot do:

```rust
fn main() {
    let spaces = "   ";         // type: &str (a string)
    let spaces = spaces.len();  // type: usize (a number) ✅

    // With mut, this would fail:
    // let mut spaces = "   ";
    // spaces = spaces.len();  // ❌ type mismatch
}
```

---

## Basic Data Types

Rust is **statically typed** — every variable has a type known at compile time. The compiler usually infers it, but you can annotate explicitly.

### Integers

| Type | Size | Range |
|------|------|-------|
| `i8` | 8-bit | −128 to 127 |
| `u8` | 8-bit | 0 to 255 |
| `i32` | 32-bit | ~−2 billion to ~2 billion |
| `u32` | 32-bit | 0 to ~4 billion |
| `i64` | 64-bit | very large range |
| `usize` | arch-dependent | used for indexing |

The default integer type is `i32`. The `u` prefix means *unsigned* (no negatives).

```rust
fn main() {
    let age: u32 = 25;
    let temperature: i32 = -10;
    let big_number: i64 = 1_000_000; // underscores for readability
}
```

### Floats

```rust
fn main() {
    let pi = 3.14159;        // defaults to f64
    let small: f32 = 2.718;  // explicit f32
}
```

### Booleans

```rust
fn main() {
    let is_learning = true;
    let is_bored: bool = false;
}
```

### The Character Type

`char` in Rust is 4 bytes and represents a Unicode scalar value — not just ASCII.

```rust
fn main() {
    let letter = 'A';
    let emoji = '🦀'; // ✅ Rust's mascot is a valid char
}
```

---

## Constants vs. Variables

Constants use `const` instead of `let`, must have an explicit type annotation, and can never be `mut`. They live for the entire duration of the program.

```rust
const MAX_SCORE: u32 = 100_000;

fn main() {
    println!("Max score: {}", MAX_SCORE);
}
```

---

## Today's Mental Model

Think of Rust's default immutability as a safety contract:

> "Unless I explicitly said this can change, it won't — and the compiler will enforce that promise."

This feels restrictive at first, but it means you can trust your values. In larger programs, knowing that a variable can't change from under you eliminates an entire class of bugs.

---

## Try It Yourself

Paste this into the [Rust Playground](https://play.rust-lang.org/) and experiment:

```rust
fn main() {
    let name = "Rust";
    let mut version = 1;

    println!("{} edition {}", name, version);

    version += 1;
    println!("{} edition {}", name, version);

    // Try: remove `mut` from version and see the compiler error
    // Try: shadow `name` with a number type and see what happens
}
```

---

*Next up → **Day 02:** Functions, Return Values, and How Rust Handles Expressions*
