# 🦀 Rust Cheatsheet

## 📝 Базовый синтаксис

### Переменные
```rust
let x = 5;                  // неизменяемая
let mut y = 5;              // изменяемая
const MAX: u32 = 100_000;   // константа
static GLOBAL: &str = "Hi"; // статическая
```

### Типы данных
```rust
// Скаляры
let int: i32 = -42;         // знаковое целое
let uint: u32 = 42;         // беззнаковое целое
let float: f64 = 3.14;      // плавающая точка
let boolean: bool = true;   // булево
let character: char = '🦀';  // символ

// Составные
let tuple: (i32, f64, u8) = (500, 6.4, 1);
let array: [i32; 5] = [1, 2, 3, 4, 5];
let slice: &[i32] = &array[1..3];
```

### Строки
```rust
let s1: &str = "Hello";           // строковый срез
let s2: String = String::from("World"); // владеющая строка
let s3 = format!("{} {}", s1, s2); // форматирование
let s4 = s2.clone();              // клонирование
```

## 🔧 Функции

```rust
fn function_name(param1: Type1, param2: Type2) -> ReturnType {
    // тело функции
    return_value // без ; для возврата
}

// Замыкания
let closure = |x: i32| -> i32 { x + 1 };
let short_closure = |x| x + 1;
```

## 🎯 Управление потоком

### if/else
```rust
if condition {
    // код
} else if other_condition {
    // код
} else {
    // код
}

let number = if condition { 5 } else { 6 };
```

### Циклы
```rust
// Бесконечный цикл
loop {
    break;      // выход
    continue;   // следующая итерация
}

// С условием
while condition {
    // код
}

// Итератор
for element in collection {
    // код
}

for i in 0..10 { }        // 0 до 9
for i in 0..=10 { }       // 0 до 10
```

### Pattern Matching
```rust
match value {
    1 => println!("one"),
    2 | 3 => println!("two or three"),
    4..=10 => println!("four through ten"),
    _ => println!("something else"),
}

// if let
if let Some(x) = option {
    // используем x
}

// while let
while let Some(x) = option {
    // используем x
}
```

## 📦 Структуры и Енумы

### Структуры
```rust
// Обычная структура
struct User {
    username: String,
    email: String,
    active: bool,
}

// Tuple struct
struct Color(i32, i32, i32);

// Unit struct
struct AlwaysEqual;

// Методы
impl User {
    // Ассоциированная функция
    fn new(username: String, email: String) -> Self {
        Self { username, email, active: true }
    }
    
    // Метод
    fn is_active(&self) -> bool {
        self.active
    }
    
    // Изменяющий метод
    fn deactivate(&mut self) {
        self.active = false;
    }
}
```

### Енумы
```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

enum Option<T> {
    None,
    Some(T),
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 🔑 Ownership & Borrowing

### Ownership правила
```rust
let s1 = String::from("hello");
let s2 = s1;        // s1 больше не валидна (move)
let s3 = s2.clone(); // s2 остается валидной (clone)
```

### Заимствование
```rust
let s = String::from("hello");
let r1 = &s;        // неизменяемая ссылка
let r2 = &s;        // еще одна неизменяемая - ОК

let mut s = String::from("hello");
let r3 = &mut s;    // изменяемая ссылка
// let r4 = &mut s; // ОШИБКА: только одна изменяемая
```

### Lifetimes
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

## 🎨 Traits

```rust
trait Summary {
    fn summarize(&self) -> String;
    
    // Метод по умолчанию
    fn default_method(&self) {
        println!("Default implementation");
    }
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.title, self.author)
    }
}

// Trait bounds
fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// where clause
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // код
}
```

## 🧬 Generics

```rust
// Обобщенная функция
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    // код
}

// Обобщенная структура
struct Point<T> {
    x: T,
    y: T,
}

// Обобщенный енум
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Обобщенная реализация
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

## ⚠️ Error Handling

```rust
// panic! для невосстановимых ошибок
panic!("crash and burn");

// Result для восстановимых ошибок
use std::fs::File;

let f = File::open("hello.txt");
let f = match f {
    Ok(file) => file,
    Err(error) => panic!("Problem: {:?}", error),
};

// Оператор ?
fn read_username() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

// unwrap и expect
let f = File::open("hello.txt").unwrap();
let f = File::open("hello.txt").expect("Failed to open");
```

## 🔄 Итераторы

```rust
let v = vec![1, 2, 3];

// Создание итератора
let v_iter = v.iter();

// Методы итераторов
v.iter()
    .map(|x| x + 1)
    .filter(|x| x % 2 == 0)
    .fold(0, |acc, x| acc + x);

// collect
let doubled: Vec<i32> = v.iter().map(|x| x * 2).collect();

// for цикл использует итераторы
for val in v {
    println!("{}", val);
}
```

## 🧠 Smart Pointers

```rust
// Box - размещение на куче
let b = Box::new(5);

// Rc - подсчет ссылок (single-threaded)
use std::rc::Rc;
let rc = Rc::new(5);
let rc_clone = Rc::clone(&rc);

// Arc - атомарный подсчет ссылок (multi-threaded)
use std::sync::Arc;
let arc = Arc::new(5);

// RefCell - внутренняя изменяемость
use std::cell::RefCell;
let cell = RefCell::new(5);
*cell.borrow_mut() = 6;

// Mutex - безопасный доступ между потоками
use std::sync::Mutex;
let mutex = Mutex::new(5);
let mut num = mutex.lock().unwrap();
*num = 6;
```

## ⚡ Async

```rust
// Async функция
async fn async_function() -> Result<(), Error> {
    let result = other_async_function().await?;
    Ok(())
}

// Запуск async кода
#[tokio::main]
async fn main() {
    async_function().await;
}

// Параллельное выполнение
use futures::join;
let (r1, r2) = join!(future1, future2);
```

## 🔮 Макросы

```rust
// Вызов макросов
println!("Hello, world!");
vec![1, 2, 3];
format!("Test {}", 42);

// Простой macro_rules!
macro_rules! my_macro {
    () => {
        println!("Hello from macro!");
    };
    ($val:expr) => {
        println!("Value: {}", $val);
    };
}

my_macro!();
my_macro!(42);
```

## 📦 Cargo команды

```bash
cargo new project_name    # новый проект
cargo build              # сборка
cargo run                # запуск
cargo test               # тесты
cargo doc                # документация
cargo publish            # публикация на crates.io

cargo build --release    # оптимизированная сборка
cargo check             # быстрая проверка
cargo fmt               # форматирование
cargo clippy            # линтер
cargo update            # обновление зависимостей
```

## 🎯 Атрибуты

```rust
#[derive(Debug, Clone, PartialEq)]  // автоматическая реализация
#[cfg(test)]                        // компилировать только для тестов
#[allow(dead_code)]                 // разрешить неиспользуемый код
#[warn(missing_docs)]               // предупреждение об отсутствии документации
#[inline]                           // подсказка для инлайнинга
#[must_use]                         // предупреждение если результат не используется
```

## 🧪 Тестирование

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_name() {
        assert_eq!(2 + 2, 4);
        assert_ne!(2 + 2, 5);
        assert!(true);
    }
    
    #[test]
    #[should_panic(expected = "panic message")]
    fn test_panic() {
        panic!("panic message");
    }
    
    #[test]
    #[ignore]
    fn expensive_test() {
        // долгий тест
    }
}
```

---
#rust #cheatsheet #reference #syntax
