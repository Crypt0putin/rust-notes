# 🚨 Ошибки компилятора Rust и их решения

Rust компилятор очень строг, но его сообщения об ошибках помогают писать корректный код. Здесь собраны частые ошибки и способы их исправления.

## 📋 Категории ошибок

### 🔐 Ошибки системы владения
- [[E0382-Borrow-Of-Moved-Value]] - Использование перемещённого значения
- [[E0384-Cannot-Assign-Twice]] - Повторное присваивание неизменяемой переменной  
- [[E0502-Cannot-Borrow-Mutably]] - Конфликт заимствований
- [[E0597-Borrowed-Value-Does-Not-Live-Long-Enough]] - Недостаточное время жизни

### 🕐 Ошибки времён жизни
- [[E0623-Lifetime-Mismatch]] - Несоответствие времён жизни
- [[E0621-Explicit-Lifetime-Required]] - Требуются явные времена жизни
- [[E0499-Cannot-Borrow-Mutably-Multiple-Times]] - Множественные изменяемые заимствования

### 🔢 Ошибки типов
- [[E0308-Type-Mismatch]] - Несоответствие типов
- [[E0277-Trait-Not-Implemented]] - Трейт не реализован
- [[E0599-Method-Not-Found]] - Метод не найден

## 🎯 Топ-10 частых ошибок

### 1. E0382: Использование перемещённого значения

**Ошибка:**
```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{}", s1); // ❌ Error: borrow of moved value: `s1`
```

**Решение:**
```rust
// Вариант 1: Клонирование
let s1 = String::from("hello");
let s2 = s1.clone();
println!("{}", s1); // ✅ OK

// Вариант 2: Заимствование
let s1 = String::from("hello");
let s2 = &s1;
println!("{}", s1); // ✅ OK

// Вариант 3: Использовать после move
let s1 = String::from("hello");
let s2 = s1;
println!("{}", s2); // ✅ OK
```

### 2. E0502: Нельзя заимствовать как изменяемое

**Ошибка:**
```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &mut s; // ❌ Error: cannot borrow as mutable
println!("{} {}", r1, r2);
```

**Решение:**
```rust
let mut s = String::from("hello");

// Вариант 1: Разделить области видимости
{
    let r1 = &s;
    println!("{}", r1);
} // r1 выходит из области видимости

let r2 = &mut s; // ✅ OK
println!("{}", r2);

// Вариант 2: Использовать неизменяемые ссылки полностью, затем изменяемую
let r1 = &s;
let r3 = &s;
println!("{} and {}", r1, r3); // Последнее использование r1 и r3

let r2 = &mut s; // ✅ OK, NLL (Non-Lexical Lifetimes)
println!("{}", r2);
```

### 3. E0384: Нельзя присвоить дважды

**Ошибка:**
```rust
let x = 5;
x = 6; // ❌ Error: cannot assign twice to immutable variable
```

**Решение:**
```rust
// Вариант 1: Сделать переменную изменяемой
let mut x = 5;
x = 6; // ✅ OK

// Вариант 2: Затенение (shadowing)
let x = 5;
let x = 6; // ✅ OK, создаём новую переменную
```

### 4. E0308: Несоответствие типов

**Ошибка:**
```rust
let x: i32 = "hello"; // ❌ Error: expected `i32`, found `&str`
```

**Решение:**
```rust
// Вариант 1: Правильный тип
let x: &str = "hello"; // ✅ OK

// Вариант 2: Преобразование типа
let x: i32 = "42".parse().unwrap(); // ✅ OK

// Вариант 3: Автоопределение типа
let x = "hello"; // ✅ OK, тип &str выводится автоматически
```

### 5. E0597: Заимствованное значение живёт недостаточно долго

**Ошибка:**
```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x; // ❌ Error: `x` does not live long enough
    }
    println!("{}", r);
}
```

**Решение:**
```rust
// Вариант 1: Переместить переменную в правильную область видимости  
fn main() {
    let x = 5;
    let r = &x; // ✅ OK
    println!("{}", r);
}

// Вариант 2: Владение вместо заимствования
fn main() {
    let r;
    {
        let x = 5;
        r = x; // ✅ OK, x копируется
    }
    println!("{}", r);
}
```

### 6. E0277: Трейт не реализован

**Ошибка:**
```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 1, y: 2 };
    println!("{}", p1 == p2); // ❌ Error: can't compare Point
}
```

**Решение:**
```rust
// Добавить нужные трейты
#[derive(Debug, PartialEq)]
struct Point { x: i32, y: i32 }

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 1, y: 2 };
    println!("{}", p1 == p2); // ✅ OK
}
```

## 🔧 Инструменты для диагностики

### Clippy - линтер для Rust
```bash
# Установка
rustup component add clippy

# Использование
cargo clippy

# С дополнительными проверками
cargo clippy -- -W clippy::all
```

### Расширенные ошибки компилятора
```bash
# Подробное объяснение ошибки
rustc --explain E0382

# Дополнительная информация при компиляции
RUST_BACKTRACE=1 cargo run
```

### IDE поддержка
- **rust-analyzer** показывает ошибки в реальном времени
- **Подсказки типов** помогают понять, что происходит
- **Быстрые исправления** для частых проблем

## 🎯 Стратегии отладки

### 1. Читайте сообщения об ошибках внимательно
```rust
error[E0382]: borrow of moved value: `s1`
  --> src/main.rs:4:20
   |
2  |     let s1 = String::from("hello");
   |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3  |     let s2 = s1;
   |              -- value moved here
4  |     println!("{}", s1);
   |                    ^^ value borrowed here after move
```

**Что искать:**
- Номер ошибки (E0382)
- Точное место ошибки
- Объяснение причины
- Подсказки по исправлению

### 2. Используйте временные переменные для отладки
```rust
// Вместо сложной цепочки
let result = data.iter().map(|x| x * 2).collect::<Vec<_>>();

// Разбейте на шаги
let iter = data.iter();
let mapped = iter.map(|x| x * 2);
let result: Vec<_> = mapped.collect();
```

### 3. Добавляйте аннотации типов
```rust
// Помогает компилятору и вам понять типы
let numbers: Vec<i32> = vec![1, 2, 3];
let sum: i32 = numbers.iter().sum();
```

## 📚 Ресурсы для изучения ошибок

### Официальная документация
- **[Compiler Error Index](https://doc.rust-lang.org/error-index.html)** - полный список ошибок
- **[Rust RFC](https://github.com/rust-lang/rfcs)** - объяснения дизайн-решений
- **[Rust Reference](https://doc.rust-lang.org/reference/)** - формальные правила

### Сообщество
- **[Stack Overflow](https://stackoverflow.com/questions/tagged/rust)** - вопросы и ответы
- **[Reddit r/rust](https://reddit.com/r/rust)** - обсуждения
- **[Rust Users Forum](https://users.rust-lang.org/)** - помощь от сообщества

## 🎓 Методика изучения ошибок

### 1. Активный подход
- **Не избегайте ошибок** - они учат правильному мышлению
- **Экспериментируйте** с кодом, который может не скомпилироваться  
- **Понимайте причины** вместо простого копирования решений

### 2. Ведите журнал ошибок
```markdown
## Ошибка E0382 - 15.01.2024
**Проблема**: Пытался использовать String после move
**Решение**: Использовал clone()  
**Урок**: String не Copy, нужно явно клонировать
```

### 3. Практикуйтесь на примерах
- Создавайте код, который специально вызывает ошибки
- Исправляйте чужой код с ошибками
- Объясняйте ошибки другим людям

## 🔗 Связанные материалы
- [[02-Ownership/README]] - Основы системы владения
- [[Flashcards/Errors/README]] - Карточки по ошибкам
- [[Learning-Journal]] - Записывайте сложные случаи

#compiler-errors #debugging #troubleshooting #learning-resource