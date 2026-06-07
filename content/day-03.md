---
title: "Day 03: Control Flow: if, loop, while, for"
date: 2026-06-07
tags: [til, control-flow, loops, conditionals]
---

# Day 03: Control Flow: if, loop, while, for

## What I Explored Today

Yesterday I mastered variables and types. Today I learned that variables are boring if you can't do anything with them! Control flow is what makes programs *alive* — deciding what to do, when to repeat, and how to iterate. Rust's control flow is familiar but has some sharp edges that I keep tripping over.

## The `if` Expression

First shocker: `if` in Rust is an **expression**, not a statement. That means it returns a value!

```rust
fn main() {
    let temperature = 30;
    
    // Classic if/else — note no parentheses around condition
    if temperature > 25 {
        println!("It's hot!");
    } else if temperature > 15 {
        println!("It's pleasant.");
    } else {
        println!("It's chilly.");
    }
    
    // if as an expression — every arm must return the same type
    let weather = if temperature > 25 {
        "hot"
    } else if temperature > 15 {
        "pleasant"
    } else {
        "chilly"
    };
    println!("The weather is {}.", weather);
}
```

**Key rules:**
- No parentheses around conditions (unlike C/JavaScript)
- Conditions **must** be `bool` — no truthy/falsy nonsense
- When using `if` as an expression, all arms must be the same type

### Compiler Fight #1: Non-boolean condition

```rust
fn main() {
    let x = 1;
    // ERROR: expected `bool`, found integer
    // if x {
    //     println!("x is truthy");
    // }
    
    // Rust forces explicit comparison
    if x != 0 {
        println!("x is non-zero");
    }
}
```

Rust doesn't play the "0 is falsey" game. You must write a proper boolean condition. Annoying at first, but it prevents a whole class of bugs where you accidentally assign instead of compare (`if x = 5` in C, anyone?).

## The `loop` Keyword — Infinite by Design

Rust has `loop` for when you want to run forever (or until told to stop). It's the simplest loop construct.

```rust
fn main() {
    let mut counter = 0;
    
    // Basic infinite loop — we must break out
    loop {
        counter += 1;
        println!("Count: {}", counter);
        
        if counter >= 3 {
            break;  // exits the loop
        }
    }
    
    // loop is also an expression! break can return a value
    let mut guess_count = 0;
    let result = loop {
        guess_count += 1;
        if guess_count >= 5 {
            break "took too long";
        }
        if guess_count == 3 {
            break "got it on try 3";
        }
    };
    println!("Result: {}", result);
}
```

The `break` returning a value is incredibly useful. I can already see how this cleans up game loops and retry logic.

## The `while` Loop — Conditional Repetition

`while` is what you expect: repeat while a condition is true.

```rust
fn main() {
    let mut countdown = 3;
    
    while countdown > 0 {
        println!("{}...", countdown);
        countdown -= 1;
    }
    println!("Liftoff!");
    
    // Classic use: waiting for a condition to change
    let mut attempts = 0;
    let max_attempts = 3;
    let mut authenticated = false;
    
    while !authenticated && attempts < max_attempts {
        attempts += 1;
        println!("Attempt {} of {}", attempts, max_attempts);
        // Simulate checking credentials
        if attempts == 2 {
            authenticated = true;
        }
    }
    
    if authenticated {
        println!("Welcome!");
    } else {
        println!("Access denied.");
    }
}
```

### Compiler Fight #2: Forgetting `break` in `loop` vs `while`

```rust
fn main() {
    let mut x = 5;
    
    // This loop runs forever — I forgot break!
    // loop {
    //     x -= 1;
    //     if x == 0 {
    //         // forgot break here — infinite loop!
    //     }
    // }
    
    // Safer: while with explicit condition
    while x > 0 {
        x -= 1;
        println!("x is now {}", x);
    }
    // The loop naturally exits when condition becomes false
}
```

The compiler won't save you from infinite loops. `while` is safer when you have a clear exit condition. `loop` is for when you need more complex exit logic.

## The `for` Loop — Iteration Done Right

Rust's `for` loop is elegant and safe. It iterates over anything iterable.

```rust
fn main() {
    // Range syntax: start..end (exclusive end)
    for i in 0..5 {
        println!("i = {}", i);  // prints 0, 1, 2, 3, 4
    }
    
    // Inclusive range: start..=end
    for i in 0..=3 {
        println!("i = {}", i);  // prints 0, 1, 2, 3
    }
    
    // Iterating over an array
    let numbers = [10, 20, 30, 40];
    for num in numbers {
        println!("{}", num);
    }
    
    // With index using .iter().enumerate()
    let fruits = ["apple", "banana", "cherry"];
    for (index, fruit) in fruits.iter().enumerate() {
        println!("Fruit #{} is {}", index + 1, fruit);
    }
    
    // Reverse iteration
    for i in (1..=5).rev() {
        println!("{}...", i);
    }
    println!("Boom!");
}
```

**Why `for` is preferred:** Rust's `for` loop is zero-cost abstraction — it compiles down to the same machine code as a manual `while` loop, but is safer and more readable.

## Loop Labels — Breaking Out of Nested Loops

Rust lets you name loops and break out of specific ones. This blew my mind.

```rust
fn main() {
    'outer: for i in 1..=3 {
        println!("Outer loop: i = {}", i);
        
        'inner: for j in 1..=3 {
            if i == 2 && j == 2 {
                println!("Breaking out of outer loop!");
                break 'outer;  // exits the outer loop, not inner
            }
            println!("  Inner loop: j = {}", j);
        }
    }
    println!("Escaped the nested loops!");
}
```

Without loop labels, `break` only exits the innermost loop. Labels give you precise control — extremely useful in search algorithms and matrix operations.

## Performance Tip: Which Loop to Use?

| Loop | When to Use |
|------|-------------|
| `loop` | When you need to retry until success, or when exit logic is complex |
| `while` | When you have a clear, simple condition to check before each iteration |
| `for` | **Preferred** — when iterating over a collection or range |

Rust's `for` loop is almost always the right choice. The compiler optimizes it heavily. Only reach for `while` or `loop` when `for` doesn't fit.

## Try It Yourself

```rust
fn main() {
    // TODO: Print the multiplication table for 7 (1 through 10)
    // Use a for loop and the range syntax
    
    // Your code here:
    for i in 1..=10 {
        println!("7 x {} = {}", i, 7 * i);
    }
    
    // BONUS: Rewrite using while
    let mut j = 1;
    while j <= 10 {
        println!("7 x {} = {}", j, 7 * j);
        j += 1;
    }
}
```

Run this and see both versions produce identical output. The `for` version is cleaner and less error-prone (no manual increment).

---
*Next up → **Day 04:** Ownership: The Core Mental Model — where Rust's most unique feature changes everything you know about memory management*