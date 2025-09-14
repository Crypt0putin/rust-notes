# 🎨 Custom Types in Rust

## 📋 Overview

Rust allows you to create custom types using **structs** and **enums**. These are the building blocks for creating complex data structures.

## 🏗️ Structures (Structs)

### Classic C-style Structs

```rust
// Define a struct
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// Create an instance
let user1 = User {
    email: String::from("user@example.com"),
    username: String::from("someusername"),
    active: true,
    sign_in_count: 1,
};

// Access fields
println!("Email: {}", user1.email);

// Mutable struct
let mut user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername"),
    active: true,
    sign_in_count: 1,
};

user2.email = String::from("newemail@example.com");
```

### Struct Update Syntax

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1  // Use remaining fields from user1
};

// Note: This moves data if fields don't implement Copy
```

### Tuple Structs

```rust
// Tuple structs - named tuples
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);

// Access like tuples
println!("Red value: {}", black.0);

// Destructuring
let Color(r, g, b) = black;
```

### Unit-like Structs

```rust
// Structs without fields
struct AlwaysEqual;

let subject = AlwaysEqual;

// Useful for implementing traits on a type
impl AlwaysEqual {
    fn new() -> Self {
        AlwaysEqual
    }
}
```

## 🎭 Enumerations (Enums)

### Basic Enums

```rust
enum IpAddrKind {
    V4,
    V6,
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;

fn route(ip_kind: IpAddrKind) {
    match ip_kind {
        IpAddrKind::V4 => println!("IPv4"),
        IpAddrKind::V6 => println!("IPv6"),
    }
}
```

### Enums with Data

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));

// More complex enum
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },   // Named fields
    Write(String),              // Single value
    ChangeColor(i32, i32, i32), // Tuple
}

impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(text) => println!("Text: {}", text),
            Message::ChangeColor(r, g, b) => {
                println!("Color: rgb({}, {}, {})", r, g, b)
            }
        }
    }
}
```

## 🎯 The Option Type

```rust
// Option is defined in the standard library
enum Option<T> {
    None,
    Some(T),
}

// Usage (no need to import, it's in prelude)
let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None;

// Pattern matching with Option
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

// Using if let for simple cases
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

## ⚡ The Result Type

```rust
// Result is defined in the standard library
enum Result<T, E> {
    Ok(T),
    Err(E),
}

use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening file: {:?}", error),
    };
}

// Using ? operator
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

## 🔧 Methods and Associated Functions

### Implementing Methods

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method (takes &self)
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // Mutable method
    fn double_size(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
    
    // Associated function (no self)
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
    
    // Method that takes ownership
    fn destroy(self) {
        println!("Rectangle destroyed!");
        // self is moved and dropped here
    }
}

// Usage
let rect = Rectangle { width: 30, height: 50 };
println!("Area: {}", rect.area());

let square = Rectangle::square(10);
```

### Multiple impl Blocks

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

## 🎨 Advanced Patterns

### Generic Types

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// Different types for x and y
struct Point2<T, U> {
    x: T,
    y: U,
}

// Generic enums
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Recursive Types with Box

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
```

### Newtype Pattern

```rust
// Create distinct types for type safety
struct Meters(f64);
struct Kilometers(f64);

impl Meters {
    fn to_kilometers(&self) -> Kilometers {
        Kilometers(self.0 / 1000.0)
    }
}

// Can't accidentally mix types
fn calculate_time(distance: Kilometers, speed: f64) -> f64 {
    distance.0 / speed
}
```

## 🏭 Builder Pattern

```rust
#[derive(Debug, Default)]
struct Server {
    host: Option<String>,
    port: Option<u16>,
    timeout: Option<u64>,
}

struct ServerBuilder {
    server: Server,
}

impl ServerBuilder {
    fn new() -> Self {
        ServerBuilder {
            server: Server::default(),
        }
    }
    
    fn host(mut self, host: impl Into<String>) -> Self {
        self.server.host = Some(host.into());
        self
    }
    
    fn port(mut self, port: u16) -> Self {
        self.server.port = Some(port);
        self
    }
    
    fn timeout(mut self, timeout: u64) -> Self {
        self.server.timeout = Some(timeout);
        self
    }
    
    fn build(self) -> Result<Server, &'static str> {
        if self.server.host.is_none() {
            return Err("Host is required");
        }
        Ok(self.server)
    }
}

// Usage
let server = ServerBuilder::new()
    .host("localhost")
    .port(8080)
    .timeout(30)
    .build()
    .unwrap();
```

## 📦 Common Derive Macros

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
enum Priority {
    Low,
    Medium,
    High,
}

// Default implementation
#[derive(Default)]
struct Config {
    #[default = 8080]
    port: u16,
    #[default = "localhost".to_string()]
    host: String,
}
```

## 🎯 Best Practices

1. **Use structs for data that belongs together**
2. **Use enums for a fixed set of possibilities**
3. **Implement Display and Debug traits**
4. **Use the newtype pattern for type safety**
5. **Consider the builder pattern for complex construction**

## 🔗 Related Topics

- [[Rust-Learning/05-Structs-Enums/Struct-Definition|Struct Definition]]
- [[Rust-Learning/05-Structs-Enums/Enum-Definition|Enum Definition]]
- [[Rust-Learning/08-Traits/Trait-Definition|Traits]]
- [[Rust-Learning/03-Data-Types/Type-Aliases|Type Aliases]]

---
#rust #types #structs #enums #custom-types
