# 🔨 Methods and Associated Functions

## 📋 Overview

Methods are functions defined within the context of a struct, enum, or trait. Associated functions are functions associated with a type but don't take `self` as a parameter.

## 🎯 Method Syntax

### Basic Methods

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
    
    // Method with mutable self
    fn double_size(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
    
    // Method that consumes self
    fn consume(self) -> u32 {
        self.width + self.height
        // self is moved and dropped here
    }
}

fn main() {
    let rect = Rectangle { 
        width: 30, 
        height: 50 
    };
    
    println!("Area: {}", rect.area());
    
    let mut rect2 = Rectangle { 
        width: 10, 
        height: 20 
    };
    rect2.double_size();
    
    let rect3 = Rectangle { 
        width: 5, 
        height: 10 
    };
    let sum = rect3.consume();
    // rect3 is no longer accessible
}
```

### Method Receivers

```rust
struct MyStruct {
    value: i32,
}

impl MyStruct {
    // Immutable borrow
    fn get_value(&self) -> i32 {
        self.value
    }
    
    // Mutable borrow
    fn set_value(&mut self, value: i32) {
        self.value = value;
    }
    
    // Take ownership
    fn into_value(self) -> i32 {
        self.value
    }
    
    // Rare: borrowed box
    fn box_method(self: Box<Self>) -> i32 {
        self.value
    }
    
    // Smart pointer receivers
    fn arc_method(self: Arc<Self>) -> i32 {
        self.value
    }
}
```

## 🏭 Associated Functions

### Constructor Pattern

```rust
struct User {
    username: String,
    email: String,
    active: bool,
}

impl User {
    // Associated function (constructor)
    fn new(username: String, email: String) -> Self {
        User {
            username,
            email,
            active: true,
        }
    }
    
    // Another associated function
    fn guest() -> Self {
        User {
            username: String::from("guest"),
            email: String::from("guest@example.com"),
            active: false,
        }
    }
    
    // Builder pattern
    fn builder() -> UserBuilder {
        UserBuilder::default()
    }
}

// Usage
let user = User::new(
    String::from("alice"),
    String::from("alice@example.com")
);
let guest = User::guest();
```

### Factory Methods

```rust
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Square { side: f64 },
}

impl Shape {
    // Factory methods
    fn new_circle(radius: f64) -> Self {
        Shape::Circle { radius }
    }
    
    fn new_rectangle(width: f64, height: f64) -> Self {
        Shape::Rectangle { width, height }
    }
    
    fn new_square(side: f64) -> Self {
        Shape::Square { side }
    }
    
    // Method on enum
    fn area(&self) -> f64 {
        match self {
            Shape::Circle { radius } => std::f64::consts::PI * radius * radius,
            Shape::Rectangle { width, height } => width * height,
            Shape::Square { side } => side * side,
        }
    }
}
```

## 🔄 Multiple impl Blocks

```rust
struct Config {
    debug: bool,
    verbose: bool,
}

// Logical grouping of methods
impl Config {
    fn new() -> Self {
        Config {
            debug: false,
            verbose: false,
        }
    }
}

// Separate impl block for different functionality
impl Config {
    fn enable_debug(&mut self) {
        self.debug = true;
    }
    
    fn enable_verbose(&mut self) {
        self.verbose = true;
    }
}

// Conditional compilation
#[cfg(debug_assertions)]
impl Config {
    fn debug_defaults() -> Self {
        Config {
            debug: true,
            verbose: true,
        }
    }
}
```

## 🎨 Generic Methods

### Methods on Generic Structs

```rust
struct Point<T> {
    x: T,
    y: T,
}

// Methods for any T
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
    
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }
}

// Methods only for specific types
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

// Methods with different generic parameters
impl<T> Point<T> {
    fn mixup<U>(self, other: Point<U>) -> Point<T, U> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```

### Methods with Trait Bounds

```rust
use std::fmt::Display;

struct Wrapper<T> {
    value: T,
}

impl<T> Wrapper<T> {
    fn new(value: T) -> Self {
        Wrapper { value }
    }
}

impl<T: Display> Wrapper<T> {
    fn show(&self) {
        println!("Value: {}", self.value);
    }
}

impl<T: Clone> Wrapper<T> {
    fn duplicate(&self) -> Self {
        Wrapper {
            value: self.value.clone(),
        }
    }
}
```

## 🔧 Method Chaining

### Builder Pattern

```rust
#[derive(Default)]
struct QueryBuilder {
    table: Option<String>,
    select: Vec<String>,
    where_clause: Option<String>,
    limit: Option<usize>,
}

impl QueryBuilder {
    fn new() -> Self {
        QueryBuilder::default()
    }
    
    fn table(mut self, table: &str) -> Self {
        self.table = Some(table.to_string());
        self
    }
    
    fn select(mut self, columns: Vec<&str>) -> Self {
        self.select = columns.iter().map(|s| s.to_string()).collect();
        self
    }
    
    fn where_clause(mut self, clause: &str) -> Self {
        self.where_clause = Some(clause.to_string());
        self
    }
    
    fn limit(mut self, limit: usize) -> Self {
        self.limit = Some(limit);
        self
    }
    
    fn build(self) -> String {
        // Build SQL query string
        format!("SELECT ... FROM {} ...", 
            self.table.unwrap_or_default())
    }
}

// Usage
let query = QueryBuilder::new()
    .table("users")
    .select(vec!["id", "name", "email"])
    .where_clause("active = true")
    .limit(10)
    .build();
```

## 🎭 Trait Methods

### Implementing Trait Methods

```rust
trait Drawable {
    fn draw(&self);
    
    // Default implementation
    fn clear(&self) {
        println!("Clearing...");
    }
}

struct Circle {
    radius: f64,
}

impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing circle with radius {}", self.radius);
    }
    
    // Override default
    fn clear(&self) {
        println!("Clearing circle");
    }
}

// Extension traits
trait StringExt {
    fn capitalize_first(&self) -> String;
}

impl StringExt for str {
    fn capitalize_first(&self) -> String {
        let mut chars = self.chars();
        match chars.next() {
            None => String::new(),
            Some(first) => first.to_uppercase().collect::<String>() + chars.as_str(),
        }
    }
}

// Usage
let text = "hello";
println!("{}", text.capitalize_first()); // "Hello"
```

## 💡 Advanced Patterns

### Self Type in Methods

```rust
struct Counter {
    value: i32,
}

impl Counter {
    // Returns Self type
    fn new() -> Self {
        Counter { value: 0 }
    }
    
    // Takes and returns Self
    fn increment(mut self) -> Self {
        self.value += 1;
        self
    }
    
    // Method with Self in generics
    fn combine<F>(self, other: Self, f: F) -> Self
    where
        F: Fn(i32, i32) -> i32,
    {
        Counter {
            value: f(self.value, other.value),
        }
    }
}
```

### Recursive Methods

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

impl List {
    fn new() -> Self {
        List::Nil
    }
    
    fn prepend(self, value: i32) -> Self {
        List::Cons(value, Box::new(self))
    }
    
    fn len(&self) -> usize {
        match self {
            List::Nil => 0,
            List::Cons(_, tail) => 1 + tail.len(),
        }
    }
    
    fn sum(&self) -> i32 {
        match self {
            List::Nil => 0,
            List::Cons(value, tail) => value + tail.sum(),
        }
    }
}
```

## 🎯 Best Practices

### 1. Use &self for Read-Only Methods
```rust
impl MyStruct {
    // Good
    fn get_value(&self) -> i32 {
        self.value
    }
    
    // Avoid unless ownership is needed
    fn get_value_bad(self) -> i32 {
        self.value
    }
}
```

### 2. Consistent Naming
```rust
impl Container {
    fn len(&self) -> usize { }      // Not length()
    fn is_empty(&self) -> bool { }  // Not empty()
    fn push(&mut self, item: T) { } // Not add()
}
```

### 3. Builder Methods Return Self
```rust
impl Builder {
    fn option1(mut self, val: T) -> Self {
        self.field = val;
        self  // Return self for chaining
    }
}
```

### 4. Use new() for Main Constructor
```rust
impl MyType {
    fn new() -> Self { }           // Primary constructor
    fn with_capacity(n: usize) { } // Alternative constructors
    fn from_string(s: String) { }  // Conversion constructors
}
```

## 🔗 Related Topics

- [[Rust-Learning/05-Structs-Enums/Struct-Methods|Struct Methods]]
- [[Rust-Learning/08-Traits/Trait-Definition|Traits]]
- [[Rust-Learning/04-Functions-Control/Function-Definition|Functions]]
- [[Rust-Learning/02-Ownership/README|Ownership in Methods]]

---
#rust #methods #associated-functions #impl #self
