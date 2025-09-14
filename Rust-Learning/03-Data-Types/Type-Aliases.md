# 🏷️ Type Aliases in Rust

## 📋 Overview

Type aliases give existing types alternative names, making code more readable and maintainable without creating new types.

## 🎯 Basic Type Aliases

### Simple Aliases

```rust
// Create an alias for a type
type Kilometers = i32;
type Miles = i32;

let distance: Kilometers = 5;
let x: i32 = 5;

// These are the same type!
assert_eq!(distance, x); // Works - both are i32

// Type aliases don't provide type safety
fn add_km(km: Kilometers) -> Kilometers {
    km + 10
}

let miles: Miles = 10;
let result = add_km(miles); // Compiles! Miles and Kilometers are both i32
```

### Complex Type Aliases

```rust
// Simplify complex types
type Thunk = Box<dyn Fn() + Send + 'static>;

fn takes_long_type(f: Thunk) {
    // Much cleaner than:
    // fn takes_long_type(f: Box<dyn Fn() + Send + 'static>)
}

fn returns_long_type() -> Thunk {
    Box::new(|| println!("hi"))
}

// Result alias for repeated error types
type Result<T> = std::result::Result<T, std::io::Error>;

fn read_file() -> Result<String> {
    // Instead of: std::result::Result<String, std::io::Error>
    std::fs::read_to_string("file.txt")
}
```

## 🔧 Generic Type Aliases

```rust
// Generic type alias
type Pair<T> = (T, T);

let integers: Pair<i32> = (10, 20);
let floats: Pair<f64> = (1.0, 2.0);

// With constraints
type GenericResult<T> = Result<T, String>;

fn process() -> GenericResult<i32> {
    Ok(42)
}

// Multiple type parameters
type Matrix<T> = Vec<Vec<T>>;

let matrix: Matrix<i32> = vec![
    vec![1, 2, 3],
    vec![4, 5, 6],
];
```

## 📦 Module-Level Type Aliases

```rust
mod database {
    // Hide implementation details
    type ConnectionPool = r2d2::Pool<r2d2_sqlite::SqliteConnectionManager>;
    type Connection = r2d2::PooledConnection<r2d2_sqlite::SqliteConnectionManager>;
    
    pub type DbResult<T> = Result<T, DatabaseError>;
    
    pub struct DatabaseError;
    
    pub fn get_connection() -> DbResult<Connection> {
        // Implementation
        todo!()
    }
}

use database::DbResult;

fn query_user(id: u32) -> DbResult<User> {
    // Clean return type
    todo!()
}
```

## 🎨 Function Pointer Aliases

```rust
// Alias for function pointers
type Operation = fn(i32, i32) -> i32;

fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

fn apply_operation(op: Operation, x: i32, y: i32) -> i32 {
    op(x, y)
}

fn main() {
    let result = apply_operation(add, 5, 3);
    println!("Result: {}", result); // 8
}

// Closure type aliases
type Filter<T> = Box<dyn Fn(&T) -> bool>;

fn filter_vec<T>(vec: Vec<T>, predicate: Filter<T>) -> Vec<T> {
    vec.into_iter().filter(|x| predicate(x)).collect()
}
```

## 🔄 Associated Type Aliases

```rust
trait Container {
    type Item;
    
    fn get(&self, index: usize) -> Option<&Self::Item>;
}

struct MyVec<T> {
    data: Vec<T>,
}

impl<T> Container for MyVec<T> {
    type Item = T;  // Associated type
    
    fn get(&self, index: usize) -> Option<&Self::Item> {
        self.data.get(index)
    }
}

// Use in trait bounds
fn process_container<C: Container>(container: &C) 
where 
    C::Item: std::fmt::Display 
{
    if let Some(item) = container.get(0) {
        println!("First item: {}", item);
    }
}
```

## 🏭 Real-World Examples

### Web Framework Types

```rust
// Common in web frameworks
type Response = hyper::Response<hyper::Body>;
type Request = hyper::Request<hyper::Body>;
type BoxFuture<T> = Pin<Box<dyn Future<Output = T> + Send>>;

async fn handler(req: Request) -> Result<Response, Error> {
    // Handler implementation
    todo!()
}
```

### State Machine Types

```rust
// State machine with type aliases
type StateId = u32;
type TransitionFn = fn(&State, Input) -> StateId;

struct State {
    id: StateId,
    name: String,
    transitions: HashMap<Input, TransitionFn>,
}

struct StateMachine {
    states: HashMap<StateId, State>,
    current: StateId,
}
```

### Database Types

```rust
// Common database type aliases
type UserId = i64;
type PostId = i64;
type Timestamp = chrono::DateTime<chrono::Utc>;

struct User {
    id: UserId,
    created_at: Timestamp,
}

struct Post {
    id: PostId,
    author_id: UserId,
    created_at: Timestamp,
}

// Clear function signatures
fn get_user_posts(user_id: UserId) -> Vec<Post> {
    todo!()
}
```

## 🎯 Type Aliases vs Newtype Pattern

```rust
// Type alias - no type safety
type UserId = i32;
type PostId = i32;

fn get_post(id: PostId) -> Post { todo!() }

let user_id: UserId = 5;
let post = get_post(user_id); // Compiles! No type safety

// Newtype pattern - type safety
struct UserId(i32);
struct PostId(i32);

fn get_post_safe(id: PostId) -> Post { todo!() }

let user_id = UserId(5);
// let post = get_post_safe(user_id); // Error! Type mismatch
let post = get_post_safe(PostId(5)); // Correct
```

## 💡 Best Practices

### 1. Use for Complex Types
```rust
// Bad
fn process(
    callback: Box<dyn Fn(Result<String, io::Error>) -> Result<(), io::Error> + Send>
) { }

// Good
type Callback = Box<dyn Fn(IoResult<String>) -> IoResult<()> + Send>;
type IoResult<T> = Result<T, io::Error>;

fn process(callback: Callback) { }
```

### 2. Document Type Aliases
```rust
/// Represents a user ID in the system
type UserId = u64;

/// A timestamp using UTC timezone
type Timestamp = chrono::DateTime<chrono::Utc>;
```

### 3. Scope Appropriately
```rust
mod network {
    // Private to module
    type Buffer = Vec<u8>;
    
    // Public alias
    pub type Port = u16;
}
```

### 4. Consider Newtype for Type Safety
```rust
// When you need type safety, use newtype
struct Meters(f64);
struct Feet(f64);

// Type alias when you just need clarity
type Celsius = f64;
type Fahrenheit = f64;
```

## 🔍 Common Patterns

### Result Type Shorthand
```rust
// Common in modules
pub type Result<T> = std::result::Result<T, Error>;

pub struct Error {
    // Error implementation
}

pub fn do_something() -> Result<String> {
    Ok("Success".to_string())
}
```

### Platform-Specific Types
```rust
#[cfg(target_pointer_width = "32")]
type PlatformInt = i32;

#[cfg(target_pointer_width = "64")]
type PlatformInt = i64;

// Use PlatformInt throughout code
```

## 🔗 Related Topics

- [[Rust-Learning/03-Data-Types/Custom-Types|Custom Types]]
- [[Rust-Learning/03-Data-Types/Type-Inference|Type Inference]]
- [[Rust-Learning/05-Structs-Enums/README|Structs and Enums]]
- [[Rust-Learning/08-Traits/README|Traits]]

---
#rust #types #type-aliases #type-system
