# 📊 Scalar Types in Rust

## 🔢 Integer Types

### Signed and Unsigned Integers

| Length | Signed | Unsigned |
|--------|--------|----------|
| 8-bit  | `i8`   | `u8`     |
| 16-bit | `i16`  | `u16`    |
| 32-bit | `i32`  | `u32`    |
| 64-bit | `i64`  | `u64`    |
| 128-bit| `i128` | `u128`   |
| arch   | `isize`| `usize`  |

```rust
// Integer literals
let decimal = 98_222;      // Decimal
let hex = 0xff;           // Hexadecimal  
let octal = 0o77;         // Octal
let binary = 0b1111_0000; // Binary
let byte = b'A';          // Byte (u8 only)

// Type suffix
let x = 5u32;  // u32
let y = 10i64; // i64

// Default is i32
let default = 42; // i32
```

### Integer Overflow

```rust
// Debug mode: panic on overflow
// Release mode: wrapping

let x: u8 = 255;

// Methods for handling overflow
let wrapped = x.wrapping_add(1);      // 0
let saturated = x.saturating_add(1);  // 255
let checked = x.checked_add(1);       // None
let (result, overflowed) = x.overflowing_add(1); // (0, true)
```

## 🔢 Floating-Point Types

```rust
let x = 2.0;      // f64 (default)
let y: f32 = 3.0; // f32

// Special values
let infinity = f64::INFINITY;
let neg_infinity = f64::NEG_INFINITY;
let not_a_number = f64::NAN;

// Operations
let sum = 5.0 + 10.0;
let difference = 95.5 - 4.3;
let product = 4.0 * 30.0;
let quotient = 56.7 / 32.2;
let remainder = 43.0 % 5.0;

// Methods
let x = 2.0_f64;
let y = x.sqrt();        // Square root
let z = x.powi(3);       // Power (integer exponent)
let w = x.powf(2.5);     // Power (float exponent)
let abs = (-3.14).abs(); // Absolute value
```

## ✅ Boolean Type

```rust
let t = true;
let f: bool = false;

// Size: 1 byte
use std::mem;
println!("Size of bool: {} byte", mem::size_of::<bool>());

// Boolean operations
let and = true && false;  // false
let or = true || false;   // true
let not = !true;          // false

// In conditions
if t {
    println!("This is true!");
}
```

## 🔤 Character Type

```rust
let c = 'z';
let z: char = 'ℤ'; // Unicode character
let heart_eyed_cat = '😻'; // Emoji

// char is 4 bytes - Unicode Scalar Value
let unicode_char = '\u{1F600}'; // 😀

// Char methods
let ch = 'a';
println!("Is alphabetic: {}", ch.is_alphabetic());
println!("Is numeric: {}", ch.is_numeric());
println!("Is lowercase: {}", ch.is_lowercase());
println!("To uppercase: {}", ch.to_uppercase());

// Iterate over chars in string
for ch in "hello".chars() {
    println!("{}", ch);
}
```

## 📏 Type Sizes and Ranges

```rust
use std::mem;

println!("Type sizes:");
println!("i8:   {} byte,  range: {} to {}", 
    mem::size_of::<i8>(), i8::MIN, i8::MAX);
println!("i16:  {} bytes, range: {} to {}", 
    mem::size_of::<i16>(), i16::MIN, i16::MAX);
println!("i32:  {} bytes, range: {} to {}", 
    mem::size_of::<i32>(), i32::MIN, i32::MAX);
println!("i64:  {} bytes, range: {} to {}", 
    mem::size_of::<i64>(), i64::MIN, i64::MAX);
println!("i128: {} bytes", mem::size_of::<i128>());

println!("f32:  {} bytes", mem::size_of::<f32>());
println!("f64:  {} bytes", mem::size_of::<f64>());
println!("bool: {} byte", mem::size_of::<bool>());
println!("char: {} bytes", mem::size_of::<char>());
```

## 🔄 Type Casting

```rust
// Explicit casting with 'as'
let decimal = 65.4321_f32;
let integer = decimal as u8;        // 65
let character = integer as char;    // 'A'

// Be careful with casting
let x = 1000_i32;
let y = x as i8; // Overflow! Result depends on platform

// Safe conversions with TryFrom
use std::convert::TryFrom;

let x = 65u8;
let y = char::try_from(x).unwrap(); // 'A'

let large = 1000_i32;
match i8::try_from(large) {
    Ok(val) => println!("Converted: {}", val),
    Err(e) => println!("Conversion failed: {}", e),
}
```

## 💻 Practical Examples

### Number Processing
```rust
fn process_numbers() {
    // Parse from string
    let parsed: i32 = "42".parse().unwrap();
    let parsed_with_type = "42".parse::<i32>().unwrap();
    
    // Convert to string
    let num = 42;
    let s = num.to_string();
    
    // Binary representation
    println!("Binary: {:b}", 42);
    println!("Hex: {:x}", 42);
    println!("Octal: {:o}", 42);
}
```

### Working with Limits
```rust
fn check_limits() {
    // Check for overflow before operation
    let x = i32::MAX;
    
    match x.checked_add(1) {
        Some(result) => println!("Result: {}", result),
        None => println!("Operation would overflow!"),
    }
    
    // Saturating arithmetic for safe boundaries
    let score = 100_u8;
    let bonus = 50_u8;
    let total = score.saturating_add(bonus); // Caps at 255
}
```

## 🎯 Best Practices

1. **Use explicit types when needed**
   ```rust
   let x = 5_i64; // Be explicit about size
   ```

2. **Handle overflow explicitly**
   ```rust
   value.checked_add(1).expect("Overflow occurred");
   ```

3. **Use type inference when obvious**
   ```rust
   let vec = vec![1, 2, 3]; // Clearly Vec<i32>
   ```

4. **Prefer usize for indexing**
   ```rust
   let index: usize = 5;
   let element = array[index];
   ```

## 🔗 Related Topics

- [[Rust-Learning/03-Data-Types/Compound-Types|Compound Types]]
- [[Rust-Learning/03-Data-Types/Type-Aliases|Type Aliases]]
- [[Rust-Learning/03-Data-Types/Type-Inference|Type Inference]]
- [[Rust-Learning/02-Ownership/README|Ownership System]]

---
#rust #types #scalars #integers #floats #bool #char
