# 🔍 Type Inference in Rust

## 📋 Overview

Rust's type inference allows the compiler to automatically determine types based on context, reducing boilerplate while maintaining type safety.

## 🎯 Basic Type Inference

### Variable Type Inference

```rust
// Compiler infers types
let x = 5;          // i32 (default integer type)
let y = 2.0;        // f64 (default float type)
let z = true;       // bool
let s = "hello";    // &str

// Inference from usage
let mut vec = Vec::new(); // Type unknown yet
vec.push(5);              // Now inferred as Vec<i32>

// Inference from function return
fn returns_u8() -> u8 { 42 }
let num = returns_u8(); // num is u8
```

### Collection Type Inference

```rust
// Vec inference
let v = vec![1, 2, 3];           // Vec<i32>
let v = vec![1.0, 2.0, 3.0];     // Vec<f64>

// HashMap inference
use std::collections::HashMap;
let mut map = HashMap::new();    // Type unknown
map.insert("key", "value");      // Now HashMap<&str, &str>

// Iterator inference
let doubled: Vec<_> = vec![1, 2, 3]
    .iter()
    .map(|x| x * 2)
    .collect();  // Vec<i32> inferred from context
```

## 🔧 Advanced Type Inference

### Partial Type Annotations

```rust
// Specify only what's needed
let numbers: Vec<_> = vec![1, 2, 3]; // Only specify container

// Turbofish syntax for methods
let parsed = "42".parse::<i32>().unwrap();

// Or let type inference work
let parsed: i32 = "42".parse().unwrap();

// Complex example
let result: Result<Vec<_>, _> = 
    ["1", "2", "3"]
    .iter()
    .map(|s| s.parse::<i32>())
    .collect();
```

### Closure Type Inference

```rust
// Closure parameter types inferred
let add_one = |x| x + 1;        // Type inferred from usage
let result = add_one(5);        // x inferred as i32

// Multiple uses lock in the type
let example_closure = |x| x;
let s = example_closure(String::from("hello")); // x is String
// let n = example_closure(5);  // Error! x already inferred as String

// Explicit types when needed
let add: fn(i32, i32) -> i32 = |x, y| x + y;
```

## 📊 Type Inference Rules

### Default Types

```rust
// Integer literals default to i32
let x = 42;           // i32
let y = 100_000;      // i32

// Float literals default to f64
let f = 3.14;         // f64
let g = 2.0;          // f64

// Array size must be known
let arr = [0; 10];    // [i32; 10]

// Override defaults
let x = 42u8;         // u8
let y = 3.14f32;      // f32
```

### Inference from Context

```rust
// Function parameter types guide inference
fn take_u8(x: u8) {}
fn take_string(s: String) {}

let num = 5;
take_u8(num);        // num must be u8

let text = String::from("hello");
take_string(text);   // text is String

// Method calls guide inference
struct Container<T> {
    value: T,
}

impl<T> Container<T> {
    fn new(value: T) -> Self {
        Container { value }
    }
}

let c = Container::new(42);  // Container<i32>
```

## 🎨 Generic Type Inference

### Function Generics

```rust
fn identity<T>(x: T) -> T {
    x
}

// Type inferred from argument
let num = identity(5);        // T is i32
let text = identity("hello"); // T is &str

// Multiple type parameters
fn swap<T, U>(pair: (T, U)) -> (U, T) {
    (pair.1, pair.0)
}

let swapped = swap((1, "hello")); // (U=&str, T=i32)
```

### Struct Generics

```rust
struct Point<T> {
    x: T,
    y: T,
}

// Inference from construction
let integer_point = Point { x: 5, y: 10 };      // Point<i32>
let float_point = Point { x: 1.0, y: 4.0 };     // Point<f64>

// Mixed inference
struct Pair<T, U> {
    first: T,
    second: U,
}

let pair = Pair { 
    first: 5, 
    second: "hello" 
}; // Pair<i32, &str>
```

## 🔄 Type Inference Limitations

### When Inference Fails

```rust
// Cannot infer without context
// let vec = Vec::new(); // Error: cannot infer type

// Solutions:
let vec: Vec<i32> = Vec::new();      // Explicit annotation
let vec = Vec::<i32>::new();         // Turbofish
let mut vec = Vec::new();
vec.push(5);                         // Inferred from usage

// Ambiguous numeric types
// let num = 5.pow(2); // Error: ambiguous

// Solutions:
let num: i32 = 5.pow(2);            // Annotation
let num = 5i32.pow(2);               // Suffix
let num = (5 as i32).pow(2);        // Cast
```

### Inference with Traits

```rust
// Sometimes needs help with trait methods
let result = "5".parse(); // Error: cannot infer

// Solutions:
let result: i32 = "5".parse().unwrap();
let result = "5".parse::<i32>().unwrap();

// Complex trait bounds
fn complex<T, U>() 
where 
    T: From<U>,
    U: Default,
{
    let u = U::default();
    let t = T::from(u); // Types inferred from trait bounds
}
```

## 💡 Tips and Tricks

### The Underscore Placeholder

```rust
// Use _ for partial inference
let numbers: Vec<_> = (0..10).collect();

// In match patterns
match some_option {
    Some(_) => println!("Has value"),
    None => println!("No value"),
}

// In function signatures (rare)
fn process(_: impl Display) {
    // Don't care about the specific type
}
```

### Inference in Iterators

```rust
// Chain of transformations
let result: Vec<_> = vec![1, 2, 3, 4, 5]
    .into_iter()
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .collect();

// Type flows through the chain
let sum = vec![1, 2, 3]
    .iter()
    .map(|x| x * 2)        // Still iterator
    .filter(|x| x > &2)    // Still iterator
    .sum::<i32>();         // Final type specified
```

### Working with Results and Options

```rust
// Inference with Result/Option chains
let result = Some(5)
    .map(|x| x * 2)
    .filter(|x| x > &5)
    .unwrap_or(0); // Type inferred as i32

// Complex error handling
let parsed: Result<Vec<i32>, _> = 
    vec!["1", "2", "3"]
    .into_iter()
    .map(|s| s.parse())
    .collect();
```

## 🏭 Real-World Examples

### Web Handler Inference

```rust
async fn handler(req: Request) -> Result<impl Response, Error> {
    let data = fetch_data().await?;      // Type inferred
    let processed = process(data);       // Type flows through
    Ok(Json(processed))                  // Return type guides inference
}
```

### Builder Pattern

```rust
let server = ServerBuilder::new()
    .port(8080)           // Method chain preserves type
    .host("localhost")    
    .timeout(30)
    .build()             // Final type from build()
    .unwrap();
```

## 🎯 Best Practices

### 1. Let Inference Work
```rust
// Don't over-annotate
// Bad
let x: i32 = 5;
let v: Vec<i32> = vec![1, 2, 3];

// Good
let x = 5;
let v = vec![1, 2, 3];
```

### 2. Annotate for Clarity
```rust
// Annotate when it helps readability
let scores: HashMap<String, u32> = HashMap::new();

// Or when inference is ambiguous
let result: Result<i32, ParseIntError> = "42".parse();
```

### 3. Use Turbofish When Needed
```rust
// Clear and concise
let numbers = vec![1, 2, 3]
    .into_iter()
    .collect::<HashSet<_>>();
```

### 4. Document Non-Obvious Types
```rust
// When type isn't immediately clear
let processor = create_processor(); // Returns Box<dyn Process>
```

## 🔗 Related Topics

- [[Rust-Learning/03-Data-Types/Scalar-Types|Scalar Types]]
- [[Rust-Learning/03-Data-Types/Type-Aliases|Type Aliases]]
- [[Rust-Learning/08-Traits/README|Traits and Bounds]]
- [[Rust-Learning/09-Lifetimes/README|Lifetime Inference]]

---
#rust #types #type-inference #compiler #type-system
