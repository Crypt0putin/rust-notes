# 📁 Структура проектов Rust

Понимание организации файлов и модулей в Rust проектах.

## 🏗️ Базовая структура

### Бинарный проект
```
my_project/
├── Cargo.toml      # Манифест проекта
├── src/
│   └── main.rs     # Точка входа
├── tests/          # Интеграционные тесты (опционально)
├── examples/       # Примеры использования (опционально)
├── benches/        # Бенчмарки (опционально)
└── README.md       # Документация
```

### Библиотечный проект
```
my_lib/
├── Cargo.toml
├── src/
│   ├── lib.rs      # Корень библиотеки
│   ├── main.rs     # Опциональный бинарный файл
│   └── bin/        # Дополнительные бинарные файлы
│       └── helper.rs
├── tests/
├── examples/
└── benches/
```

## 📂 Директория src/

### Главные файлы
- **`main.rs`** - точка входа для бинарных приложений
- **`lib.rs`** - корневой модуль библиотеки

### Организация модулей
```
src/
├── main.rs         # или lib.rs
├── utils.rs        # модуль utils
├── config/         # модуль config
│   ├── mod.rs      # корень модуля config
│   ├── parser.rs   # подмодуль
│   └── validator.rs
└── database/
    ├── mod.rs
    ├── connection.rs
    └── models.rs
```

### Объявление модулей

#### В `main.rs` или `lib.rs`
```rust
// Подключение модулей
mod utils;           // utils.rs
mod config;          // config/mod.rs
mod database;        // database/mod.rs

// Публичные экспорты
pub use config::Config;
pub use database::models::User;
```

#### В `config/mod.rs`
```rust
pub mod parser;      // config/parser.rs
pub mod validator;   // config/validator.rs

// Публичные импорты
pub use parser::ConfigParser;
pub use validator::validate_config;

// Внутренние структуры
pub struct Config {
    // поля
}
```

## 🎯 Специальные директории

### tests/ - Интеграционные тесты
```
tests/
├── common/
│   └── mod.rs      # Общий код для тестов
├── integration_test.rs
└── api_test.rs
```

```rust
// tests/integration_test.rs
use my_lib::Config;

#[test]
fn test_config_loading() {
    let config = Config::new();
    assert!(config.is_valid());
}
```

### examples/ - Примеры использования
```
examples/
├── basic_usage.rs
├── advanced_config.rs
└── custom_setup.rs
```

```rust
// examples/basic_usage.rs
use my_lib::{Config, Database};

fn main() {
    let config = Config::load("config.toml").unwrap();
    let db = Database::new(&config);
    println!("Connected to database");
}
```

### benches/ - Бенчмарки
```
benches/
├── performance_test.rs
└── comparison_bench.rs
```

```rust
// benches/performance_test.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use my_lib::heavy_computation;

fn benchmark_computation(c: &mut Criterion) {
    c.bench_function("heavy computation", |b| {
        b.iter(|| heavy_computation(black_box(1000)))
    });
}

criterion_group!(benches, benchmark_computation);
criterion_main!(benches);
```

## 🔧 Файл Cargo.toml

### Детальная конфигурация
```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

# Множественные бинарные файлы
[[bin]]
name = "main_app"
path = "src/main.rs"

[[bin]]
name = "helper_tool"  
path = "src/bin/helper.rs"

# Библиотека
[lib]
name = "my_project_lib"
path = "src/lib.rs"

# Примеры
[[example]]
name = "basic_usage"
path = "examples/basic_usage.rs"

# Бенчмарки
[[bench]]
name = "performance"
path = "benches/performance_test.rs"
harness = false  # Для criterion

[dependencies]
serde = "1.0"
tokio = "1.0"

[dev-dependencies]
criterion = "0.4"
tempfile = "3.0"
```

## 🎭 Система модулей

### Inline модули
```rust
// src/main.rs
mod network {
    pub fn connect() {
        println!("Connecting...");
    }
    
    pub mod tcp {
        pub fn send_data(data: &str) {
            println!("Sending: {}", data);
        }
    }
}

fn main() {
    network::connect();
    network::tcp::send_data("Hello");
}
```

### Файловые модули
```rust
// src/main.rs
mod network;  // network.rs или network/mod.rs

fn main() {
    network::connect();
}
```

```rust
// src/network.rs
pub fn connect() {
    println!("Connecting...");
}

pub mod tcp {
    pub fn send_data(data: &str) {
        println!("Sending: {}", data);
    }
}
```

### Модули в директориях
```rust
// src/network/mod.rs
pub mod tcp;     // network/tcp.rs
pub mod udp;     // network/udp.rs

pub fn connect() {
    println!("Connecting...");
}
```

## 🔒 Видимость (Visibility)

### Модификаторы доступа
```rust
mod my_mod {
    fn private_function() {}           // Приватная
    pub fn public_function() {}        // Публичная
    pub(crate) fn crate_public() {}    // Видна в текущем crate
    pub(super) fn parent_public() {}   // Видна в родительском модуле
    pub(self) fn self_public() {}      // То же что private
    pub(in crate::my_mod) fn restricted() {} // Видна в указанном пути
}
```

### Структуры и поля
```rust
pub struct Config {
    pub name: String,        // Публичное поле
    version: u32,            // Приватное поле
    pub(crate) debug: bool,  // Видно в crate
}

impl Config {
    pub fn new(name: String) -> Self {
        Config {
            name,
            version: 1,
            debug: false,
        }
    }
    
    pub fn version(&self) -> u32 {
        self.version  // Доступ через метод
    }
}
```

## 🔄 Use и импорты

### Основы use
```rust
// Импорт функции
use std::collections::HashMap;

// Импорт нескольких элементов
use std::io::{self, Read, Write};

// Импорт с переименованием
use std::fmt::Result as FmtResult;

// Импорт всего
use std::prelude::*;

// Относительные импорты
use super::parent_module;
use crate::root_module;
use self::current_module;
```

### Паттерны импорта
```rust
// Группировка импортов
use std::{
    collections::{HashMap, HashSet},
    io::{Read, Write},
    fmt::Display,
};

// Импорт из текущего crate
use crate::{
    config::Config,
    database::{Connection, models::User},
};

// Условные импорты
#[cfg(feature = "serde")]
use serde::{Deserialize, Serialize};
```

## 🏢 Workspace проекты

### Структура workspace
```
my_workspace/
├── Cargo.toml          # Workspace manifest
├── app/                # Главное приложение
│   ├── Cargo.toml
│   └── src/
│       └── main.rs
├── core/               # Основная библиотека
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
├── utils/              # Утилиты
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
└── tests/              # Общие тесты
    └── integration_tests.rs
```

### Workspace Cargo.toml
```toml
[workspace]
members = [
    "app",
    "core", 
    "utils",
]

# Общие зависимости
[workspace.dependencies]
serde = "1.0"
tokio = "1.0"

# Общие метаданные
[workspace.package]
version = "0.1.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]
```

### Зависимости между пакетами
```toml
# app/Cargo.toml
[dependencies]
core = { path = "../core" }
utils = { path = "../utils" }
serde = { workspace = true }
```

## 🎯 Лучшие практики

### 1. Структура модулей
```rust
// Хорошо: логическая группировка
src/
├── lib.rs
├── config/
├── database/
├── api/
└── utils/

// Плохо: плоская структура
src/
├── lib.rs
├── config.rs
├── database.rs
├── api.rs
├── utils.rs
├── helper1.rs
├── helper2.rs
```

### 2. Публичные API
```rust
// src/lib.rs - чистый публичный API
pub use config::Config;
pub use database::Database;
pub use api::{start_server, ApiError};

// Скрываем детали реализации
mod internal;
```

### 3. Тестирование
```rust
// Юнит-тесты в том же файле
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_function() {
        // тест
    }
}

// Интеграционные тесты в tests/
// tests/integration_test.rs
use my_lib::Config;
```

## 🔗 Связанные темы
- [[Cargo-Basics]] - Управление проектами
- [[Module-System]] - Система модулей в деталях
- [[Visibility-Rules]] - Правила видимости

#project-structure #modules #organization #cargo #workspace #intermediate