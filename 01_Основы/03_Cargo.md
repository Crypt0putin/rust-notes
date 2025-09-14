# 📦 Cargo - система сборки и менеджер пакетов

## 🎯 Что такое Cargo?

**Cargo** — это инструмент Rust для:
- 📦 Управления зависимостями
- 🔨 Сборки проектов
- 📚 Публикации библиотек
- 🧪 Запуска тестов
- 📖 Генерации документации

## 🚀 Основные команды

### Создание проекта
```bash
# Создать новый бинарный проект
cargo new мой_проект
cd мой_проект

# Создать библиотеку
cargo new --lib моя_библиотека

# Создать в текущей директории
cargo init
```

### Сборка и запуск
```bash
# Сборка проекта
cargo build

# Сборка с оптимизациями (release)
cargo build --release

# Запуск проекта
cargo run

# Запуск с аргументами
cargo run -- аргумент1 аргумент2

# Быстрая проверка без компиляции
cargo check
```

### Тестирование
```bash
# Запустить все тесты
cargo test

# Запустить конкретный тест
cargo test имя_теста

# Показать вывод даже для успешных тестов
cargo test -- --show-output

# Запустить тесты последовательно
cargo test -- --test-threads=1
```

## 📁 Структура проекта

```
мой_проект/
├── Cargo.toml          # Манифест проекта
├── Cargo.lock          # Фиксированные версии зависимостей
├── src/
│   ├── main.rs         # Точка входа для бинарного проекта
│   ├── lib.rs          # Корень библиотеки (если есть)
│   └── bin/            # Дополнительные бинарные файлы
├── tests/              # Интеграционные тесты
├── examples/           # Примеры использования
├── benches/            # Бенчмарки
└── target/             # Директория сборки (игнорируется git)
```

## 📋 Файл Cargo.toml

### Базовая структура
```toml
[package]
name = "мой_проект"
version = "0.1.0"
edition = "2021"
authors = ["Ваше Имя <email@example.com>"]
description = "Описание проекта"
license = "MIT"

[dependencies]
# Зависимости проекта

[dev-dependencies]
# Зависимости для разработки (тесты, бенчмарки)

[build-dependencies]
# Зависимости для build.rs скрипта
```

### Добавление зависимостей
```toml
[dependencies]
# Из crates.io
serde = "1.0"
tokio = { version = "1.0", features = ["full"] }

# Из git репозитория
my_lib = { git = "https://github.com/user/repo.git" }

# Локальная зависимость
local_lib = { path = "../local_lib" }

# С определённой версией
rand = "=0.8.5"     # Точная версия
rand = "^0.8.5"     # Совместимая версия (по умолчанию)
rand = "~0.8.5"     # Минимальная версия
rand = "*"          # Любая версия (не рекомендуется)
```

## 🔧 Полезные команды

### Управление зависимостями
```bash
# Обновить зависимости
cargo update

# Обновить конкретную зависимость
cargo update -p имя_пакета

# Показать дерево зависимостей
cargo tree

# Проверить устаревшие зависимости
cargo outdated

# Аудит безопасности
cargo audit
```

### Документация
```bash
# Генерировать документацию
cargo doc

# Генерировать и открыть в браузере
cargo doc --open

# Включая приватные элементы
cargo doc --document-private-items
```

### Форматирование и линтинг
```bash
# Форматировать код
cargo fmt

# Проверить форматирование
cargo fmt --check

# Запустить clippy (линтер)
cargo clippy

# Clippy с предупреждениями
cargo clippy -- -W clippy::all
```

## 🎯 Профили сборки

### Настройка в Cargo.toml
```toml
# Профиль для разработки
[profile.dev]
opt-level = 0      # Без оптимизаций
debug = true       # Включить отладочную информацию
debug-assertions = true
overflow-checks = true

# Профиль для релиза
[profile.release]
opt-level = 3      # Максимальные оптимизации
debug = false
lto = true         # Link Time Optimization
codegen-units = 1  # Меньше параллелизма, лучше оптимизация

# Кастомный профиль
[profile.bench]
inherits = "release"
debug = true
```

## 📦 Workspaces (Рабочие области)

### Структура workspace
```toml
# Корневой Cargo.toml
[workspace]
members = [
    "проект1",
    "проект2",
    "общая_библиотека",
]

[workspace.dependencies]
serde = "1.0"
tokio = "1.0"
```

### Использование в проектах
```toml
# проект1/Cargo.toml
[dependencies]
serde = { workspace = true }
общая_библиотека = { path = "../общая_библиотека" }
```

## 🚀 Публикация на crates.io

### Подготовка к публикации
```toml
[package]
name = "уникальное_имя"
version = "0.1.0"
authors = ["Ваше Имя <email@example.com>"]
edition = "2021"
description = "Краткое описание пакета"
readme = "README.md"
homepage = "https://example.com"
repository = "https://github.com/user/repo"
license = "MIT OR Apache-2.0"
keywords = ["ключевое", "слово"]
categories = ["category"]
```

### Команды публикации
```bash
# Логин на crates.io
cargo login [токен]

# Проверка перед публикацией
cargo publish --dry-run

# Публикация
cargo publish
```

## 💻 Примеры использования

### Пример 1: Создание и настройка проекта
```bash
# Создаём проект
cargo new игра_угадайка
cd игра_угадайка

# Добавляем зависимость
echo 'rand = "0.8"' >> Cargo.toml

# Или через cargo-edit (нужно установить)
cargo add rand
```

### Пример 2: Скрипт сборки (build.rs)
```rust
// build.rs
fn main() {
    // Генерация кода во время сборки
    println!("cargo:rerun-if-changed=build.rs");
    
    // Установка переменных окружения
    println!("cargo:rustc-env=BUILD_TIME={}", 
             chrono::Utc::now().to_rfc3339());
}
```

### Пример 3: Кастомные команды
```toml
# .cargo/config.toml
[alias]
b = "build"
r = "run"
t = "test"
c = "check"
d = "doc --open"
```

## 🎯 Лучшие практики

1. **Используйте Cargo.lock**
   - Коммитьте для бинарных проектов
   - Игнорируйте для библиотек

2. **Версионирование**
   - Следуйте Semantic Versioning
   - Обновляйте зависимости регулярно

3. **Оптимизация сборки**
   - Используйте `cargo check` для быстрой проверки
   - Включайте LTO для финальных релизов

4. **Организация кода**
   - Разделяйте на модули
   - Используйте workspaces для больших проектов

## 📚 Полезные инструменты

```bash
# Установка дополнительных инструментов
cargo install cargo-edit    # add, rm, upgrade команды
cargo install cargo-watch   # автоматическая пересборка
cargo install cargo-expand  # раскрытие макросов
cargo install cargo-outdated # проверка устаревших зависимостей
cargo install cargo-audit   # аудит безопасности
```

## 🔗 Связанные темы

- [[01_Основы/02_Hello_World|Первая программа]]
- [[01_Основы/01_Установка|Установка Rust]]
- [[Проекты/00_Список_проектов|Проекты]]

---
#rust #cargo #сборка #зависимости #основы
