---
title: "Day 05: References & Borrowing"
date: 2026-06-09
tags: [til, rust, ownership, references]
---

# Day 05: References & Borrowing

## What I Explored Today

Yesterday I learned that ownership means every value has exactly one owner, and when that owner goes out of scope, the value is dropped. That's powerful for memory safety, but it's also *incredibly* restrictive. What if I want to let a function look at my data without taking it away from me? What if I want to pass a value to multiple functions?

Today I learned about **references** and **borrowing** — Rust's solution to these problems. References let me *borrow* a value temporarily without taking ownership. The borrow checker then ensures I don't accidentally create data races or dangling pointers. Let's dig in.

## The Borrowing Concept

A **reference** is like a pointer that's guaranteed to point to a valid value. Creating a reference is called *borrowing* because you're temporarily using something that belongs to someone else.

```rust
fn main() {
    let s = String::from("hello");
    
    // `&s` creates a reference to `s` — we're borrowing it
    // `len` takes a reference, not ownership
    let length = calculate_length(&s);
    
    // `s` is still mine! I can use it here
    println!("The string '{}' has length {}", s, length);
}

// The function signature says: I want a reference to a String
// `&String` means "a reference to a String"
fn calculate_length(s: &String) -> usize {
    // s.len() works because we can read through a reference
    s.len()
    // When `s` (the reference) goes out of scope, nothing happens
    // to the original String — we never owned it
}
```

Key insight: `calculate_length` borrows `s` but doesn't own it. When the function ends, `s` (the reference) is dropped, but `s` (the original String) lives on in `main`. No ownership transfer, no double-free, no problem.

## References Are Immutable by Default

Just like variables, references are immutable by default. You can't modify something through a reference:

```rust
fn main() {
    let s = String::from("hello");
    change(&s);
}

fn change(some_string: &String) {
    // ERROR! Can't mutate through an immutable reference
    some_string.push_str(", world");
}
```

The compiler catches this:

```
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
```

If you *do* want to modify through a reference, you need a **mutable reference**:

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s); // prints "hello, world"
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Three things had to change:
1. `s` must be declared `mut`
2. We pass `&mut s` instead of `&s`
3. The function must accept `&mut String`

## The One Big Rule: No Aliasing + Mutation

Here's where Rust's borrow checker earns its reputation. Rust enforces **one rule** that prevents data races at compile time:

> At any given time, you can have *either* one mutable reference *or* any number of immutable references.

Let's see what happens when you break this:

```rust
fn main() {
    let mut s = String::from("hello");
    
    let r1 = &s;     // immutable reference — fine
    let r2 = &s;     // another immutable reference — also fine
    let r3 = &mut s; // mutable reference — COMPILER ERROR!
    
    println!("{}, {}, {}", r1, r2, r3);
}
```

The compiler says:

```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
```

Why? Because if `r1` and `r2` are reading `s` while `r3` could change it, we'd have a data race. Rust prevents this at compile time with zero runtime cost.

## Fighting the Compiler (and Winning)

Here's a mistake I made today — trying to use a reference after the original value was dropped:

```rust
fn main() {
    let r;
    {
        let s = String::from("hello");
        r = &s; // r borrows s
    } // s is dropped here
    println!("{}", r); // ERROR: s no longer exists!
}
```

The compiler catches this too:

```
error[E0597]: `s` does not live long enough
```

This is a **dangling reference** — a pointer to memory that's been freed. In C or C++, this would be undefined behavior. In Rust, it's a compile-time error. The borrow checker tracks *lifetimes* (a topic for later) to ensure references never outlive the data they point to.

## Try It Yourself

```rust
// 1. Fix this code by determining whether to use &, &mut, or something else
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let first = get_first(&numbers);
    numbers.push(6); // Is this allowed?
    println!("First: {}", first);
}

fn get_first(nums: &Vec<i32>) -> &i32 {
    &nums[0]
}
// Hint: Think about the borrow checker rule — can we mutate `numbers`
// while `first` is still borrowing it?

// 2. Try swapping the order: push first, *then* borrow
```

**Challenge:** Read the compiler error carefully. Can you restructure the code so that `first` is printed *after* the push? (You'll need to think about when references go out of scope.)

---
*Next up → **Day 06:** The Slice Type — borrowing without owning*