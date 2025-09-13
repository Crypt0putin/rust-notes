# 📦 Основы Cargo

Cargo - это встроенный инструмент Rust для управления проектами, зависимостями и сборкой.

## 🎯 Что такое Cargo

Cargo выполняет множество задач:
- Создание новых проектов
- Сборка проекта
- Управление зависимостями  
- Запуск тестов
- Генерация документации
- Публикация пакетов

## 🚀 Основные команды

### Создание проекта
```bash
# Новый бинарный проект
cargo new my_project

# Новая библиотека
cargo new my_lib --lib

# В существующей директории
cargo init
```

### Сборка и запуск
```bash
# Компиляция в debug режиме
cargo build

# Компиляция в release режиме (оптимизированная)
cargo build --release

# Компиляция и запуск
cargo run

# Быстрая проверка компиляции без создания исполняемого файла
cargo check
```

### Тестирование
```bash
# Запуск всех тестов
cargo test

# Запуск конкретного теста
cargo test test_name

# Запуск тестов с выводом println!
cargo test -- --nocapture
```

### Документация
```bash
# Генерация документации
cargo doc

# Генерация и открытие в браузере
cargo doc --open
```

## 📄 Файл Cargo.toml

### Базовая структура
```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]
description = "A sample Rust project"
license = "MIT"

[dependencies]
serde = "1.0"
tokio = { version = "1.0", features = ["full"] }

[dev-dependencies]
criterion = "0.4"

[build-dependencies]
cc = "1.0"
```

### Секции файла

#### [package]
```toml
[package]
name = "my_project"          # Имя пакета
version = "0.1.0"           # Версия (SemVer)
edition = "2021"            # Версия Rust edition
authors = ["Name <email>"]   # Авторы
description = "Description"  # Описание
license = "MIT"             # Лицензия
repository = "https://..."  # Репозиторий
keywords = ["cli", "tool"]   # Ключевые слова
categories = ["command-line-utilities"]
```

#### [dependencies] 
```toml
[dependencies]
# Простая зависимость
serde = "1.0"

# С дополнительными features
tokio = { version = "1.0", features = ["full"] }

# Из git репозитория
my_lib = { git = "https://github.com/user/repo" }

# Локальная зависимость
local_lib = { path = "../local_lib" }

# Опциональная зависимость
optional_dep = { version = "1.0", optional = true }
```

## 🎯 Workspace (рабочие области)

### Создание workspace
```toml
# Cargo.toml в корне workspace
[workspace]
members = [
    "app",
    "lib1", 
    "lib2",
]

[workspace.dependencies]
serde = "1.0"
```

### Структура workspace
```
my_workspace/
├── Cargo.toml
├── app/
│   ├── Cargo.toml
│   └── src/
│       └── main.rs
├── lib1/
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
└── lib2/
    ├── Cargo.toml
    └── src/
        └── lib.rs
```

## 🔧 Конфигурация Cargo

### Файл .cargo/config.toml
```toml
[build]
target-dir = "target"

[cargo-new]
name = "Your Name"
email = "your.email@example.com"

[registries.my-registry]
index = "https://my-intranet:8080/git/index"

[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```

## 📊 Профили сборки

### Настройка профилей
```toml
[profile.dev]
opt-level = 0      # Уровень оптимизации (0-3)
debug = true       # Отладочная информация
panic = 'unwind'   # Обработка паники

[profile.release]
opt-level = 3      # Максимальная оптимизация
debug = false      # Без отладочной информации
lto = true         # Link Time Optimization
codegen-units = 1  # Параллельная кодогенерация

[profile.test]
opt-level = 0
debug = 2

[profile.bench]
opt-level = 3
debug = false
```

## 🎪 Features (функции)

### Определение features
```toml
[features]
default = ["std"]
std = []
serde_support = ["serde"]
experimental = []

[dependencies]
serde = { version = "1.0", optional = true }
```

### Использование features
```bash
# Сборка без default features
cargo build --no-default-features

# Сборка с конкретными features
cargo build --features "serde_support,experimental"

# Сборка со всеми features
cargo build --all-features
```

## 🔍 Полезные команды

### Информация о проекте
```bash
# Показать дерево зависимостей
cargo tree

# Информация о пакете
cargo metadata

# Поиск в реестре
cargo search keyword

# Обновление зависимостей
cargo update

# Очистка артефактов сборки
cargo clean
```

### Линтинг и форматирование
```bash
# Clippy (линтер)
cargo clippy

# Форматирование кода
cargo fmt

# Проверка форматирования
cargo fmt --check
```

## 🚀 Публикация пакетов

### Подготовка к публикации
```bash
# Регистрация на crates.io
cargo login

# Упаковка пакета
cargo package

# Публикация
cargo publish

# Публикация dry-run
cargo publish --dry-run
```

### Версионирование
```bash
# Обновление версии в Cargo.toml
# 0.1.0 -> 0.1.1 (patch)
# 0.1.0 -> 0.2.0 (minor)  
# 0.1.0 -> 1.0.0 (major)
```

## 🎯 Лучшие практики

### 1. Используйте точные версии для важных зависимостей
```toml
[dependencies]
# Хорошо для библиотек
serde = "1.0.130"

# Хорошо для приложений
serde = "=1.0.130"
```

### 2. Группируйте зависимости логически
```toml
# Основные зависимости
[dependencies]
serde = "1.0"
tokio = "1.0"

# Зависимости для тестов
[dev-dependencies]
criterion = "0.4"
tempfile = "3.0"
```

### 3. Используйте workspace для больших проектов
- Общие зависимости в workspace
- Локальные зависимости через path
- Единая система сборки

## 🔗 Связанные темы
- [[Project-Structure]] - Структура Rust проектов
- [[External-Crates]] - Работа с внешними библиотеками
- [[Testing-Basics]] - Тестирование в Rust

#cargo #build-system #package-manager #dependencies #beginner