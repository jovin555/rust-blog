---
title: "Day 04: Ownership: The Core Mental Model"
date: 2026-06-08
tags: [til, rust, ownership, memory]
---

# Day 04: Ownership: The Core Mental Model

## What I Explored Today

Yesterday I learned about variables and mutability. Today I hit the concept that makes Rust truly unique: **ownership**. The Rust book calls this "Rust's most unique feature," and after today's exploration, I understand why. Ownership is the system that lets Rust guarantee memory safety without a garbage collector.

## The Three Rules of Ownership

Ownership in Rust follows three simple rules:

1. **Each value in Rust has exactly one owner** (the variable that holds it)
2. **There can only be one owner at a time**
3. **When the owner goes out of scope, the value is dropped** (memory is freed)

Let's see these rules in action.

## Ownership and Scope

```rust
fn main() {
    // Rule 1: s is the owner of the string "hello"
    let s = String::from("hello");
    
    // s is in scope here, so we can use it
    println!("{s}");  // hello
    
    // When this block ends, s goes out of scope
    // Rust automatically calls `drop()` to free the memory
} // <-- s is no longer valid here
```

Notice I used `String::from` instead of a string literal (`"hello"`). String literals are special—they're stored directly in the program binary. `String::from` allocates memory on the heap, which is where ownership really matters.

## Move: Transferring Ownership

Here's where Rust differs from many other languages:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;  // Ownership MOVES from s1 to s2
    
    // println!("{s1}"); // ERROR! s1 is no longer valid
    println!("{s2}");    // Works: s2 is the new owner
}
```

In Python or Java, `s2 = s1` would create a reference to the same object. Both variables would be usable. In Rust, assigning `s1` to `s2` **moves** ownership. After the move, `s1` is invalid.

Why does Rust do this? When `s1` owns a heap-allocated `String`, Rust could either:
- **Copy the data** (expensive for large strings)
- **Share the data** (but then who frees it when both go out of scope?)

Rust's solution: move ownership. Only one owner, only one `drop()` call. No double-free bugs.

## The Compiler Fight: Using a Moved Value

Here's the error you'll see constantly when learning ownership:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    
    // This line causes a compiler error:
    println!("{s1}");  // ERROR!
}
```

**Error:**
```
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:20
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |     
5 |     println!("{s1}");
  |                    ^^ value borrowed here after move
```

The fix? Either don't use `s1` after the move, or clone the data:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Deep copy: both are valid
    
    println!("{s1}");  // Works!
    println!("{s2}");  // Works!
}
```

**Important:** `clone()` copies the heap data, which is expensive. Only clone when you truly need both copies.

## Ownership and Functions

Ownership applies to function calls too. Passing a value to a function **moves** ownership:

```rust
fn main() {
    let s = String::from("hello");
    
    take_ownership(s);  // s's ownership moves into the function
    
    // println!("{s}"); // ERROR: s is no longer valid
} // s was already moved, nothing to drop here

fn take_ownership(some_string: String) {
    println!("{some_string}");
} // some_string goes out of scope, memory is freed
```

To get the value back, you need to return it:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = give_and_take_back(s1);  // Ownership moves in, then back out
    
    println!("{s2}");  // Works!
} // s2 goes out of scope, memory freed

fn give_and_take_back(s: String) -> String {
    println!("Inside function: {s}");
    s  // Return ownership back to caller
}
```

## Stack-Only Data: The Copy Trait

Simple types like integers don't have this problem:

```rust
fn main() {
    let x = 5;
    let y = x;  // x is COPIED, not moved
    
    println!("{x}");  // Works!
    println!("{y}");  // Also works!
}
```

Types that implement the `Copy` trait are copied instead of moved. These are types whose size is known at compile time and stored entirely on the stack:
- All integer types (`i32`, `u64`, etc.)
- Boolean (`bool`)
- Floating point (`f64`, etc.)
- Character (`char`)
- Tuples containing only `Copy` types (like `(i32, i32)`)

## The Mental Model

Here's how I think about ownership now:

**Every value has exactly one owner.** When I assign or pass a value, I'm either:
- **Moving** it (for heap-allocated types like `String`)
- **Copying** it (for stack-only types like `i32`)

The compiler enforces this at compile time, so there's zero runtime overhead. No garbage collector, no reference counting, just compile-time checks.

## Try It Yourself

```rust
fn main() {
    let favorite = String::from("Rust");
    let greeting = create_greeting(favorite);
    
    // Try uncommenting this line:
    // println!("I love {favorite}!");
    
    println!("{greeting}");
}

fn create_greeting(name: String) -> String {
    let greeting = format!("Hello, {name}!");
    greeting  // Return ownership
}
```

**Challenge:** Modify the code so you can print both `favorite` and `greeting` in `main()`.

---
*Next up → **Day 05:** References & Borrowing — using values without taking ownership*