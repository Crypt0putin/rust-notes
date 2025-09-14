# 🔧 Function Definition in Rust

## 📋 Overview

Functions are the building blocks of Rust programs. They encapsulate behavior and can be reused throughout your code.

## 🎯 Basic Function Syntax

### Simple Functions

```rust
// Basic function
fn main() {
    println!("Hello, world!");
    another_function();
}

fn another_function() {
    println!("Another function.");
}

// Function with parameters
fn print_value(x: i32) {
    println!("The value is: {}", x);
}

// Multiple parameters
fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

### Functions with Return Values

```rust
// Explicit return
fn five() -> i32 {
    return 5;
}

// Implicit return (last expression)
fn plus_one(x: i32) -> i32 {
    x + 1  // No semicolon!
}

// Early return
fn check_number(x: i32) -> &'static str {
    if x < 0 {
        return "negative";
    }
    if x == 0 {
        return "zero";
    }
    "positive"  // Implicit return
}
```

## 📊 Function Parameters

### By Value (Move/Copy)

```rust
// Copy types
fn print_number(x: i32) {
    println!("Number: {}", x);
}

// Move types
fn consume_string(s: String) {
    println!("String: {}", s);
    // s is dropped here
}

fn main() {
    let num = 5;
    print_number(num);
    println!("Can still use num: {}", num); // OK - Copy

    let text = String::from("hello");
    consume_string(text);
    // println!("{}", text); // Error! text was moved
}
```

### By Reference

```rust
// Immutable reference
fn calculate_length(s: &String) -> usize {
    s.len()
}

// Mutable reference
fn append_world(s: &mut String) {
    s.push_str(", world!");
}

fn main() {
    let s = String::from("hello");
    let len = calculate_length(&s);
    println!("Length of '{}' is {}", s, len);
    
    let mut s2 = String::from("hello");
    append_world(&mut s2);
    println!("{}", s2); // "hello, world!"
}
```

### Slices as Parameters

```rust
// Better: accept string slices
fn better_length(s: &str) -> usize {
    s.len()
}

// Works with both String and &str
fn main() {
    let string = String::from("hello");
    let literal = "world";
    
    println!("{}", better_length(&string));
    println!("{}", better_length(literal));
}

// Array slices
fn sum_slice(values: &[i32]) -> i32 {
    values.iter().sum()
}
```

## 🎨 Advanced Function Features

### Multiple Return Values

```rust
// Using tuples
fn calculate_stats(numbers: &[i32]) -> (i32, i32, f64) {
    let sum: i32 = numbers.iter().sum();
    let count = numbers.len() as i32;
    let average = sum as f64 / count as f64;
    
    (sum, count, average)
}

// Using custom struct
struct Stats {
    sum: i32,
    count: usize,
    average: f64,
}

fn calculate_stats_struct(numbers: &[i32]) -> Stats {
    let sum: i32 = numbers.iter().sum();
    let count = numbers.len();
    let average = sum as f64 / count as f64;
    
    Stats { sum, count, average }
}
```

### Generic Functions

```rust
// Single type parameter
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

// Multiple type parameters
fn swap<T, U>(pair: (T, U)) -> (U, T) {
    (pair.1, pair.0)
}

// With trait bounds
fn print_debug<T: std::fmt::Debug>(value: &T) {
    println!("{:?}", value);
}
```

### Functions with Lifetimes

```rust
// Lifetime annotations
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Multiple lifetimes
fn first_word<'a>(s: &'a str) -> &'a str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    &s[..]
}
```

## 🔄 Function Pointers

```rust
// Function pointer type
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn subtract(a: i32, b: i32) -> i32 {
    a - b
}

fn do_operation(f: fn(i32, i32) -> i32, x: i32, y: i32) -> i32 {
    f(x, y)
}

fn main() {
    let result = do_operation(add, 5, 3);
    println!("Result: {}", result); // 8
    
    let result = do_operation(subtract, 5, 3);
    println!("Result: {}", result); // 2
}

// Returning function pointers
fn get_operation(op: &str) -> fn(i32, i32) -> i32 {
    match op {
        "add" => add,
        "subtract" => subtract,
        _ => |a, b| a * b,  // Default to multiplication
    }
}
```

## 🎭 Closures

```rust
// Basic closures
let add_one = |x| x + 1;
let result = add_one(5);

// With type annotations
let add: fn(i32, i32) -> i32 = |x, y| x + y;

// Capturing environment
let factor = 2;
let multiply = |x| x * factor;
println!("{}", multiply(5)); // 10

// Move semantics
let s = String::from("hello");
let consume = move || println!("{}", s);
consume();
// println!("{}", s); // Error! s was moved

// Closures as parameters
fn apply_twice<F>(f: F, x: i32) -> i32
where
    F: Fn(i32) -> i32,
{
    f(f(x))
}

let double = |x| x * 2;
let result = apply_twice(double, 3); // 12
```

## 🏭 Function Design Patterns

### Builder Functions

```rust
struct Config {
    host: String,
    port: u16,
    timeout: u64,
}

impl Config {
    // Constructor function
    fn new(host: impl Into<String>) -> Self {
        Config {
            host: host.into(),
            port: 8080,
            timeout: 30,
        }
    }
    
    // Builder methods
    fn with_port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }
    
    fn with_timeout(mut self, timeout: u64) -> Self {
        self.timeout = timeout;
        self
    }
}

// Usage
let config = Config::new("localhost")
    .with_port(3000)
    .with_timeout(60);
```

### Result-Returning Functions

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_contents(path: &str) -> Result<String, io::Error> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

// With custom error types
#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(std::num::ParseIntError),
}

fn process_file(path: &str) -> Result<i32, AppError> {
    let contents = read_file_contents(path)
        .map_err(AppError::Io)?;
    
    contents.trim()
        .parse()
        .map_err(AppError::Parse)
}
```

## 💡 Best Practices

### 1. Prefer &str over &String
```rust
// Good
fn process(s: &str) { }

// Less flexible
fn process(s: &String) { }
```

### 2. Use impl Trait for parameters
```rust
// Good - accepts String, &str, etc.
fn set_name(name: impl Into<String>) { }

// More restrictive
fn set_name(name: String) { }
```

### 3. Return iterators when possible
```rust
fn get_even_numbers() -> impl Iterator<Item = i32> {
    (0..100).filter(|x| x % 2 == 0)
}
```

### 4. Document panic conditions
```rust
/// Divides two numbers.
/// 
/// # Panics
/// 
/// Panics if `divisor` is zero.
fn divide(dividend: i32, divisor: i32) -> i32 {
    if divisor == 0 {
        panic!("Division by zero!");
    }
    dividend / divisor
}
```

## 🎯 Common Patterns

### Option/Result Returning
```rust
fn find_user(id: u32) -> Option<User> {
    // Return Some(user) or None
    todo!()
}

fn validate_input(input: &str) -> Result<(), ValidationError> {
    // Return Ok(()) or Err(error)
    todo!()
}
```

### Default Parameters (via builder)
```rust
struct Request {
    url: String,
    method: String,
    timeout: u64,
}

impl Request {
    fn get(url: impl Into<String>) -> Self {
        Request {
            url: url.into(),
            method: "GET".to_string(),
            timeout: 30,
        }
    }
}
```

## 🔗 Related Topics

- [[Rust-Learning/04-Functions-Control/Closures|Closures]]
- [[Rust-Learning/04-Functions-Control/Methods|Methods and Associated Functions]]
- [[Rust-Learning/07-Error-Handling/Result-Type|Error Handling]]
- [[Rust-Learning/09-Lifetimes/Lifetime-Basics|Lifetimes in Functions]]

---
#rust #functions #parameters #return-values #closures
