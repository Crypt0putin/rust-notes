# 🚀 Продвинутые темы Rust

Сложные и специализированные возможности Rust для опытных разработчиков.

## 📋 Содержание

### Unsafe Rust
- [[Unsafe-Basics]] - Основы небезопасного кода
- [[Raw-Pointers]] - Сырые указатели
- [[Unsafe-Functions]] - Небезопасные функции
- [[External-Functions]] - Вызов внешних функций (FFI)

### Продвинутые трейты
- [[Associated-Types-Advanced]] - Продвинутые ассоциированные типы
- [[Default-Generic-Parameters]] - Параметры типов по умолчанию
- [[Operator-Overloading]] - Перегрузка операторов
- [[Supertraits]] - Супертрейты

### Продвинутые типы
- [[Newtype-Pattern]] - Паттерн Newtype
- [[Type-Aliases]] - Псевдонимы типов
- [[Never-Type]] - Тип "никогда" (!)
- [[Dynamically-Sized-Types]] - Динамически размерные типы

### Продвинутые функции
- [[Function-Pointers]] - Указатели на функции
- [[Returning-Closures]] - Возврат замыканий

## ⚠️ Unsafe Rust

### Сырые указатели
```rust
fn main() {
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}
```

### Unsafe функции
```rust
unsafe fn dangerous() {
    // Опасные операции
}

fn main() {
    unsafe {
        dangerous();
    }
}
```

### FFI (Foreign Function Interface)
```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

## 🎯 Продвинутые трейты

### Ассоциированные типы с ограничениями
```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

trait Collect<T> {
    fn collect<I: Iterator<Item = T>>(iter: I) -> Self;
}
```

### Перегрузка операторов
```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

### Супертрейты
```rust
trait OutlinePrint: std::fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

## 🔧 Продвинутые типы

### Newtype паттерн
```rust
struct Millimeters(u32);
struct Meters(u32);

impl Millimeters {
    fn to_meters(&self) -> Meters {
        Meters(self.0 / 1000)
    }
}
```

### Never type
```rust
fn bar() -> ! {
    panic!("This function never returns!");
}

fn main() {
    let guess = "3";
    let guess: u32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => panic!("Not a number!"), // ! может приводиться к любому типу
    };
}
```

### Динамически размерные типы
```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}

// Эквивалентно:
fn generic<T>(t: &T) where T: ?Sized {
    // --snip--
}
```

## 🎭 Функции высшего порядка

### Указатели на функции
```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer);
}
```

### Возврат замыканий
```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}

fn main() {
    let f = returns_closure();
    println!("{}", f(1));
}
```

## 🔬 Макросы для продвинутых

### Условная компиляция
```rust
#[cfg(target_os = "linux")]
fn are_you_on_linux() {
    println!("You are running linux!");
}

#[cfg(not(target_os = "linux"))]
fn are_you_on_linux() {
    println!("You are *not* running linux!");
}

// Атрибуты для различных целей
#[cfg(feature = "debug")]
fn debug_function() {
    println!("Debug mode");
}
```

### Документационные тесты
```rust
/// This function adds two numbers
/// 
/// # Examples
/// 
/// ```
/// let result = my_crate::add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## 🎯 Паттерны проектирования

### Builder с типами состояний
```rust
struct Waiting;
struct Ready;

struct Request<State = Waiting> {
    message: String,
    _state: std::marker::PhantomData<State>,
}

impl Request<Waiting> {
    fn new(message: String) -> Request<Waiting> {
        Request {
            message,
            _state: std::marker::PhantomData,
        }
    }
    
    fn ready(self) -> Request<Ready> {
        Request {
            message: self.message,
            _state: std::marker::PhantomData,
        }
    }
}

impl Request<Ready> {
    fn send(self) {
        println!("Sending: {}", self.message);
    }
}
```

### Zero-cost abstractions
```rust
// Высокоуровневый код компилируется в эффективный машинный код
let numbers: Vec<i32> = (0..1_000_000).collect();
let sum: i32 = numbers
    .iter()
    .map(|x| x * x)
    .filter(|&x| x % 2 == 0)
    .sum();

// Компилируется примерно в:
let mut sum = 0;
for i in 0..1_000_000 {
    let square = i * i;
    if square % 2 == 0 {
        sum += square;
    }
}
```

## 💡 Best Practices

### 1. Минимизируйте unsafe код
```rust
// Изолируйте unsafe в маленьких функциях
fn get_at_index(slice: &[i32], index: usize) -> Option<i32> {
    if index < slice.len() {
        unsafe {
            Some(*slice.get_unchecked(index))
        }
    } else {
        None
    }
}
```

### 2. Используйте type-driven development
```rust
// Пусть система типов помогает в корректности
#[derive(Debug)]
enum State {
    Loading,
    Loaded(Data),
    Error(String),
}

fn handle_state(state: State) {
    match state {
        State::Loading => println!("Still loading..."),
        State::Loaded(data) => process(data),
        State::Error(msg) => eprintln!("Error: {}", msg),
    }
}
```

## ✅ Чек-лист изучения
- [ ] Понимаю когда и как использовать unsafe
- [ ] Работаю с продвинутыми трейтами
- [ ] Применяю продвинутые паттерны типов
- [ ] Использую zero-cost abstractions
- [ ] Знаю принципы безопасного дизайна API

#advanced #unsafe #ffi #zero-cost #type-system #expert