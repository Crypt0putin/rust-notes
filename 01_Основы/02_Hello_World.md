# 👋 Hello, World! - Первая программа на Rust

## 🎯 Традиционное начало

Каждое изучение языка программирования начинается с программы "Hello, World!". Давайте создадим её на Rust!

## 📝 Создание программы вручную

### Шаг 1: Создайте файл
Создайте новый файл с именем `main.rs`:

```rust
fn main() {
    println!("Привет, мир!");
}
```

### Шаг 2: Компиляция
```bash
# Компилируем файл
rustc main.rs

# В результате появится исполняемый файл:
# Linux/Mac: main
# Windows: main.exe
```

### Шаг 3: Запуск
```bash
# Linux/Mac
./main

# Windows
main.exe

# Вывод:
# Привет, мир!
```

## 📦 Создание проекта с Cargo (рекомендуется)

### Создание нового проекта
```bash
cargo new hello_world
cd hello_world
```

### Структура проекта
```
hello_world/
├── Cargo.toml      # Манифест проекта
└── src/
    └── main.rs     # Главный файл программы
```

### Файл Cargo.toml
```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
# Здесь будут зависимости
```

### Файл src/main.rs
```rust
fn main() {
    println!("Привет, мир!");
}
```

### Запуск через Cargo
```bash
# Компиляция и запуск
cargo run

# Вывод:
#    Compiling hello_world v0.1.0 (/path/to/hello_world)
#     Finished dev [unoptimized + debuginfo] target(s) in 0.58s
#      Running `target/debug/hello_world`
# Привет, мир!
```

## 🔍 Разбор программы

### Анатомия Hello World
```rust
fn main() {
    println!("Привет, мир!");
}
```

Давайте разберём каждую часть:

### 1. Функция main
```rust
fn main() {
    // Тело функции
}
```
- `fn` - ключевое слово для объявления функции
- `main` - имя функции (точка входа в программу)
- `()` - список параметров (пустой в данном случае)
- `{}` - фигурные скобки обозначают тело функции

### 2. Макрос println!
```rust
println!("Привет, мир!");
```
- `println!` - макрос для вывода текста с новой строкой
- `!` - восклицательный знак означает, что это макрос, а не функция
- `"Привет, мир!"` - строковый литерал
- `;` - точка с запятой завершает выражение

## 🎨 Варианты Hello World

### С переменной
```rust
fn main() {
    let приветствие = "Привет, мир!";
    println!("{}", приветствие);
}
```

### С форматированием
```rust
fn main() {
    let имя = "Rust";
    let версия = "1.70";
    println!("Привет от {} версии {}!", имя, версия);
}
```

### С несколькими строками
```rust
fn main() {
    println!("Привет,");
    println!("прекрасный");
    println!("мир!");
}
```

### С Unicode
```rust
fn main() {
    println!("Привет, мир! 🦀");
    println!("Hello, 世界! 👋");
    println!("Здравствуй, мир! 🚀");
}
```

### С цветным выводом (требует крейт)
```rust
// Добавьте в Cargo.toml:
// [dependencies]
// colored = "2.0"

use colored::*;

fn main() {
    println!("{}", "Привет, мир!".green());
    println!("{}", "Hello, World!".blue().bold());
    println!("{}", "¡Hola, Mundo!".red().underline());
}
```

## 💡 Расширенные примеры

### Hello World с аргументами командной строки
```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    if args.len() > 1 {
        println!("Привет, {}!", args[1]);
    } else {
        println!("Привет, мир!");
    }
}
```

Запуск:
```bash
cargo run -- Rustacean
# Вывод: Привет, Rustacean!
```

### Интерактивный Hello World
```rust
use std::io;

fn main() {
    println!("Как вас зовут?");
    
    let mut имя = String::new();
    
    io::stdin()
        .read_line(&mut имя)
        .expect("Не удалось прочитать строку");
    
    let имя = имя.trim();
    
    println!("Привет, {}! Добро пожаловать в мир Rust!", имя);
}
```

### Hello World с обработкой ошибок
```rust
use std::fs;
use std::io::Error;

fn main() -> Result<(), Error> {
    let сообщение = "Привет, мир!";
    
    // Записываем в файл
    fs::write("hello.txt", сообщение)?;
    
    // Читаем из файла
    let содержимое = fs::read_to_string("hello.txt")?;
    
    println!("Из файла: {}", содержимое);
    
    Ok(())
}
```

## 🧪 Тестирование Hello World

### Добавьте тест в main.rs
```rust
fn привет() -> String {
    String::from("Привет, мир!")
}

fn main() {
    println!("{}", привет());
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn тест_привет() {
        assert_eq!(привет(), "Привет, мир!");
    }
}
```

Запуск теста:
```bash
cargo test
```

## 📊 Сравнение с другими языками

### C
```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

### Python
```python
print("Hello, World!")
```

### JavaScript
```javascript
console.log("Hello, World!");
```

### Java
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### Rust
```rust
fn main() {
    println!("Hello, World!");
}
```

## 🎯 Упражнения

### Упражнение 1: Модифицируйте приветствие
Измените программу так, чтобы она:
1. Выводила ваше имя
2. Выводила текущую дату
3. Использовала разные цвета

### Упражнение 2: Множественные приветствия
Создайте программу, которая приветствует на 5 разных языках.

### Упражнение 3: ASCII арт
```rust
fn main() {
    println!(r#"
    ____            _   
   |  _ \ _   _ ___| |_ 
   | |_) | | | / __| __|
   |  _ <| |_| \__ \ |_ 
   |_| \_\\__,_|___/\__|
   
   Привет, мир!
    "#);
}
```

## ⚠️ Частые ошибки

### Ошибка 1: Забыли точку с запятой
```rust
// ❌ Ошибка
fn main() {
    println!("Привет, мир!")  // Забыли ;
}

// ✅ Правильно
fn main() {
    println!("Привет, мир!");
}
```

### Ошибка 2: print vs println
```rust
fn main() {
    print!("Привет, ");   // Без новой строки
    print!("мир!");
    println!();           // Добавляем новую строку
    
    println!("Привет, мир!"); // С новой строкой
}
```

### Ошибка 3: Неправильное имя файла
- ✅ `main.rs` - правильно для исполняемого файла
- ❌ `Main.rs` - Rust чувствителен к регистру
- ❌ `hello.rs` - Cargo ищет main.rs для бинарного проекта

## 🚀 Что дальше?

Теперь, когда вы создали свою первую программу на Rust:

1. **Поэкспериментируйте** с разными вариантами вывода
2. **Изучите [[01_Основы/03_Cargo|Cargo]]** для управления проектами
3. **Перейдите к [[01_Основы/04_Переменные|переменным]]**
4. **Попробуйте [[01_Основы/06_Функции|создать функции]]**

## 💡 Интересные факты

- Макрос `println!` проверяет форматирование во время компиляции
- Rust компилируется в нативный код без виртуальной машины
- Размер hello world в Rust ~3MB (можно уменьшить до ~200KB)
- Rust гарантирует безопасность памяти без сборщика мусора

## 📚 Дополнительное чтение

- [Официальная книга: Hello, World!](https://doc.rust-lang.org/book/ch01-02-hello-world.html)
- [Rust by Example: Hello World](https://doc.rust-lang.org/rust-by-example/hello.html)
- [Почему println! это макрос?](https://stackoverflow.com/questions/29611387/why-does-the-println-function-use-an-exclamation-mark-in-rust)

---

**Поздравляем! 🎉** Вы написали свою первую программу на Rust!

Помните: каждый эксперт когда-то написал свой первый "Hello, World!"

---
#rust #hello_world #начало #первая_программа #основы
