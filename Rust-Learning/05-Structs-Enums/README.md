# 🏗️ Структуры и перечисления в Rust

Структуры и енамы позволяют создавать пользовательские типы данных для моделирования предметной области.

## 📋 Содержание

### Структуры (Structs)
- [[Struct-Definition]] - Определение структур
- [[Struct-Methods]] - Методы и ассоциированные функции
- [[Struct-Fields]] - Поля и видимость
- [[Tuple-Structs]] - Кортежные структуры
- [[Unit-Structs]] - Единичные структуры

### Перечисления (Enums)
- [[Enum-Definition]] - Определение перечислений
- [[Enum-Variants]] - Варианты с данными
- [[Pattern-Matching]] - Сопоставление с образцом
- [[Option-Result]] - Option и Result как специальные енамы

### Продвинутые техники
- [[Impl-Blocks]] - Множественные impl блоки
- [[Associated-Functions]] - Ассоциированные функции
- [[Destructuring]] - Деструктуризация
- [[Struct-Update-Syntax]] - Синтаксис обновления структур

## 🏗️ Основы структур

### Определение структуры
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// Создание экземпляра
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

### Доступ к полям
```rust
// Чтение полей
println!("Username: {}", user1.username);

// Изменение полей (только для mut экземпляров)
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

### Функции-конструкторы
```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,      // Сокращённый синтаксис
        username,   // когда имя переменной = имени поля
        active: true,
        sign_in_count: 1,
    }
}
```

### Синтаксис обновления структур
```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1 // Остальные поля копируются из user1
};

// ⚠️ После этого user1 частично перемещён!
// Доступны только active и sign_in_count (Copy типы)
```

## 🎯 Методы структур

### Определение методов
```rust
impl User {
    // Метод с &self (неизменяемое заимствование)
    fn is_active(&self) -> bool {
        self.active
    }
    
    // Метод с &mut self (изменяемое заимствование)
    fn deactivate(&mut self) {
        self.active = false;
    }
    
    // Метод с self (потребляющий метод)
    fn into_email(self) -> String {
        self.email
    }
    
    // Ассоциированная функция (конструктор)
    fn new(email: String, username: String) -> Self {
        Self {
            email,
            username,
            active: true,
            sign_in_count: 1,
        }
    }
}

// Использование
let mut user = User::new(
    String::from("test@example.com"),
    String::from("testuser")
);

println!("Active: {}", user.is_active());
user.deactivate();
let email = user.into_email(); // user больше недоступен
```

### Множественные impl блоки
```rust
impl User {
    fn get_username(&self) -> &str {
        &self.username
    }
}

impl User {
    fn increment_sign_in(&mut self) {
        self.sign_in_count += 1;
    }
}
```

## 📦 Типы структур

### Обычные структуры
```rust
struct Rectangle {
    width: u32,
    height: u32,
}
```

### Кортежные структуры
```rust
struct Color(i32, i32, i32);  // RGB
struct Point(i32, i32, i32);  // 3D координаты

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);

// Доступ по индексу
println!("Red component: {}", black.0);
```

### Единичные структуры
```rust
struct AlwaysEqual;  // Тип без данных

let subject = AlwaysEqual;

// Полезны для реализации трейтов без данных
impl SomeTrait for AlwaysEqual {
    // реализация
}
```

## 🎭 Перечисления (Enums)

### Базовое определение
```rust
enum IpAddrKind {
    V4,
    V6,
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

### Енамы с данными
```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

### Сложные варианты
```rust
enum Message {
    Quit,                       // Единичный вариант
    Move { x: i32, y: i32 },   // Структурный вариант
    Write(String),              // Кортежный вариант
    ChangeColor(i32, i32, i32), // Кортежный с несколькими значениями
}
```

### Методы для енамов
```rust
impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => {
                println!("Move to ({}, {})", x, y);
            }
            Message::Write(text) => println!("Text: {}", text),
            Message::ChangeColor(r, g, b) => {
                println!("Change color to RGB({}, {}, {})", r, g, b);
            }
        }
    }
}

let m = Message::Write(String::from("hello"));
m.call(); // Text: hello
```

## 🎯 Pattern Matching

### Match выражения
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

enum UsState {
    Alabama,
    Alaska,
    // ... остальные штаты
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

### If let для простых случаев
```rust
let some_value = Some(3);

// Вместо match с одним вариантом:
match some_value {
    Some(3) => println!("three"),
    _ => (),
}

// Можно использовать if let:
if let Some(3) = some_value {
    println!("three");
}
```

### While let
```rust
let mut stack = Vec::new();
stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
// Выведет: 3, 2, 1
```

## 💎 Option и Result

### Option<T>
```rust
enum Option<T> {
    None,
    Some(T),
}

fn divide(numerator: f64, denominator: f64) -> Option<f64> {
    if denominator == 0.0 {
        None
    } else {
        Some(numerator / denominator)
    }
}

// Использование
match divide(2.0, 3.0) {
    Some(result) => println!("Result: {}", result),
    None => println!("Cannot divide by zero"),
}
```

### Result<T, E>
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
                Err(e) => panic!("Problem creating file: {:?}", e),
            },
            other_error => panic!("Problem opening file: {:?}", other_error),
        },
    };
}
```

## 🔧 Продвинутые техники

### Деструктуризация
```rust
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 0, y: 7 };

// Деструктуризация в let
let Point { x, y } = p;
println!("x: {}, y: {}", x, y);

// Деструктуризация с переименованием
let Point { x: a, y: b } = p;
println!("a: {}, b: {}", a, b);

// Частичная деструктуризация
let Point { x, .. } = p; // игнорируем y
```

### Вложенные структуры
```rust
struct Address {
    street: String,
    city: String,
    zipcode: String,
}

struct Person {
    name: String,
    address: Address,
}

let person = Person {
    name: String::from("John"),
    address: Address {
        street: String::from("123 Main St"),
        city: String::from("Anytown"),
        zipcode: String::from("12345"),
    },
};

// Доступ к вложенным полям
println!("City: {}", person.address.city);
```

### Енамы с обобщениями
```rust
enum MyResult<T, E> {
    Success(T),
    Failure(E),
}

impl<T, E> MyResult<T, E> {
    fn is_success(&self) -> bool {
        matches!(self, MyResult::Success(_))
    }
    
    fn unwrap(self) -> T
    where
        E: std::fmt::Debug,
    {
        match self {
            MyResult::Success(val) => val,
            MyResult::Failure(err) => panic!("Unwrap failed: {:?}", err),
        }
    }
}
```

## 📊 Практические примеры

### Моделирование игрового состояния
```rust
#[derive(Debug, Clone)]
struct Player {
    name: String,
    health: u32,
    position: Point,
    inventory: Vec<Item>,
}

#[derive(Debug, Clone)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Debug, Clone)]
enum Item {
    Weapon { name: String, damage: u32 },
    Potion { name: String, healing: u32 },
    Key { id: u32 },
}

impl Player {
    fn new(name: String) -> Self {
        Player {
            name,
            health: 100,
            position: Point { x: 0, y: 0 },
            inventory: Vec::new(),
        }
    }
    
    fn move_to(&mut self, x: i32, y: i32) {
        self.position = Point { x, y };
    }
    
    fn add_item(&mut self, item: Item) {
        self.inventory.push(item);
    }
    
    fn use_potion(&mut self) -> bool {
        for (i, item) in self.inventory.iter().enumerate() {
            if let Item::Potion { healing, .. } = item {
                self.health += healing;
                self.inventory.remove(i);
                return true;
            }
        }
        false
    }
}
```

### HTTP запросы
```rust
#[derive(Debug)]
enum HttpMethod {
    Get,
    Post,
    Put,
    Delete,
}

#[derive(Debug)]
struct HttpRequest {
    method: HttpMethod,
    url: String,
    headers: Vec<(String, String)>,
    body: Option<String>,
}

impl HttpRequest {
    fn get(url: &str) -> Self {
        HttpRequest {
            method: HttpMethod::Get,
            url: url.to_string(),
            headers: Vec::new(),
            body: None,
        }
    }
    
    fn post(url: &str, body: String) -> Self {
        HttpRequest {
            method: HttpMethod::Post,
            url: url.to_string(),
            headers: Vec::new(),
            body: Some(body),
        }
    }
    
    fn add_header(mut self, key: &str, value: &str) -> Self {
        self.headers.push((key.to_string(), value.to_string()));
        self
    }
}

// Использование
let request = HttpRequest::get("https://api.example.com/users")
    .add_header("Authorization", "Bearer token123")
    .add_header("Content-Type", "application/json");
```

## 💡 Лучшие практики

### 1. Предпочитайте енамы большим количествам булевых флагов
```rust
// Вместо:
struct Config {
    debug: bool,
    verbose: bool,
    quiet: bool,
}

// Лучше:
enum LogLevel {
    Quiet,
    Normal,
    Verbose,
    Debug,
}

struct Config {
    log_level: LogLevel,
}
```

### 2. Используйте новые типы для доменной логики
```rust
// Вместо примитивных типов:
fn transfer_money(from: u32, to: u32, amount: f64) -> Result<(), String> {
    // Легко перепутать параметры местами!
}

// Лучше новые типы:
#[derive(Debug, Clone, Copy)]
struct UserId(u32);

#[derive(Debug, Clone, Copy)]
struct Amount(f64);

fn transfer_money(from: UserId, to: UserId, amount: Amount) -> Result<(), String> {
    // Теперь нельзя перепутать!
}
```

### 3. Реализуйте полезные трейты
```rust
#[derive(Debug, Clone, PartialEq, Eq)]
struct User {
    id: u32,
    name: String,
}

impl Default for User {
    fn default() -> Self {
        User {
            id: 0,
            name: String::from("Anonymous"),
        }
    }
}
```

## 🎯 Упражнения для практики

### Начальный уровень
1. Создайте структуру `Book` с полями title, author, pages
2. Реализуйте методы для структуры `Rectangle` (area, perimeter)
3. Создайте енам для дней недели с методом `is_weekend`

### Средний уровень
1. Моделируйте систему заказов с енамами для статуса
2. Создайте калькулятор с енамом для операций
3. Реализуйте простую файловую систему с директориями и файлами

### Продвинутый уровень
1. Создайте AST (Abstract Syntax Tree) для простого языка
2. Реализуйте state machine с енамами
3. Создайте систему типов с рекурсивными енамами

## ✅ Чек-лист изучения
- [ ] Создаю и использую структуры
- [ ] Определяю методы для структур
- [ ] Работаю с енамами и их вариантами
- [ ] Использую pattern matching с match
- [ ] Применяю if let и while let
- [ ] Понимаю Option и Result
- [ ] Моделирую предметную область с помощью типов

## 🔗 Связанные темы
- [[08-Traits/README]] - Реализация трейтов для структур
- [[Pattern-Matching-Advanced]] - Продвинутое pattern matching
- [[Memory-Layout]] - Расположение структур в памяти

#structs #enums #pattern-matching #data-modeling #intermediate