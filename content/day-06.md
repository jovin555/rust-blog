---
title: "Day 06: The Slice Type"
date: 2026-06-10
tags: [til, rust, slices, borrowing]
---

# Day 06: The Slice Type

## What I Explored Today

Yesterday I learned how references let us borrow values without taking ownership. Today I discovered **slices**—a special kind of reference that doesn't point to a single value, but to a *contiguous sequence* of elements in a collection. Slices are one of Rust's most elegant features because they give us safe, efficient access to parts of an array, String, or vector without copying data.

## Slices: Borrowing a View Into Data

A slice is a **dynamically-sized view** into a sequence of elements. The key insight: slices are always *borrowed*—they never take ownership of the underlying data. This means slicing is cheap (no memory allocation) and safe (the borrow checker ensures the original data lives long enough).

### String Slices

The most common slice is a `&str` (pronounced "string slice"), which borrows a portion of a `String`:

```rust
fn main() {
    let full_name = String::from("Ada Lovelace");
    
    // Slice syntax: &variable[start..end]
    // This borrows characters from index 0 (inclusive) to 3 (exclusive)
    let first_name = &full_name[0..3];  // "Ada"
    
    // Rust allows shorthand when slicing from start or to end
    let last_name = &full_name[4..];    // "Lovelace" (from index 4 to end)
    let whole = &full_name[..];         // "Ada Lovelace" (entire string)
    
    println!("First: {first_name}, Last: {last_name}, Whole: {whole}");
}
```

**Why this matters:** The slice `&full_name[0..3]` doesn't copy "Ada" into a new String. It's just a pointer to the original String's memory, plus a length. Zero-copy access—very efficient!

### Important: String Slices Work on Bytes, Not Characters

Rust strings are UTF-8 encoded, so slice indices must fall on **character boundaries**:

```rust
fn main() {
    let greeting = String::from("Hello!");
    // This works: 'H' is 1 byte, so index 0 is valid
    let h = &greeting[0..1];
    
    let emoji = String::from("😊");
    // This will PANIC at runtime! 😊 is 4 bytes, so index 1 is in the middle
    // let broken = &emoji[0..1];  // UNCOMMENTING THIS CRASHES
    
    // Correct way: use full range for multi-byte characters
    let safe = &emoji[0..4];  // Works: 4 bytes = 1 complete character
    println!("{safe}");
}
```

**Compiler lesson:** Always slice strings with care. For general string manipulation, prefer methods like `.chars()` or `split_at()` that respect UTF-8 boundaries.

## Arrays Have Slices Too

Slices work with any sequential collection, including arrays:

```rust
fn main() {
    let numbers = [10, 20, 30, 40, 50];
    
    // Slice of array: type is &[i32]
    let mid_three: &[i32] = &numbers[1..4];  // [20, 30, 40]
    
    println!("Middle three: {:?}", mid_three);
    
    // Slices are great for function parameters
    print_first_two(&numbers[..2]);  // Pass only the first two
}

fn print_first_two(slice: &[i32]) {
    // This function accepts ANY slice of i32 values
    println!("First two: {} and {}", slice[0], slice[1]);
}
```

**Key insight:** `&[i32]` as a parameter type is incredibly flexible—the function can accept a slice of any length from any array, vector, or even another slice.

## The Real Power: Writing Slice-Aware Functions

Before slices, if you wanted a function that works with both `String` and `&str`, you'd need two versions. Slices solve this elegantly:

```rust
fn main() {
    let my_string = String::from("hello world");
    let my_str = "hello world";  // This is already a &str
    
    // Same function works with both!
    let word1 = first_word(&my_string[..]);  // Slice of String
    let word2 = first_word(my_str);          // &str directly
    let word3 = first_word(&my_string);      // String auto-derefs to &str!
    
    println!("Words: {word1}, {word2}, {word3}");
}

// Best practice: take &str as parameter for flexibility
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' ' {  // Found a space? Return slice up to it
            return &s[0..i];
        }
    }
    
    &s[..]  // No space found, whole string is the first word
}
```

## Compiler Fight: The Borrow Checker Saves Us

Here's where slices shine—they prevent a whole class of bugs that plague other languages:

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);  // Immutable borrow of s
    
    // WRONG: Can't mutate s while word (a slice) borrows it
    // s.clear();  // ERROR! Cannot borrow `s` as mutable because it is also borrowed as immutable
    
    println!("The first word is: {word}");  // This is fine
    
    // AFTER we stop using word, we can mutate s
    s.clear();  // Now it works!
}
```

**The error you'd see:**
```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:7:5
   |
5  |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
6  |     s.clear();
   |     ^^^^^^^^^ mutable borrow occurs here
7  |     println!("{word}");
   |               ---- immutable borrow later used here
```

Without slices, in C or Python, you might call `clear()` and then try to use the "word" reference—crash or garbage data. Rust catches this at compile time!

## Other Slice Patterns

### Mutable Slices

You can also create mutable slices to modify portions of data:

```rust
fn main() {
    let mut numbers = [1, 2, 3, 4, 5];
    
    // Mutable slice: &mut [i32]
    let slice = &mut numbers[1..4];  // [2, 3, 4]
    slice[0] = 100;  // Modify through the slice
    
    println!("{:?}", numbers);  // [1, 100, 3, 4, 5]
}
```

### Empty Slices

Slices can be empty—useful as sentinel values:

```rust
fn main() {
    let empty: &[i32] = &[];  // Valid empty slice
    let also_empty = &"hello"[0..0];  // Also valid
    println!("Length: {}", empty.len());  // 0
}
```

## Try It Yourself

Write a function `last_three` that returns a slice of the last three elements of any `&[i32]` slice. If the input has fewer than 3 elements, return the entire slice.

```rust
fn last_three(slice: &[i32]) -> &[i32] {
    // Your code here—hint: use conditional slicing
}

fn main() {
    let nums = vec![1, 2, 3, 4, 5];
    println!("Last three: {:?}", last_three(&nums));  // Should print [3, 4, 5]
    
    let small = [10, 20];
    println!("Last three: {:?}", last_three(&small)); // Should print [10, 20]
}
```

**Hint:** Use `slice.len()` and conditional logic to compute the start index.

---
*Next up → **Day 07:** Structs: Defining & Instantiating—we'll build our own data types!*