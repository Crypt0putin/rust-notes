# 📦 Тип Result<T, E>

Result - это enum для обработки ошибок в Rust, представляющий либо успешный результат, либо ошибку.

## 🎯 Определение Result

### Базовое определение
```rust
enum Result<T, E> {
    Ok(T),    // Успешный результат типа T
    Err(E),   // Ошибка типа E
}
```

### Простой пример
```rust
fn divide(dividend: f64, divisor: f64) -> Result<f64, String> {
    if divisor == 0.0 {
        Err("Cannot divide by zero".to_string())
    } else {
        Ok(dividend / divisor)
    }
}

fn main() {
    match divide(10.0, 2.0) {
        Ok(result) => println!("Result: {}", result),
        Err(error) => println!("Error: {}", error),
    }
}
```

## 🔧 Методы Result

### Проверка содержимого
```rust
let success: Result<i32, &str> = Ok(42);
let failure: Result<i32, &str> = Err("Something went wrong");

// Проверка типа результата
println!("Is success OK? {}", success.is_ok());     // true
println!("Is success Err? {}", success.is_err());   // false
println!("Is failure OK? {}", failure.is_ok());     // false
println!("Is failure Err? {}", failure.is_err());   // true
```

### Извлечение значений
```rust
let success: Result<i32, &str> = Ok(42);
let failure: Result<i32, &str> = Err("error");

// unwrap() - паника при ошибке
let value = success.unwrap(); // 42
// let bad_value = failure.unwrap(); // ❌ panic!

// expect() - паника с пользовательским сообщением
let value = success.expect("This should not fail"); // 42
// let bad_value = failure.expect("Custom panic message"); // ❌ panic!

// unwrap_or() - значение по умолчанию
let value = success.unwrap_or(0); // 42
let default = failure.unwrap_or(0); // 0

// unwrap_or_else() - вычисление значения по умолчанию
let value = failure.unwrap_or_else(|err| {
    println!("Error occurred: {}", err);
    -1
}); // -1
```

## 🔄 Трансформация Result

### map() - преобразование успешного значения
```rust
fn parse_and_double(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse::<i32>().map(|n| n * 2)
}

// Использование
let result = parse_and_double("21");
match result {
    Ok(value) => println!("Doubled: {}", value), // 42
    Err(e) => println!("Parse error: {}", e),
}
```

### map_err() - преобразование ошибки
```rust
fn parse_with_custom_error(s: &str) -> Result<i32, String> {
    s.parse::<i32>()
        .map_err(|e| format!("Failed to parse '{}': {}", s, e))
}

// Использование
let result = parse_with_custom_error("not_a_number");
match result {
    Ok(value) => println!("Parsed: {}", value),
    Err(e) => println!("Custom error: {}", e),
}
```

### and_then() - цепочка операций
```rust
fn divide_strings(dividend: &str, divisor: &str) -> Result<f64, String> {
    dividend.parse::<f64>()
        .map_err(|e| format!("Invalid dividend: {}", e))
        .and_then(|d| {
            divisor.parse::<f64>()
                .map_err(|e| format!("Invalid divisor: {}", e))
                .and_then(|div| {
                    if div == 0.0 {
                        Err("Division by zero".to_string())
                    } else {
                        Ok(d / div)
                    }
                })
        })
}

// Использование
match divide_strings("10.5", "2.0") {
    Ok(result) => println!("Result: {}", result),
    Err(e) => println!("Error: {}", e),
}
```

## ⚡ Оператор `?`

### Основы оператора `?`
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_content(path: &str) -> Result<String, io::Error> {
    let mut file = File::open(path)?;  // Возвращает ошибку, если не удалось
    let mut content = String::new();
    file.read_to_string(&mut content)?; // Возвращает ошибку, если не удалось
    Ok(content)
}

// Эквивалентно:
fn read_file_content_verbose(path: &str) -> Result<String, io::Error> {
    let mut file = match File::open(path) {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    
    let mut content = String::new();
    match file.read_to_string(&mut content) {
        Ok(_) => Ok(content),
        Err(e) => Err(e),
    }
}
```

### `?` в main функции
```rust
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let content = std::fs::read_to_string("config.txt")?;
    println!("Config: {}", content);
    Ok(())
}
```

## 🎯 Практические паттерны

### Множественные типы ошибок
```rust
use std::num::ParseIntError;
use std::io;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
    Custom(String),
}

impl From<io::Error> for AppError {
    fn from(err: io::Error) -> AppError {
        AppError::Io(err)
    }
}

impl From<ParseIntError> for AppError {
    fn from(err: ParseIntError) -> AppError {
        AppError::Parse(err)
    }
}

fn process_file(path: &str) -> Result<i32, AppError> {
    let content = std::fs::read_to_string(path)?; // io::Error -> AppError
    let number = content.trim().parse()?; // ParseIntError -> AppError
    
    if number < 0 {
        return Err(AppError::Custom("Number must be positive".to_string()));
    }
    
    Ok(number)
}
```

### Сбор Results
```rust
fn parse_numbers(strings: Vec<&str>) -> Result<Vec<i32>, std::num::ParseIntError> {
    strings.iter().map(|s| s.parse::<i32>()).collect()
}

// Использование
let strings = vec!["1", "2", "3", "4"];
match parse_numbers(strings) {
    Ok(numbers) => println!("Parsed: {:?}", numbers),
    Err(e) => println!("Parse error: {}", e),
}

// Игнорирование ошибок
fn parse_valid_numbers(strings: Vec<&str>) -> Vec<i32> {
    strings.iter()
        .filter_map(|s| s.parse().ok())
        .collect()
}
```

### Комбинирование Results
```rust
fn add_numbers(a: &str, b: &str) -> Result<i32, std::num::ParseIntError> {
    let num_a = a.parse::<i32>()?;
    let num_b = b.parse::<i32>()?;
    Ok(num_a + num_b)
}

// Или с комбинаторами:
fn add_numbers_combinators(a: &str, b: &str) -> Result<i32, std::num::ParseIntError> {
    a.parse::<i32>().and_then(|num_a|
        b.parse::<i32>().map(|num_b| num_a + num_b)
    )
}
```

## 🔄 Конвертация между Result и Option

### Result → Option
```rust
let result: Result<i32, &str> = Ok(42);
let option: Option<i32> = result.ok(); // Some(42)

let result: Result<i32, &str> = Err("error");
let option: Option<i32> = result.ok(); // None
```

### Option → Result
```rust
let option: Option<i32> = Some(42);
let result: Result<i32, &str> = option.ok_or("No value");

let option: Option<i32> = None;
let result: Result<i32, String> = option.ok_or_else(|| "No value found".to_string());
```

### transpose() - переключение вложенности
```rust
let option_result: Option<Result<i32, &str>> = Some(Ok(42));
let result_option: Result<Option<i32>, &str> = option_result.transpose();
// Ok(Some(42))

let option_result: Option<Result<i32, &str>> = Some(Err("error"));
let result_option: Result<Option<i32>, &str> = option_result.transpose();
// Err("error")
```

## 🎨 Продвинутые техники

### Result с пользовательскими типами
```rust
#[derive(Debug)]
struct User {
    id: u32,
    name: String,
}

#[derive(Debug)]
enum UserError {
    NotFound(u32),
    InvalidName,
    DatabaseError(String),
}

fn find_user(id: u32) -> Result<User, UserError> {
    if id == 0 {
        return Err(UserError::InvalidName);
    }
    
    if id > 1000 {
        return Err(UserError::NotFound(id));
    }
    
    Ok(User {
        id,
        name: format!("User {}", id),
    })
}

// Использование
match find_user(42) {
    Ok(user) => println!("Found user: {:?}", user),
    Err(UserError::NotFound(id)) => println!("User {} not found", id),
    Err(UserError::InvalidName) => println!("Invalid user name"),
    Err(UserError::DatabaseError(msg)) => println!("Database error: {}", msg),
}
```

### Early return паттерн
```rust
fn complex_operation(data: &str) -> Result<String, String> {
    if data.is_empty() {
        return Err("Data cannot be empty".to_string());
    }
    
    if data.len() < 5 {
        return Err("Data too short".to_string());
    }
    
    if !data.chars().all(|c| c.is_alphanumeric()) {
        return Err("Data contains invalid characters".to_string());
    }
    
    // Все проверки пройдены
    Ok(data.to_uppercase())
}
```

### Result в итераторах
```rust
fn process_numbers(strings: Vec<&str>) -> Result<Vec<i32>, String> {
    strings
        .iter()
        .map(|s| s.parse::<i32>().map_err(|e| e.to_string()))
        .collect()
}

// Или с filter_map для игнорирования ошибок:
fn process_valid_numbers(strings: Vec<&str>) -> Vec<i32> {
    strings
        .iter()
        .filter_map(|s| s.parse().ok())
        .collect()
}
```

## 💡 Лучшие практики

### 1. Используйте значимые типы ошибок
```rust
// ❌ Плохо - общий String
fn parse_config(content: &str) -> Result<Config, String> { ... }

// ✅ Хорошо - специфичный тип ошибки
fn parse_config(content: &str) -> Result<Config, ConfigError> { ... }
```

### 2. Предпочитайте `?` вместо match для простых случаев
```rust
// ❌ Многословно
fn read_number() -> Result<i32, std::io::Error> {
    let content = match std::fs::read_to_string("number.txt") {
        Ok(content) => content,
        Err(e) => return Err(e),
    };
    
    match content.trim().parse() {
        Ok(num) => Ok(num),
        Err(_) => Err(std::io::Error::new(
            std::io::ErrorKind::InvalidData,
            "Invalid number format"
        )),
    }
}

// ✅ Лаконично с ?
fn read_number() -> Result<i32, Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("number.txt")?;
    let number = content.trim().parse()?;
    Ok(number)
}
```

### 3. Документируйте возможные ошибки
```rust
/// Reads a configuration file and parses it.
/// 
/// # Errors
/// 
/// Returns `ConfigError::FileNotFound` if the file doesn't exist.
/// Returns `ConfigError::ParseError` if the file has invalid syntax.
/// Returns `ConfigError::ValidationError` if the values are invalid.
fn load_config(path: &str) -> Result<Config, ConfigError> {
    // ...
}
```

## 🔗 Связанные темы
- [[Option-Type]] - Тип Option для nullable значений
- [[Error-Propagation]] - Распространение ошибок с `?`
- [[Custom-Error-Types]] - Создание собственных типов ошибок
- [[Error-Trait]] - Стандартный трейт Error

#result-type #error-handling #option #unwrap #question-mark #intermediate