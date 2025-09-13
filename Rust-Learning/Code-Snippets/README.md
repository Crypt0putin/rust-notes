# 🛠️ Коллекция полезных сниппетов кода

Готовые паттерны и примеры кода для быстрого использования в проектах.

## 📚 Категории сниппетов

### 🔤 Базовые паттерны
- [[Basic-Patterns]] - Фундаментальные конструкции
- [[String-Operations]] - Работа со строками
- [[Vector-Operations]] - Операции с векторами
- [[HashMap-Patterns]] - Работа с HashMap

### 🔧 Функциональное программирование
- [[Iterator-Patterns]] - Паттерны итераторов
- [[Closure-Examples]] - Примеры замыканий
- [[Filter-Map-Collect]] - Цепочки обработки данных

### 🚨 Обработка ошибок
- [[Error-Handling-Patterns]] - Паттерны обработки ошибок
- [[Result-Patterns]] - Работа с Result
- [[Option-Patterns]] - Работа с Option

### 🏗️ Структуры и трейты
- [[Struct-Patterns]] - Полезные паттерны структур
- [[Trait-Implementations]] - Реализации трейтов
- [[Builder-Pattern]] - Паттерн Builder

### 📁 Файлы и I/O
- [[File-Operations]] - Работа с файлами
- [[CLI-Patterns]] - Паттерны для CLI приложений
- [[JSON-Operations]] - Работа с JSON

### ⚡ Производительность
- [[Performance-Patterns]] - Оптимизация производительности
- [[Memory-Efficient]] - Эффективная работа с памятью
- [[Unsafe-Patterns]] - Безопасные unsafe блоки

## 🎯 Быстрые паттерны

### 📝 Чтение файла в строку
```rust
use std::fs;
use std::io::Result;

fn read_file_to_string(path: &str) -> Result<String> {
    fs::read_to_string(path)
}

// Использование
match read_file_to_string("config.txt") {
    Ok(content) => println!("File content: {}", content),
    Err(e) => eprintln!("Error reading file: {}", e),
}
```

### 🔄 Итерация с обработкой ошибок
```rust
use std::fs;

fn process_files(paths: &[&str]) -> Result<Vec<String>, std::io::Error> {
    paths.iter()
        .map(|&path| fs::read_to_string(path))
        .collect()
}
```

### 📊 Группировка данных
```rust
use std::collections::HashMap;

fn group_by<T, K, F>(items: Vec<T>, key_fn: F) -> HashMap<K, Vec<T>>
where
    K: Eq + std::hash::Hash,
    F: Fn(&T) -> K,
{
    let mut groups = HashMap::new();
    for item in items {
        let key = key_fn(&item);
        groups.entry(key).or_insert_with(Vec::new).push(item);
    }
    groups
}
```

### 🎯 Builder паттерн
```rust
#[derive(Debug, Default)]
pub struct Config {
    pub host: String,
    pub port: u16,
    pub timeout: Option<u32>,
}

impl Config {
    pub fn builder() -> ConfigBuilder {
        ConfigBuilder::default()
    }
}

#[derive(Default)]
pub struct ConfigBuilder {
    host: Option<String>,
    port: Option<u16>,
    timeout: Option<u32>,
}

impl ConfigBuilder {
    pub fn host(mut self, host: impl Into<String>) -> Self {
        self.host = Some(host.into());
        self
    }
    
    pub fn port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }
    
    pub fn timeout(mut self, timeout: u32) -> Self {
        self.timeout = Some(timeout);
        self
    }
    
    pub fn build(self) -> Config {
        Config {
            host: self.host.unwrap_or_else(|| "localhost".to_string()),
            port: self.port.unwrap_or(8080),
            timeout: self.timeout,
        }
    }
}

// Использование:
let config = Config::builder()
    .host("example.com")
    .port(3000)
    .timeout(30)
    .build();
```

## 🎨 Макросы для генерации кода

### 📝 Макрос для создания enum с методами
```rust
macro_rules! create_enum {
    ($name:ident { $($variant:ident),* }) => {
        #[derive(Debug, Clone, PartialEq)]
        pub enum $name {
            $($variant,)*
        }
        
        impl $name {
            pub fn all() -> Vec<Self> {
                vec![$(Self::$variant,)*]
            }
            
            pub fn as_str(&self) -> &'static str {
                match self {
                    $(Self::$variant => stringify!($variant),)*
                }
            }
        }
    };
}

// Использование:
create_enum!(Status { Active, Inactive, Pending });
```

### 🔧 Макрос для цепочки методов
```rust
macro_rules! chain {
    ($initial:expr $(, $method:ident($($arg:expr),*))* ) => {{
        let mut result = $initial;
        $(
            result = result.$method($($arg),*);
        )*
        result
    }};
}

// Использование:
let result = chain!(vec![1, 2, 3, 4, 5]
    , into_iter()
    , filter(|&x| x % 2 == 0)
    , map(|x| x * 2)
    , collect::<Vec<_>>()
);
```

## 📋 Шаблоны для различных типов кода

### 🏗️ Шаблон CLI приложения
```rust
use clap::{Arg, Command};
use std::process;

fn main() {
    let matches = Command::new("myapp")
        .version("1.0")
        .about("Does awesome things")
        .arg(Arg::new("input")
            .help("The input file to use")
            .required(true)
            .index(1))
        .arg(Arg::new("verbose")
            .short('v')
            .long("verbose")
            .help("Enable verbose output"))
        .get_matches();

    let input_file = matches.get_one::<String>("input").unwrap();
    let verbose = matches.get_flag("verbose");

    if let Err(e) = run(input_file, verbose) {
        eprintln!("Application error: {}", e);
        process::exit(1);
    }
}

fn run(input_file: &str, verbose: bool) -> Result<(), Box<dyn std::error::Error>> {
    if verbose {
        println!("Processing file: {}", input_file);
    }
    
    // Ваша логика здесь
    
    Ok(())
}
```

### 🗃️ Шаблон работы с конфигурацией
```rust
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::Path;

#[derive(Debug, Serialize, Deserialize)]
pub struct AppConfig {
    pub database_url: String,
    pub port: u16,
    pub log_level: String,
}

impl Default for AppConfig {
    fn default() -> Self {
        Self {
            database_url: "sqlite::memory:".to_string(),
            port: 8080,
            log_level: "info".to_string(),
        }
    }
}

impl AppConfig {
    pub fn load_from_file<P: AsRef<Path>>(path: P) -> Result<Self, Box<dyn std::error::Error>> {
        let content = fs::read_to_string(path)?;
        let config: AppConfig = toml::from_str(&content)?;
        Ok(config)
    }
    
    pub fn load_or_default<P: AsRef<Path>>(path: P) -> Self {
        Self::load_from_file(path).unwrap_or_default()
    }
}
```

## 🧪 Тестовые утилиты

### 📝 Шаблон тестов
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_function_name() {
        // Arrange
        let input = setup_test_data();
        
        // Act  
        let result = function_under_test(input);
        
        // Assert
        assert_eq!(result.unwrap(), expected_value);
    }
    
    #[test]
    #[should_panic(expected = "Expected error message")]
    fn test_error_case() {
        let bad_input = create_bad_input();
        function_that_should_panic(bad_input);
    }
    
    fn setup_test_data() -> TestData {
        TestData {
            // ... test data setup
        }
    }
}
```

### 🔧 Полезные макросы для тестов
```rust
macro_rules! assert_contains {
    ($haystack:expr, $needle:expr) => {
        assert!(
            $haystack.contains($needle),
            "Expected '{}' to contain '{}'",
            $haystack,
            $needle
        );
    };
}

macro_rules! assert_error_type {
    ($result:expr, $error_type:path) => {
        match $result {
            Err($error_type(_)) => {},
            other => panic!("Expected {}, got {:?}", stringify!($error_type), other),
        }
    };
}
```

## 🎯 Часто используемые функции

### 🔄 Retry логика
```rust
use std::time::Duration;
use std::thread::sleep;

pub fn retry<F, T, E>(mut f: F, max_attempts: u32, delay: Duration) -> Result<T, E>
where
    F: FnMut() -> Result<T, E>,
{
    for attempt in 1..=max_attempts {
        match f() {
            Ok(result) => return Ok(result),
            Err(e) => {
                if attempt == max_attempts {
                    return Err(e);
                }
                sleep(delay);
            }
        }
    }
    unreachable!()
}
```

### 📊 Измерение времени выполнения
```rust
use std::time::Instant;

pub fn measure_time<F, T>(f: F) -> (T, Duration)
where
    F: FnOnce() -> T,
{
    let start = Instant::now();
    let result = f();
    let duration = start.elapsed();
    (result, duration)
}

// Использование:
let (result, time_taken) = measure_time(|| {
    // Ваш код здесь
    expensive_computation()
});
println!("Operation took: {:?}", time_taken);
```

## 🔗 Организация сниппетов

### 📁 Структура файлов
- Каждая категория в отдельном файле
- Примеры использования для каждого паттерна
- Комментарии с объяснением логики
- Теги для быстрого поиска

### 🏷️ Система тегов
- `#snippet` - основной тег
- `#performance` - оптимизация производительности
- `#error-handling` - обработка ошибок
- `#async` - асинхронный код
- `#cli` - командная строка
- `#testing` - тестирование

### 🔍 Быстрый поиск
Используйте поиск Obsidian для быстрого нахождения нужного паттерна:
- Поиск по тегам: `tag:#snippet #performance`
- Поиск по содержимому: `content:"Result<"`
- Комбинированный поиск: `tag:#snippet "HashMap"`

#code-snippets #patterns #templates #reusable-code