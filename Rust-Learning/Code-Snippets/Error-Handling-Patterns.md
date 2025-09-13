# 🚨 Паттерны обработки ошибок

Эффективные техники работы с Result и Option в Rust.

## 🎯 Базовые паттерны Result

### Цепочки операций с ?
```rust
use std::fs;
use std::io;

fn read_and_process_file(path: &str) -> Result<String, io::Error> {
    let content = fs::read_to_string(path)?;
    let processed = content.trim().to_uppercase();
    Ok(processed)
}

// Несколько операций подряд
fn complex_operation(path: &str) -> Result<usize, io::Error> {
    let content = fs::read_to_string(path)?;
    let lines = content.lines().count();
    
    // Условная ошибка
    if lines == 0 {
        return Err(io::Error::new(io::ErrorKind::InvalidData, "File is empty"));
    }
    
    Ok(lines)
}
```

### Map и map_err
```rust
use std::num::ParseIntError;

fn parse_and_double(s: &str) -> Result<i32, ParseIntError> {
    s.parse::<i32>().map(|n| n * 2)
}

fn parse_with_custom_error(s: &str) -> Result<i32, String> {
    s.parse::<i32>()
        .map_err(|e| format!("Failed to parse '{}': {}", s, e))
}

// Цепочка map операций
fn process_number(s: &str) -> Result<String, ParseIntError> {
    s.parse::<i32>()
        .map(|n| n * 2)
        .map(|n| n + 1)
        .map(|n| format!("Result: {}", n))
}
```

### And_then для цепочек Result
```rust
fn divide_string(s: &str) -> Result<f64, String> {
    let parts: Vec<&str> = s.split('/').collect();
    
    if parts.len() != 2 {
        return Err("Input must contain exactly one '/'".to_string());
    }
    
    parts[0].parse::<f64>()
        .map_err(|e| format!("Invalid numerator: {}", e))
        .and_then(|num| {
            parts[1].parse::<f64>()
                .map_err(|e| format!("Invalid denominator: {}", e))
                .and_then(|denom| {
                    if denom == 0.0 {
                        Err("Division by zero".to_string())
                    } else {
                        Ok(num / denom)
                    }
                })
        })
}
```

## 🔧 Продвинутые техники

### Коллекционирование Results
```rust
fn parse_all_numbers(strings: Vec<&str>) -> Result<Vec<i32>, String> {
    strings
        .iter()
        .map(|s| s.parse::<i32>().map_err(|e| e.to_string()))
        .collect() // Останавливается на первой ошибке
}

// Сбор всех ошибок
fn parse_all_with_errors(strings: Vec<&str>) -> (Vec<i32>, Vec<String>) {
    strings
        .iter()
        .map(|s| s.parse::<i32>().map_err(|e| e.to_string()))
        .fold((Vec::new(), Vec::new()), |(mut oks, mut errs), result| {
            match result {
                Ok(val) => oks.push(val),
                Err(e) => errs.push(e),
            }
            (oks, errs)
        })
}

// Игнорирование ошибок
fn parse_valid_numbers(strings: Vec<&str>) -> Vec<i32> {
    strings
        .iter()
        .filter_map(|s| s.parse().ok())
        .collect()
}
```

### Конвертация типов ошибок
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

fn read_number_from_file(path: &str) -> Result<i32, AppError> {
    let content = std::fs::read_to_string(path)?; // io::Error -> AppError
    let number = content.trim().parse()?; // ParseIntError -> AppError
    
    if number < 0 {
        return Err(AppError::Custom("Number must be positive".to_string()));
    }
    
    Ok(number)
}
```

## 💡 Паттерны с Option

### Цепочки Option
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: Option<u32>,
    address: Option<Address>,
}

#[derive(Debug)]
struct Address {
    street: String,
    city: String,
    postal_code: Option<String>,
}

fn get_postal_code(person: &Person) -> Option<&String> {
    person.address.as_ref()?.postal_code.as_ref()
}

// Map и and_then с Option
fn process_age(person: &Person) -> Option<String> {
    person.age
        .map(|age| age + 1) // Увеличиваем возраст
        .and_then(|age| {   // Проверяем диапазон
            if age > 120 {
                None
            } else {
                Some(format!("Next year you'll be {}", age))
            }
        })
}
```

### Option и Result вместе
```rust
fn find_and_parse(items: &[&str], target: &str) -> Result<Option<i32>, ParseIntError> {
    items
        .iter()
        .find(|&&item| item.starts_with(target))
        .map(|item| item[target.len()..].parse::<i32>())
        .transpose() // Option<Result<T, E>> -> Result<Option<T>, E>
}

// Обратная конвертация
fn parse_optional(s: Option<&str>) -> Option<Result<i32, ParseIntError>> {
    s.map(|s| s.parse::<i32>())
}
```

## 🎯 Паттерны обработки

### Early return pattern
```rust
fn validate_user_input(input: &str) -> Result<String, String> {
    if input.is_empty() {
        return Err("Input cannot be empty".to_string());
    }
    
    if input.len() < 3 {
        return Err("Input must be at least 3 characters".to_string());
    }
    
    if !input.chars().all(|c| c.is_alphanumeric()) {
        return Err("Input must contain only alphanumeric characters".to_string());
    }
    
    Ok(input.to_uppercase())
}
```

### Match с множественными ошибками
```rust
use std::fs;
use std::io::ErrorKind;

fn read_config_file(path: &str) -> Result<String, String> {
    match fs::read_to_string(path) {
        Ok(content) => {
            if content.trim().is_empty() {
                Err("Config file is empty".to_string())
            } else {
                Ok(content)
            }
        }
        Err(error) => match error.kind() {
            ErrorKind::NotFound => {
                // Создаём файл с настройками по умолчанию
                let default_config = "default=true\n";
                fs::write(path, default_config)
                    .map_err(|e| format!("Failed to create default config: {}", e))?;
                Ok(default_config.to_string())
            }
            ErrorKind::PermissionDenied => {
                Err("Permission denied to read config file".to_string())
            }
            _ => Err(format!("Failed to read config file: {}", error)),
        }
    }
}
```

## 🔄 Retry логика

### Простой retry
```rust
use std::thread;
use std::time::Duration;

fn retry_operation<F, T, E>(mut operation: F, max_attempts: u32) -> Result<T, E>
where
    F: FnMut() -> Result<T, E>,
{
    for attempt in 1..=max_attempts {
        match operation() {
            Ok(result) => return Ok(result),
            Err(e) => {
                if attempt == max_attempts {
                    return Err(e);
                }
                thread::sleep(Duration::from_millis(100 * attempt as u64));
            }
        }
    }
    unreachable!()
}

// Использование
fn unreliable_network_call() -> Result<String, String> {
    // Имитация ненадёжного сетевого вызова
    if rand::random::<f64>() < 0.7 {
        Err("Network error".to_string())
    } else {
        Ok("Success!".to_string())
    }
}

fn main() {
    let result = retry_operation(unreliable_network_call, 3);
    println!("{:?}", result);
}
```

### Экспоненциальный backoff
```rust
use std::time::Duration;

fn retry_with_backoff<F, T, E>(
    mut operation: F, 
    max_attempts: u32,
    initial_delay: Duration,
) -> Result<T, E>
where
    F: FnMut() -> Result<T, E>,
{
    let mut delay = initial_delay;
    
    for attempt in 1..=max_attempts {
        match operation() {
            Ok(result) => return Ok(result),
            Err(e) => {
                if attempt == max_attempts {
                    return Err(e);
                }
                
                thread::sleep(delay);
                delay = delay * 2; // Экспоненциальное увеличение
            }
        }
    }
    unreachable!()
}
```

## 🎨 Пользовательские Result типы

### Typed Result
```rust
type DatabaseResult<T> = Result<T, DatabaseError>;
type NetworkResult<T> = Result<T, NetworkError>;
type ParseResult<T> = Result<T, ParseError>;

#[derive(Debug)]
enum DatabaseError {
    ConnectionLost,
    QueryFailed(String),
    InvalidSchema,
}

// Использование
fn find_user(id: u32) -> DatabaseResult<User> {
    if id == 0 {
        Err(DatabaseError::InvalidSchema)
    } else {
        Ok(User { id, name: "Alice".to_string() })
    }
}
```

### Result extensions
```rust
trait ResultExt<T, E> {
    fn log_error(self) -> Self;
    fn with_context<F>(self, f: F) -> Result<T, String> 
    where 
        F: FnOnce() -> String,
        E: std::fmt::Display;
}

impl<T, E> ResultExt<T, E> for Result<T, E>
where
    E: std::fmt::Display,
{
    fn log_error(self) -> Self {
        if let Err(ref e) = self {
            eprintln!("Error occurred: {}", e);
        }
        self
    }
    
    fn with_context<F>(self, f: F) -> Result<T, String>
    where
        F: FnOnce() -> String,
    {
        self.map_err(|e| format!("{}: {}", f(), e))
    }
}

// Использование
fn process_file(path: &str) -> Result<String, String> {
    std::fs::read_to_string(path)
        .log_error()
        .with_context(|| format!("Failed to read file '{}'", path))
}
```

## 🧪 Тестирование ошибок

### Assert для ошибок
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_division_by_zero() {
        let result = divide(10.0, 0.0);
        assert!(result.is_err());
        
        if let Err(e) = result {
            assert_eq!(e, MathError::DivisionByZero);
        }
    }
    
    #[test]
    fn test_successful_parsing() {
        let result = parse_number("42");
        assert_eq!(result.unwrap(), 42);
    }
    
    #[test] 
    #[should_panic(expected = "Invalid input")]
    fn test_panic_on_invalid_input() {
        let result = parse_number("not_a_number").unwrap();
    }
}
```

### Макросы для тестирования
```rust
macro_rules! assert_error {
    ($result:expr, $error_pattern:pat) => {
        match $result {
            Err($error_pattern) => {},
            other => panic!("Expected error matching {}, got {:?}", 
                           stringify!($error_pattern), other),
        }
    };
}

// Использование
#[test]
fn test_custom_error() {
    let result = some_operation();
    assert_error!(result, MyError::InvalidInput(_));
}
```

#snippet #error-handling #result #option #resilience #intermediate