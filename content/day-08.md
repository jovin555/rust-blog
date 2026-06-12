---
title: "Day 08: Methods on Structs"
date: 2026-06-12
tags: [til, rust, structs, methods, impl]
---

# Day 08: Methods on Structs

## What I Explored Today

Yesterday I learned how to define structs and create instances of them. Today I discovered how to attach **methods** — functions that belong to a struct — using `impl` blocks. Methods let me bundle behavior with data, which is the heart of object-oriented programming in Rust.

The magic moment came when I realized that Rust's methods are just functions with a special first parameter: `self` (or `&self`, or `&mut self`). This makes the syntax clean and the ownership semantics explicit.

## Anatomy of an `impl` Block

Every method lives inside an `impl` block for its struct. Here's the basic pattern:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

// impl block: all methods for Rectangle go here
impl Rectangle {
    // &self borrows the struct immutably — we're just reading data
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Another read-only method
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };

    // Call methods with dot notation — just like other languages!
    println!("Area: {}", rect.area());          // 1500
    println!("Is square? {}", rect.is_square()); // false
}
```

Key insight: `&self` is shorthand for `self: &Self`. The `Self` type (capital S) refers to the struct type inside the `impl` block. So `&self` means "I'm borrowing this struct immutably."

## Three Flavors of `self`

Rust gives you three choices for the `self` parameter, each with different ownership implications:

```rust
struct Counter {
    count: u32,
}

impl Counter {
    // 1. &self — borrows immutably (most common for read-only)
    fn value(&self) -> u32 {
        self.count
    }

    // 2. &mut self — borrows mutably (for methods that change the struct)
    fn increment(&mut self) {
        self.count += 1;
    }

    // 3. self — takes ownership (rare, used for "consuming" the struct)
    fn into_string(self) -> String {
        format!("Counter({})", self.count)
        // self is dropped here — can't use the struct after this!
    }
}

fn main() {
    let mut counter = Counter { count: 0 }; // must be mut for mutable borrows
    println!("Start: {}", counter.value());  // 0

    counter.increment(); // &mut self — works because counter is mut
    counter.increment();
    println!("After increment: {}", counter.value()); // 2

    let string_repr = counter.into_string(); // takes ownership!
    println!("String: {}", string_repr);
    // println!("{}", counter.value()); // ERROR! counter is gone
}
```

**Why three flavors?** Rust forces you to think about ownership even at the method level. `&self` is safe and non-destructive, `&mut self` allows mutation, and `self` is for when you want to consume the struct (e.g., converting it to another type).

## Associated Functions (No `self`)

Not every function in an `impl` block needs `self`. These are **associated functions** — like static methods in other languages. They're called with `::` syntax.

```rust
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    // Associated function — no self parameter
    // Often used as "constructors"
    fn origin() -> Point {
        Point { x: 0.0, y: 0.0 }
    }

    // Another associated function — takes two f64s
    fn new(x: f64, y: f64) -> Point {
        Point { x, y }
    }

    // Regular method
    fn distance_from(&self, other: &Point) -> f64 {
        let dx = self.x - other.x;
        let dy = self.y - other.y;
        (dx * dx + dy * dy).sqrt()
    }
}

fn main() {
    // Call associated functions with ::
    let origin = Point::origin();
    let p1 = Point::new(3.0, 4.0);

    // Call methods with .
    println!("Distance: {}", p1.distance_from(&origin)); // 5.0
}
```

The convention is to name your "constructor" `new`, but it's not a keyword — you could call it `create` or `from_coords`. `new` is just the Rust community standard.

## Compiler Fight: Forgetting `mut` on the Binding

Here's a mistake I made three times today:

```rust
struct BankAccount {
    balance: f64,
}

impl BankAccount {
    fn deposit(&mut self, amount: f64) {
        self.balance += amount;
    }
}

fn main() {
    let account = BankAccount { balance: 100.0 };
    //   ^^^^^^^ not mutable!

    account.deposit(50.0);
    //     ^^^^^^^ ERROR: cannot borrow `account` as mutable
}
```

**The error:**
```
error[E0596]: cannot borrow `account` as mutable, as it is not declared as mutable
 --> src/main.rs:14:5
  |
8 |     let account = BankAccount { balance: 100.0 };
  |         ------- help: consider changing this to be mutable: `mut account`
```

**The fix:** Change `let account` to `let mut account`. The method signature says `&mut self`, which means the compiler checks that the variable is mutable at the call site.

## Chaining Methods with `&mut self`

One cool pattern: if your methods return `&mut Self`, you can chain calls:

```rust
impl Counter {
    fn increment(&mut self) -> &mut Self {
        self.count += 1;
        self // return the mutable reference for chaining
    }

    fn reset(&mut self) -> &mut Self {
        self.count = 0;
        self
    }
}

fn main() {
    let mut counter = Counter { count: 0 };
    counter.increment().increment().reset().increment();
    // Chained! count is now 1
    println!("{}", counter.value()); // 1
}
```

This pattern is common in builder APIs and configuration structs.

## Try It Yourself

Create a `Book` struct with fields `title`, `author`, and `pages`. Implement:
- A `new` associated function that creates a new book
- A `summary` method that returns a formatted string
- A `read` method that takes `&mut self` and decrements `pages` by 1 (but never below 0)
- Test it in `main` with a few method calls

```rust
// Your code here
struct Book {
    title: String,
    author: String,
    pages: u32,
}

impl Book {
    // Implement new, summary, and read
}

fn main() {
    // Create a book and call its methods
}
```

**Hint:** For `summary`, use `format!("{} by {} ({} pages)", self.title, self.author, self.pages)`.

---
*Next up → **Day 09:** Enums & Pattern Matching with `match` — Rust's superpower for handling multiple states and making illegal states unrepresentable.*