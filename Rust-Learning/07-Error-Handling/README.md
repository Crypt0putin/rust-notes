# 🚨 Обработка ошибок в Rust

Rust использует алгебраические типы данных для безопасной и явной обработки ошибок без исключений.

## 📋 Содержание

### Основы обработки ошибок
- [[Result-Type]] - Тип Result<T, E>
- [[Option-Type]] - Тип Option<T>  
- [[Error-Propagation]] - Распространение ошибок с `?`
- [[Panic-vs-Result]] - Когда использовать panic vs Result

### Продвинутые техники
- [[Custom-Error-Types]] - Пользовательские типы ошибок
- [[Error-Trait]] - Стандартный трейт Error
- [[Error-Chaining]] - Цепочки ошибок
- [[Anyhow-Thiserror]] - Популярные библиотеки для ошибок

## 🎯 Философия обработки ошибок в Rust

### Нет исключений!
```rust
// В других языках:
// try {
//     let file = File::open("config.txt");
//     // ...
// } catch (IOException e) {
//     // handle error
// }

// В Rust - явная обработка:
match File::open("config.txt") {
    Ok(file) => {
        // работаем с файлом
    },
    Err(error) => {
        // обрабатываем ошибку
    }
}
```

### Два типа ошибок:
1. **Восстанавливаемые** - `Result<T, E>`
2. **Невосстанавливаемые** - `panic!`

## 📦 Result<T, E>

### Определение Result
```rust
enum Result<T, E> {
    Ok(T),    // Успешный результат
    Err(E),   // Ошибка
}
```

### Базовое использование
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
```

### Методы Result
```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("config.txt")?;
    
    // unwrap() - паника при ошибке
    // let content = std::fs::read_to_string("config.txt").unwrap();
    
    // expect() - паника с пользовательским сообщением
    // let content = std::fs::read_to_string("config.txt")
    //     .expect("Failed to read config file");
    
    // unwrap_or() - значение по умолчанию при ошибке
    let content = std::fs::read_to_string("config.txt")
        .unwrap_or_else(|_| "default config".to_string());
    
    // unwrap_or_else() - вызов замыкания при ошибке
    let content = std::fs::read_to_string("config.txt")
        .unwrap_or_else(|error| {
            eprintln!("Warning: using default config due to: {}", error);
            "default config".to_string()
        });
    
    println!("Config: {}", content);
    Ok(())
}
```

## 🔗 Оператор `?` для распространения ошибок

### Как работает `?`
```rust
// Длинная версия:
fn read_username_from_file() -> Result<String, std::io::Error> {
    let f = File::open("hello.txt");
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}

// С оператором ?:
fn read_username_from_file() -> Result<String, std::io::Error> {
    let mut f = File::open("hello.txt")?;  // Возвращает ошибку, если не удалось
    let mut s = String::new();
    f.read_to_string(&mut s)?;             // Возвращает ошибку, если не удалось
    Ok(s)
}

// Ещё короче:
fn read_username_from_file() -> Result<String, std::io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}

// Самый короткий способ:
fn read_username_from_file() -> Result<String, std::io::Error> {
    std::fs::read_to_string("hello.txt")
}
```

### `?` в main функции
```rust
use std::error::Error;

// main может возвращать Result!
fn main() -> Result<(), Box<dyn Error>> {
    let content = std::fs::read_to_string("config.txt")?;
    println!("File content: {}", content);
    Ok(())
}
```

## 🎭 Option<T>

### Когда использовать Option
```rust
fn find_user(id: u32) -> Option<User> {
    // Поиск пользователя может не найти результат
    if id == 1 {
        Some(User { name: "Alice".to_string() })
    } else {
        None
    }
}

// Работа с Option
fn main() {
    match find_user(1) {
        Some(user) => println!("Found user: {}", user.name),
        None => println!("User not found"),
    }
    
    // Или с if let
    if let Some(user) = find_user(1) {
        println!("User: {}", user.name);
    }
    
    // Методы Option
    let user = find_user(1)
        .unwrap_or_else(|| User { name: "Default".to_string() });
}
```

### Конвертация между Option и Result
```rust
fn find_user_result(id: u32) -> Result<User, String> {
    find_user(id).ok_or_else(|| format!("User {} not found", id))
}

fn find_user_option(id: u32) -> Option<User> {
    find_user_result(id).ok()
}
```

## 🛠️ Пользовательские типы ошибок

### Простой enum для ошибок
```rust
#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
    InvalidInput(String),
}

impl std::fmt::Display for MathError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            MathError::DivisionByZero => write!(f, "Cannot divide by zero"),
            MathError::NegativeSquareRoot => write!(f, "Cannot take square root of negative number"),
            MathError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
        }
    }
}

impl std::error::Error for MathError {}

fn divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}
```

### Структура для ошибки
```rust
#[derive(Debug)]
struct ConfigError {
    message: String,
    line: Option<usize>,
}

impl ConfigError {
    fn new(message: impl Into<String>) -> Self {
        ConfigError {
            message: message.into(),
            line: None,
        }
    }
    
    fn with_line(mut self, line: usize) -> Self {
        self.line = Some(line);
        self
    }
}

impl std::fmt::Display for ConfigError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self.line {
            Some(line) => write!(f, "Config error at line {}: {}", line, self.message),
            None => write!(f, "Config error: {}", self.message),
        }
    }
}

impl std::error::Error for ConfigError {}
```

## 📚 Популярные библиотеки

### anyhow - для приложений
```rust
// Cargo.toml: anyhow = "1.0"
use anyhow::{Result, Context, anyhow};

fn read_config() -> Result<Config> {
    let content = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;
    
    let config: Config = toml::from_str(&content)
        .context("Failed to parse config file")?;
    
    if config.port == 0 {
        return Err(anyhow!("Port cannot be zero"));
    }
    
    Ok(config)
}

fn main() -> Result<()> {
    let config = read_config()
        .context("Failed to initialize configuration")?;
    
    println!("Starting server on port {}", config.port);
    Ok(())
}
```

### thiserror - для библиотек
```rust
// Cargo.toml: thiserror = "1.0"
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] std::io::Error),
    
    #[error("the data for key `{0}` is not available")]
    NotFound(String),
    
    #[error("invalid header (expected {expected:?}, found {found:?})")]
    InvalidHeader {
        expected: String,
        found: String,
    },
    
    #[error("unknown data store error")]
    Unknown,
}

fn get_data(key: &str) -> Result<String, DataStoreError> {
    if key == "missing" {
        Err(DataStoreError::NotFound(key.to_string()))
    } else {
        Ok("data".to_string())
    }
}
```

## 🔧 Продвинутые техники

### Цепочки ошибок
```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
struct HighLevelError {
    source: Box<dyn Error>,
}

impl fmt::Display for HighLevelError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "High level error occurred")
    }
}

impl Error for HighLevelError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        Some(self.source.as_ref())
    }
}

// Функция для печати всей цепочки ошибок
fn print_error_chain(err: &dyn Error) {
    eprintln!("Error: {}", err);
    let mut source = err.source();
    while let Some(err) = source {
        eprintln!("Caused by: {}", err);
        source = err.source();
    }
}
```

### Конвертация между типами ошибок
```rust
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(std::io::Error),
    Parse(ParseIntError),
    Custom(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::Io(err) => write!(f, "I/O error: {}", err),
            AppError::Parse(err) => write!(f, "Parse error: {}", err),
            AppError::Custom(msg) => write!(f, "Application error: {}", msg),
        }
    }
}

impl Error for AppError {}

impl From<std::io::Error> for AppError {
    fn from(err: std::io::Error) -> AppError {
        AppError::Io(err)
    }
}

impl From<ParseIntError> for AppError {
    fn from(err: ParseIntError) -> AppError {
        AppError::Parse(err)
    }
}

// Теперь ? автоматически конвертирует ошибки
fn process_file() -> Result<i32, AppError> {
    let content = std::fs::read_to_string("number.txt")?; // io::Error -> AppError
    let number: i32 = content.trim().parse()?; // ParseIntError -> AppError
    Ok(number * 2)
}
```

## 💡 Лучшие практики

### 1. Предпочитайте Result panic'у
```rust
// Вместо:
fn divide(a: f64, b: f64) -> f64 {
    if b == 0.0 {
        panic!("Division by zero!");
    }
    a / b
}

// Лучше:
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}
```

### 2. Используйте осмысленные типы ошибок
```rust
// Вместо общих String:
fn parse_config(content: &str) -> Result<Config, String> { ... }

// Лучше специализированные типы:
fn parse_config(content: &str) -> Result<Config, ConfigError> { ... }
```

### 3. Документируйте возможные ошибки
```rust
/// Reads configuration from file
/// 
/// # Errors
/// 
/// Returns `ConfigError::FileNotFound` if config file doesn't exist
/// Returns `ConfigError::ParseError` if config file has invalid syntax
/// Returns `ConfigError::ValidationError` if config values are invalid
fn read_config(path: &Path) -> Result<Config, ConfigError> {
    // ...
}
```

### 4. Группируйте похожие ошибки
```rust
#[derive(Error, Debug)]
pub enum NetworkError {
    #[error("Connection failed")]
    Connection(#[from] std::io::Error),
    
    #[error("Timeout after {seconds}s")]
    Timeout { seconds: u64 },
    
    #[error("Invalid response: {0}")]
    InvalidResponse(String),
}
```

## 🎯 Практические примеры

### Парсер конфигурации
```rust
#[derive(Debug)]
struct Config {
    host: String,
    port: u16,
    database_url: String,
}

#[derive(Error, Debug)]
enum ConfigError {
    #[error("Missing required field: {field}")]
    MissingField { field: String },
    
    #[error("Invalid port: {port}")]
    InvalidPort { port: String },
    
    #[error("I/O error")]
    Io(#[from] std::io::Error),
}

fn parse_config_line(line: &str) -> Result<(String, String), ConfigError> {
    let parts: Vec<&str> = line.split('=').collect();
    if parts.len() != 2 {
        return Err(ConfigError::MissingField {
            field: "key=value format".to_string(),
        });
    }
    Ok((parts[0].trim().to_string(), parts[1].trim().to_string()))
}

fn load_config(path: &str) -> Result<Config, ConfigError> {
    let content = std::fs::read_to_string(path)?;
    
    let mut host = None;
    let mut port = None;
    let mut database_url = None;
    
    for line in content.lines() {
        if line.trim().is_empty() || line.starts_with('#') {
            continue;
        }
        
        let (key, value) = parse_config_line(line)?;
        
        match key.as_str() {
            "host" => host = Some(value),
            "port" => {
                port = Some(value.parse().map_err(|_| ConfigError::InvalidPort { 
                    port: value 
                })?);
            }
            "database_url" => database_url = Some(value),
            _ => {} // Игнорируем неизвестные ключи
        }
    }
    
    Ok(Config {
        host: host.ok_or(ConfigError::MissingField { 
            field: "host".to_string() 
        })?,
        port: port.ok_or(ConfigError::MissingField { 
            field: "port".to_string() 
        })?,
        database_url: database_url.ok_or(ConfigError::MissingField { 
            field: "database_url".to_string() 
        })?,
    })
}
```

## 🧪 Упражнения

### Начальный уровень
1. Напишите функцию деления с обработкой ошибок
2. Создайте парсер целых чисел с пользовательскими ошибками
3. Реализуйте функцию чтения файла с graceful fallback

### Средний уровень
1. Создайте калькулятор с различными типами ошибок
2. Реализуйте парсер CSV с детализированными ошибками
3. Напишите HTTP клиент с обработкой сетевых ошибок

### Продвинутый уровень
1. Создайте систему ошибок с цепочками и контекстом
2. Реализуйте retry логику с экспоненциальным backoff
3. Напишите макрос для автоматической генерации Error типов

## 🔗 Связанные темы
- [[Option-Patterns]] - Паттерны работы с Option
- [[Async-Error-Handling]] - Ошибки в асинхронном коде
- [[Testing-Errors]] - Тестирование обработки ошибок

#error-handling #result #option #safety #reliability