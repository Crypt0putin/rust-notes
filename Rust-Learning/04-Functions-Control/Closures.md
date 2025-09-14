# 🔗 Closures in Rust

## 📋 Overview

Closures are anonymous functions that can capture values from their enclosing scope. They're similar to lambdas in other languages.

## 🎯 Basic Closure Syntax

### Simple Closures

```rust
fn main() {
    // Basic closure
    let add_one = |x| x + 1;
    let result = add_one(5); // 6
    
    // With type annotations
    let add_one: fn(i32) -> i32 = |x: i32| -> i32 { x + 1 };
    
    // Multiple parameters
    let add = |x, y| x + y;
    let sum = add(2, 3); // 5
    
    // No parameters
    let greet = || println!("Hello!");
    greet();
    
    // Multi-line closure
    let complex = |x: i32| {
        let y = x * 2;
        let z = y + 1;
        z * z
    };
}
```

### Type Inference in Closures

```rust
fn main() {
    // Type inferred from first use
    let example_closure = |x| x;
    
    let s = example_closure(String::from("hello"));
    // let n = example_closure(5); // Error! x already inferred as String
    
    // Generic-like behavior requires explicit functions
    fn identity<T>(x: T) -> T { x }
    
    let s = identity(String::from("hello"));
    let n = identity(5); // Works!
}
```

## 📦 Capturing the Environment

### Capture Modes

```rust
fn main() {
    let x = 4;
    
    // Immutable borrow
    let equal_to_x = |z| z == x;
    
    let y = 4;
    assert!(equal_to_x(y));
    println!("x is still accessible: {}", x);
    
    // Mutable borrow
    let mut x = 5;
    let mut add_to_x = |y| x += y;
    
    add_to_x(3);
    // println!("{}", x); // Error! x is mutably borrowed
    
    // After closure is done, x is accessible
    assert_eq!(x, 8);
    
    // Move ownership
    let s = String::from("hello");
    let consume = move || println!("s: {}", s);
    
    consume();
    // println!("{}", s); // Error! s was moved
}
```

### The move Keyword

```rust
fn main() {
    let x = vec![1, 2, 3];
    
    // Without move - borrows x
    let print_vec = || println!("vec: {:?}", x);
    print_vec();
    println!("x is still here: {:?}", x);
    
    // With move - takes ownership
    let x = vec![1, 2, 3];
    let owns_vec = move || println!("vec: {:?}", x);
    owns_vec();
    // println!("{:?}", x); // Error! x was moved
    
    // Useful for threads
    use std::thread;
    let v = vec![1, 2, 3];
    
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });
    
    handle.join().unwrap();
}
```

## 🔧 Closure Traits

### Fn, FnMut, and FnOnce

```rust
// FnOnce - consumes captured values
fn apply_once<F>(f: F)
where
    F: FnOnce(),
{
    f();
    // f(); // Error! Can only call once
}

// FnMut - mutably borrows values
fn apply_mut<F>(mut f: F)
where
    F: FnMut(),
{
    f();
    f(); // Can call multiple times
}

// Fn - immutably borrows values
fn apply<F>(f: F)
where
    F: Fn(),
{
    f();
    f(); // Can call multiple times
}

fn main() {
    let s = String::from("hello");
    
    // FnOnce - moves s
    let consume = move || {
        println!("{}", s);
        drop(s); // Explicitly consume s
    };
    apply_once(consume);
    
    // FnMut - mutates captured value
    let mut count = 0;
    let mut increment = || {
        count += 1;
        println!("Count: {}", count);
    };
    apply_mut(increment);
    
    // Fn - only reads captured value
    let text = String::from("hello");
    let print = || println!("{}", text);
    apply(print);
}
```

## 🎨 Closures as Parameters

### Using Trait Bounds

```rust
// Generic function accepting closure
fn map_vec<F, T, U>(vec: Vec<T>, f: F) -> Vec<U>
where
    F: Fn(T) -> U,
{
    let mut result = Vec::new();
    for item in vec {
        result.push(f(item));
    }
    result
}

// Using impl Trait
fn apply_twice(f: impl Fn(i32) -> i32, x: i32) -> i32 {
    f(f(x))
}

// With lifetime parameters
fn apply_to_str<'a, F>(f: F, s: &'a str) -> String
where
    F: Fn(&'a str) -> String,
{
    f(s)
}

fn main() {
    let numbers = vec![1, 2, 3];
    let doubled = map_vec(numbers, |x| x * 2);
    println!("{:?}", doubled); // [2, 4, 6]
    
    let result = apply_twice(|x| x + 1, 5); // 7
}
```

### Returning Closures

```rust
// Return closure with impl Trait
fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    move |y| x + y
}

// Return closure with Box
fn make_transformer(factor: i32) -> Box<dyn Fn(i32) -> i32> {
    Box::new(move |x| x * factor)
}

// Factory pattern
fn create_validator(min: i32, max: i32) -> impl Fn(i32) -> bool {
    move |value| value >= min && value <= max
}

fn main() {
    let add_5 = make_adder(5);
    println!("{}", add_5(10)); // 15
    
    let multiply_by_3 = make_transformer(3);
    println!("{}", multiply_by_3(4)); // 12
    
    let is_valid = create_validator(0, 100);
    println!("{}", is_valid(50)); // true
}
```

## 🔄 Closures with Iterators

### Common Iterator Methods

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // map
    let doubled: Vec<_> = numbers.iter().map(|x| x * 2).collect();
    
    // filter
    let evens: Vec<_> = numbers.iter().filter(|&&x| x % 2 == 0).collect();
    
    // fold/reduce
    let sum = numbers.iter().fold(0, |acc, x| acc + x);
    
    // find
    let first_even = numbers.iter().find(|&&x| x % 2 == 0);
    
    // any/all
    let has_negative = numbers.iter().any(|&x| x < 0);
    let all_positive = numbers.iter().all(|&x| x > 0);
    
    // Complex chaining
    let result: Vec<_> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .filter(|&x| x > 10)
        .collect();
}
```

### Custom Iterator Adapters

```rust
trait IteratorExt: Iterator {
    fn every_other(self) -> EveryOther<Self>
    where
        Self: Sized,
    {
        EveryOther {
            iter: self,
            state: true,
        }
    }
}

impl<I: Iterator> IteratorExt for I {}

struct EveryOther<I> {
    iter: I,
    state: bool,
}

impl<I> Iterator for EveryOther<I>
where
    I: Iterator,
{
    type Item = I::Item;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.state {
            self.state = false;
            self.iter.next()
        } else {
            self.state = true;
            self.iter.next();
            self.iter.next()
        }
    }
}
```

## 💡 Advanced Patterns

### Memoization with Closures

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }
    
    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}

fn main() {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        std::thread::sleep(std::time::Duration::from_secs(2));
        num
    });
    
    println!("{}", expensive_result.value(5));
    println!("{}", expensive_result.value(5)); // Cached!
}
```

### Event Handlers

```rust
struct Button {
    handlers: Vec<Box<dyn Fn()>>,
}

impl Button {
    fn new() -> Self {
        Button {
            handlers: Vec::new(),
        }
    }
    
    fn on_click(&mut self, handler: impl Fn() + 'static) {
        self.handlers.push(Box::new(handler));
    }
    
    fn click(&self) {
        for handler in &self.handlers {
            handler();
        }
    }
}

fn main() {
    let mut button = Button::new();
    
    let message = String::from("Clicked!");
    button.on_click(move || println!("{}", message));
    button.on_click(|| println!("Another handler"));
    
    button.click();
}
```

## 🎯 Best Practices

### 1. Prefer move for Thread Safety
```rust
let data = Arc::new(Mutex::new(vec![1, 2, 3]));
let data_clone = Arc::clone(&data);

thread::spawn(move || {
    let mut data = data_clone.lock().unwrap();
    data.push(4);
});
```

### 2. Use Type Inference Wisely
```rust
// Let compiler infer when obvious
let add_one = |x| x + 1;

// Be explicit when needed for clarity
let complex: Box<dyn Fn(i32) -> Result<i32, Error>> = Box::new(|x| {
    // Complex logic
    Ok(x)
});
```

### 3. Consider Performance
```rust
// Avoid unnecessary allocations
let process = |s: &str| s.to_uppercase(); // Better than String parameter

// Use references when possible
let data = vec![1, 2, 3];
let sum = data.iter().fold(0, |acc, &x| acc + x);
```

## 🔗 Related Topics

- [[Rust-Learning/04-Functions-Control/Function-Definition|Functions]]
- [[Rust-Learning/04-Functions-Control/Iterator-Patterns|Iterator Patterns]]
- [[Rust-Learning/08-Traits/README|Fn Traits]]
- [[Rust-Learning/02-Ownership/README|Ownership and Borrowing]]

---
#rust #closures #lambdas #functional-programming #capture
