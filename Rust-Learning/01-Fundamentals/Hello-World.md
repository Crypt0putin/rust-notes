# 👋 Hello World - Первая программа на Rust

Традиционная первая программа для знакомства с языком.

## 🎯 Создание проекта

### С помощью Cargo
```bash
cargo new hello_world
cd hello_world
```

### Структура проекта
```
hello_world/
├── Cargo.toml      # Конфигурация проекта
└── src/
    └── main.rs     # Главный файл программы
```

## 💻 Код программы

### Файл `src/main.rs`
```rust
fn main() {
    println!("Hello, world!");
}
```

### Разбор кода
- `fn main()` - главная функция, точка входа в программу
- `println!` - макрос для вывода текста с переводом строки
- `!` - указывает что это макрос, а не функция
- `;` - завершает выражение

## 🚀 Запуск программы

### Компиляция и запуск
```bash
cargo run
```

### Только компиляция
```bash
cargo build
```

### Запуск скомпилированного файла
```bash
# На Windows:
./target/debug/hello_world.exe

# На Linux/macOS:
./target/debug/hello_world
```

## 🔧 Cargo.toml

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
```

**Объяснение полей:**
- `name` - имя проекта
- `version` - версия (семантическое версионирование)
- `edition` - версия языка Rust (2015, 2018, 2021)
- `dependencies` - внешние библиотеки

## 🎯 Варианты Hello World

### С переменными
```rust
fn main() {
    let name = "Rust";
    println!("Hello, {}!", name);
}
```

### С пользовательским вводом
```rust
use std::io;

fn main() {
    println!("What's your name?");
    
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read line");
    
    println!("Hello, {}!", input.trim());
}
```

### С аргументами командной строки
```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    if args.len() > 1 {
        println!("Hello, {}!", args[1]);
    } else {
        println!("Hello, world!");
    }
}
```

## 🔗 Связанные темы
- [[Cargo-Basics]] - Подробнее о Cargo
- [[Project-Structure]] - Структура проектов Rust
- [[Variables-Mutability]] - Переменные в Rust

#hello-world #first-program #basics #beginner