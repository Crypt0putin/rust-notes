# 🔀 Control Flow in Rust

## 📋 Overview

Control flow determines the order in which code executes. Rust provides several constructs for controlling program flow.

## 🎯 if Expressions

### Basic if/else

```rust
fn main() {
    let number = 7;
    
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}

// No parentheses needed around condition
let x = 5;
if x > 0 {  // Not if (x > 0)
    println!("positive");
}
```

### else if Chains

```rust
fn main() {
    let number = 6;
    
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

### if as an Expression

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };
    
    println!("The value of number is: {}", number);
    
    // Can be used in more complex expressions
    let result = if number > 3 {
        number * 2
    } else {
        number + 10
    };
    
    // Both branches must return the same type
    // let result = if condition { 5 } else { "six" }; // Error!
}
```

## 🔁 Loops

### loop - Infinite Loop

```rust
fn main() {
    // Basic infinite loop
    loop {
        println!("again!");
        break; // Exit the loop
    }
    
    // Returning values from loop
    let mut counter = 0;
    let result = loop {
        counter += 1;
        
        if counter == 10 {
            break counter * 2; // Return value with break
        }
    };
    
    println!("The result is {}", result); // 20
}
```

### Loop Labels

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;
        
        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break; // Breaks inner loop
            }
            if count == 2 {
                break 'counting_up; // Breaks outer loop
            }
            remaining -= 1;
        }
        
        count += 1;
    }
    println!("End count = {}", count);
}
```

### while - Conditional Loop

```rust
fn main() {
    let mut number = 3;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("LIFTOFF!!!");
    
    // while with complex conditions
    let mut x = 5;
    let mut done = false;
    
    while !done && x < 100 {
        x *= 2;
        if x > 50 {
            done = true;
        }
    }
}
```

### for - Iterator Loop

```rust
fn main() {
    // Iterating over array
    let a = [10, 20, 30, 40, 50];
    
    for element in a {
        println!("the value is: {}", element);
    }
    
    // With iterator methods
    for element in a.iter() {
        println!("the value is: {}", element);
    }
    
    // Range iteration
    for number in 1..4 {
        println!("{}!", number); // 1, 2, 3
    }
    
    // Inclusive range
    for number in 1..=3 {
        println!("{}!", number); // 1, 2, 3
    }
    
    // Reverse iteration
    for number in (1..4).rev() {
        println!("{}!", number); // 3, 2, 1
    }
    
    // With index
    for (index, value) in a.iter().enumerate() {
        println!("Index {}: {}", index, value);
    }
}
```

## 🎲 Pattern Matching

### match Expression

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

// match must be exhaustive
fn process_option(x: Option<i32>) {
    match x {
        Some(value) => println!("Value: {}", value),
        None => println!("No value"),
    }
}
```

### match with Multiple Patterns

```rust
fn main() {
    let x = 1;
    
    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        4..=10 => println!("four through ten"),
        _ => println!("anything else"),
    }
    
    // Matching with guards
    let num = Some(4);
    
    match num {
        Some(x) if x < 5 => println!("less than five: {}", x),
        Some(x) => println!("{}", x),
        None => (),
    }
}
```

### Destructuring in match

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    
    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
    
    // Ignoring values
    let numbers = (2, 4, 8, 16, 32);
    
    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {}, {}, {}", first, third, fifth)
        }
    }
}
```

## 🎯 if let and while let

### if let - Simpler match

```rust
fn main() {
    let config_max = Some(3u8);
    
    // Instead of match
    match config_max {
        Some(max) => println!("Maximum is {}", max),
        _ => (),
    }
    
    // Use if let
    if let Some(max) = config_max {
        println!("Maximum is {}", max);
    }
    
    // With else
    let mut count = 0;
    let coin = Coin::Quarter;
    
    if let Coin::Quarter = coin {
        println!("State quarter!");
    } else {
        count += 1;
    }
}
```

### while let - Loop with Pattern

```rust
fn main() {
    let mut stack = Vec::new();
    
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
    
    // Processing optional values
    let mut optional = Some(0);
    
    while let Some(i) = optional {
        if i > 9 {
            println!("Greater than 9, quit!");
            optional = None;
        } else {
            println!("`i` is {}. Try again.", i);
            optional = Some(i + 1);
        }
    }
}
```

## 🔀 Advanced Control Flow

### Early Returns

```rust
fn find_first_even(numbers: &[i32]) -> Option<i32> {
    for &num in numbers {
        if num % 2 == 0 {
            return Some(num); // Early return
        }
    }
    None
}

fn validate_age(age: i32) -> Result<(), String> {
    if age < 0 {
        return Err("Age cannot be negative".to_string());
    }
    if age > 150 {
        return Err("Age seems unrealistic".to_string());
    }
    Ok(())
}
```

### continue and break

```rust
fn main() {
    // Using continue
    for i in 0..10 {
        if i % 2 == 0 {
            continue; // Skip even numbers
        }
        println!("{}", i);
    }
    
    // Using break with value
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    
    // Labeled break and continue
    'outer: for x in 0..10 {
        'inner: for y in 0..10 {
            if x == 2 && y == 2 {
                break 'outer; // Break outer loop
            }
            if y > 5 {
                continue 'outer; // Continue outer loop
            }
            println!("({}, {})", x, y);
        }
    }
}
```

## 💡 Best Practices

### 1. Use if let for Simple Cases
```rust
// Good
if let Some(value) = optional {
    process(value);
}

// Verbose
match optional {
    Some(value) => process(value),
    None => {},
}
```

### 2. Prefer Iterator Methods
```rust
// Good
let sum: i32 = numbers.iter().sum();

// Less idiomatic
let mut sum = 0;
for num in numbers {
    sum += num;
}
```

### 3. Use Descriptive Match Arms
```rust
match status {
    Status::Success => handle_success(),
    Status::Error(code) if code >= 500 => handle_server_error(code),
    Status::Error(code) => handle_client_error(code),
    Status::Pending => wait_for_completion(),
}
```

### 4. Avoid Deeply Nested Control Flow
```rust
// Bad
if condition1 {
    if condition2 {
        if condition3 {
            // Deep nesting
        }
    }
}

// Better - early returns
if !condition1 { return; }
if !condition2 { return; }
if !condition3 { return; }
// Main logic
```

## 🎯 Common Patterns

### State Machines
```rust
enum State {
    Waiting,
    Processing,
    Complete,
}

fn process_state(state: State) -> State {
    match state {
        State::Waiting => State::Processing,
        State::Processing => State::Complete,
        State::Complete => State::Complete,
    }
}
```

### Error Propagation
```rust
fn process() -> Result<String, Error> {
    let data = read_file()?;
    let parsed = parse_data(&data)?;
    let result = transform(parsed)?;
    Ok(result)
}
```

## 🔗 Related Topics

- [[Rust-Learning/04-Functions-Control/Pattern-Matching|Pattern Matching]]
- [[Rust-Learning/04-Functions-Control/Iterator-Patterns|Iterator Patterns]]
- [[Rust-Learning/07-Error-Handling/README|Error Handling]]
- [[Rust-Learning/05-Structs-Enums/Enum-Definition|Enums and Matching]]

---
#rust #control-flow #loops #conditionals #pattern-matching
