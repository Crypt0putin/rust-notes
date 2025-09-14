# 📦 Compound Types in Rust

## 📋 Overview

Compound types can group multiple values into one type. Rust has two primitive compound types: **tuples** and **arrays**.

## 🎯 Tuples

### Basic Tuple Usage

```rust
// Creating tuples
let tup: (i32, f64, u8) = (500, 6.4, 1);

// Type inference
let tup = (500, 6.4, 1);

// Destructuring
let (x, y, z) = tup;
println!("The value of y is: {}", y);

// Accessing by index
let five_hundred = tup.0;
let six_point_four = tup.1;
let one = tup.2;

// Unit type - empty tuple
let unit: () = ();
```

### Tuple Patterns

```rust
// Nested tuples
let nested = ((1, 2), (3, 4), 5);
let ((a, b), (c, d), e) = nested;

// Partial destructuring with _
let (first, _, third) = (1, 2, 3);

// Using .. to ignore rest
let (first, ..) = (1, 2, 3, 4, 5);
let (.., last) = (1, 2, 3, 4, 5);
let (first, .., last) = (1, 2, 3, 4, 5);
```

### Tuples as Function Returns

```rust
fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length) // Return ownership and length
}

fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1);
    println!("The length of '{}' is {}.", s2, len);
}
```

## 📊 Arrays

### Fixed-Size Arrays

```rust
// Creating arrays
let a: [i32; 5] = [1, 2, 3, 4, 5];

// Type inference
let a = [1, 2, 3, 4, 5];

// Initialize with same value
let a = [3; 5]; // [3, 3, 3, 3, 3]

// Accessing elements
let first = a[0];
let second = a[1];

// Array of strings
let months = ["January", "February", "March", "April", 
              "May", "June", "July", "August", 
              "September", "October", "November", "December"];
```

### Array Operations

```rust
// Iterating over arrays
let a = [10, 20, 30, 40, 50];

for element in a {
    println!("Value: {}", element);
}

// With index
for (index, element) in a.iter().enumerate() {
    println!("Index {}: {}", index, element);
}

// Array methods
let arr = [1, 2, 3, 4, 5];
println!("Length: {}", arr.len());
println!("Is empty: {}", arr.is_empty());

// Pattern matching on arrays
match arr {
    [1, 2, ..] => println!("Starts with 1, 2"),
    [.., 4, 5] => println!("Ends with 4, 5"),
    _ => println!("Something else"),
}
```

### Multi-dimensional Arrays

```rust
// 2D array (matrix)
let matrix: [[i32; 3]; 3] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
];

// Accessing elements
let element = matrix[1][2]; // 6

// Iterating over 2D array
for row in &matrix {
    for &element in row {
        print!("{} ", element);
    }
    println!();
}

// 3D array
let cube: [[[i32; 2]; 2]; 2] = [
    [[1, 2], [3, 4]],
    [[5, 6], [7, 8]],
];
```

## 🔪 Slices

### String and Array Slices

```rust
// String slices
let s = String::from("hello world");
let hello = &s[0..5];  // "hello"
let world = &s[6..11]; // "world"

// Array slices
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3]; // [2, 3]

// Full slice
let full_slice = &a[..]; // [1, 2, 3, 4, 5]

// From beginning
let from_start = &a[..2]; // [1, 2]

// To end
let to_end = &a[2..]; // [3, 4, 5]
```

### Slice Patterns

```rust
fn analyze_slice(slice: &[i32]) {
    match slice {
        [] => println!("Empty slice"),
        [x] => println!("Single element: {}", x),
        [first, second] => println!("Two elements: {} and {}", first, second),
        [first, middle @ .., last] => {
            println!("First: {}, Middle: {:?}, Last: {}", first, middle, last)
        }
    }
}

fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    &s[..]
}
```

## 🎨 Advanced Patterns

### Tuple Structs

```rust
// Tuple structs - named tuples
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);

// Access like tuples
let r = black.0;
let x = origin.0;
```

### Array Const Generics

```rust
// Generic arrays with const generics (Rust 1.51+)
fn print_array<T: std::fmt::Debug, const N: usize>(arr: &[T; N]) {
    println!("{:?}", arr);
}

let arr1 = [1, 2, 3];
let arr2 = [1, 2, 3, 4, 5];
print_array(&arr1);
print_array(&arr2);

// Generic struct with const size
struct Matrix<T, const ROWS: usize, const COLS: usize> {
    data: [[T; COLS]; ROWS],
}
```

## 💡 Tips and Tricks

### Safe Array Access

```rust
let a = [1, 2, 3, 4, 5];

// Unsafe - can panic
// let element = a[10]; // panic!

// Safe with get()
match a.get(10) {
    Some(element) => println!("Element: {}", element),
    None => println!("Index out of bounds"),
}

// Safe with get_mut()
let mut a = [1, 2, 3];
if let Some(elem) = a.get_mut(1) {
    *elem = 10;
}
```

### Array/Slice Conversion

```rust
// Array to slice
let arr = [1, 2, 3, 4, 5];
let slice: &[i32] = &arr;
let slice: &[i32] = &arr[..];

// Vec to slice
let vec = vec![1, 2, 3, 4, 5];
let slice: &[i32] = &vec;
let slice: &[i32] = &vec[..];

// Slice to Vec
let slice = &[1, 2, 3, 4, 5];
let vec: Vec<i32> = slice.to_vec();
let vec: Vec<i32> = Vec::from(slice);
```

## 🔍 Memory Layout

```rust
use std::mem;

// Tuple memory
let tup = (1i32, 2.0f64, 3u8);
println!("Tuple size: {} bytes", mem::size_of_val(&tup));
// Usually 16 bytes (4 + 8 + 1 + padding)

// Array memory
let arr = [1, 2, 3, 4, 5];
println!("Array size: {} bytes", mem::size_of_val(&arr));
// 20 bytes (5 * 4)

// Slice reference size
let slice: &[i32] = &arr;
println!("Slice reference size: {} bytes", mem::size_of_val(&slice));
// 16 bytes on 64-bit (pointer + length)
```

## 🎯 Common Use Cases

### Returning Multiple Values
```rust
fn min_max(arr: &[i32]) -> Option<(i32, i32)> {
    if arr.is_empty() {
        return None;
    }
    
    let mut min = arr[0];
    let mut max = arr[0];
    
    for &val in arr {
        if val < min { min = val; }
        if val > max { max = val; }
    }
    
    Some((min, max))
}
```

### Fixed-Size Buffers
```rust
const BUFFER_SIZE: usize = 1024;

struct Buffer {
    data: [u8; BUFFER_SIZE],
    len: usize,
}

impl Buffer {
    fn new() -> Self {
        Buffer {
            data: [0; BUFFER_SIZE],
            len: 0,
        }
    }
    
    fn push(&mut self, byte: u8) -> Result<(), &'static str> {
        if self.len >= BUFFER_SIZE {
            return Err("Buffer full");
        }
        self.data[self.len] = byte;
        self.len += 1;
        Ok(())
    }
}
```

## 🔗 Related Topics

- [[Rust-Learning/03-Data-Types/Scalar-Types|Scalar Types]]
- [[Rust-Learning/03-Data-Types/Custom-Types|Custom Types]]
- [[Rust-Learning/06-Collections/README|Collections (Vec, HashMap)]]
- [[Rust-Learning/09-Lifetimes/Lifetime-Basics|Lifetimes with Slices]]

---
#rust #types #tuples #arrays #slices #compound-types
