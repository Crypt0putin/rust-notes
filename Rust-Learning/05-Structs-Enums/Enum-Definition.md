# 🎭 Определение перечислений (Enums)

Перечисления в Rust позволяют определить тип, который может быть одним из нескольких вариантов.

## 🎯 Базовый синтаксис

### Простое перечисление
```rust
enum IpAddrKind {
    V4,
    V6,
}

// Создание значений
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

### Использование в функциях
```rust
fn route(ip_kind: IpAddrKind) {
    match ip_kind {
        IpAddrKind::V4 => println!("IPv4 address"),
        IpAddrKind::V6 => println!("IPv6 address"),
    }
}

route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

## 📦 Варианты с данными

### Простые данные
```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));
```

### Разные типы данных
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
    Quit,                       // Без данных
    Move { x: i32, y: i32 },   // Именованные поля (как struct)
    Write(String),              // Одно значение
    ChangeColor(i32, i32, i32), // Кортеж значений
}
```

## 🎨 Методы для енамов

### Определение методов
```rust
impl Message {
    fn call(&self) {
        match self {
            Message::Quit => {
                println!("Quit message");
            }
            Message::Move { x, y } => {
                println!("Move to x: {}, y: {}", x, y);
            }
            Message::Write(text) => {
                println!("Text message: {}", text);
            }
            Message::ChangeColor(r, g, b) => {
                println!("Change color to RGB({}, {}, {})", r, g, b);
            }
        }
    }
    
    fn is_quit(&self) -> bool {
        matches!(self, Message::Quit)
    }
}

// Использование
let m = Message::Write(String::from("hello"));
m.call();
```

### Ассоциированные функции
```rust
impl Message {
    fn new_move(x: i32, y: i32) -> Self {
        Message::Move { x, y }
    }
    
    fn new_text(text: &str) -> Self {
        Message::Write(text.to_string())
    }
    
    fn quit() -> Self {
        Message::Quit
    }
}

// Использование
let move_msg = Message::new_move(10, 20);
let text_msg = Message::new_text("Hello");
let quit_msg = Message::quit();
```

## 🌟 Стандартные енамы

### Option<T>
```rust
enum Option<T> {
    None,
    Some(T),
}

// Примеры использования
let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None;

// Обработка Option
match some_number {
    Some(value) => println!("Got a value: {}", value),
    None => println!("No value"),
}
```

### Result<T, E>
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Пример функции, возвращающей Result
fn divide(dividend: f64, divisor: f64) -> Result<f64, String> {
    if divisor == 0.0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(dividend / divisor)
    }
}

// Использование
match divide(10.0, 2.0) {
    Ok(result) => println!("Result: {}", result),
    Err(error) => println!("Error: {}", error),
}
```

## 🔧 Продвинутые техники

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
    
    fn map<U, F>(self, f: F) -> MyResult<U, E>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            MyResult::Success(value) => MyResult::Success(f(value)),
            MyResult::Failure(error) => MyResult::Failure(error),
        }
    }
}
```

### Вложенные енамы
```rust
enum Expression {
    Number(i32),
    Add(Box<Expression>, Box<Expression>),
    Multiply(Box<Expression>, Box<Expression>),
}

impl Expression {
    fn evaluate(&self) -> i32 {
        match self {
            Expression::Number(n) => *n,
            Expression::Add(left, right) => {
                left.evaluate() + right.evaluate()
            }
            Expression::Multiply(left, right) => {
                left.evaluate() * right.evaluate()
            }
        }
    }
}

// Создание выражения: (2 + 3) * 4
let expr = Expression::Multiply(
    Box::new(Expression::Add(
        Box::new(Expression::Number(2)),
        Box::new(Expression::Number(3))
    )),
    Box::new(Expression::Number(4))
);

println!("Result: {}", expr.evaluate()); // 20
```

## 🎯 Практические примеры

### Система состояний
```rust
enum ConnectionState {
    Disconnected,
    Connecting { timeout: u32 },
    Connected { session_id: String },
    Error { code: u32, message: String },
}

impl ConnectionState {
    fn is_connected(&self) -> bool {
        matches!(self, ConnectionState::Connected { .. })
    }
    
    fn get_session_id(&self) -> Option<&str> {
        match self {
            ConnectionState::Connected { session_id } => Some(session_id),
            _ => None,
        }
    }
    
    fn transition_to_error(&mut self, code: u32, message: String) {
        *self = ConnectionState::Error { code, message };
    }
}
```

### Система команд
```rust
enum Command {
    Start,
    Stop,
    Restart { force: bool },
    Status,
    Configure { key: String, value: String },
}

struct Application {
    running: bool,
    config: std::collections::HashMap<String, String>,
}

impl Application {
    fn execute_command(&mut self, command: Command) -> Result<String, String> {
        match command {
            Command::Start => {
                if self.running {
                    Err("Application is already running".to_string())
                } else {
                    self.running = true;
                    Ok("Application started".to_string())
                }
            }
            Command::Stop => {
                if self.running {
                    self.running = false;
                    Ok("Application stopped".to_string())
                } else {
                    Err("Application is not running".to_string())
                }
            }
            Command::Restart { force } => {
                if self.running || force {
                    self.running = false;
                    self.running = true;
                    Ok("Application restarted".to_string())
                } else {
                    Err("Cannot restart: application is not running".to_string())
                }
            }
            Command::Status => {
                let status = if self.running { "running" } else { "stopped" };
                Ok(format!("Application is {}", status))
            }
            Command::Configure { key, value } => {
                self.config.insert(key.clone(), value);
                Ok(format!("Configuration updated: {} = {}", key, value))
            }
        }
    }
}
```

### HTTP статусы
```rust
#[derive(Debug, PartialEq)]
enum HttpStatus {
    Ok,
    NotFound,
    InternalServerError,
    BadRequest { message: String },
    Unauthorized,
    Forbidden,
}

impl HttpStatus {
    fn code(&self) -> u16 {
        match self {
            HttpStatus::Ok => 200,
            HttpStatus::NotFound => 404,
            HttpStatus::InternalServerError => 500,
            HttpStatus::BadRequest { .. } => 400,
            HttpStatus::Unauthorized => 401,
            HttpStatus::Forbidden => 403,
        }
    }
    
    fn message(&self) -> String {
        match self {
            HttpStatus::Ok => "OK".to_string(),
            HttpStatus::NotFound => "Not Found".to_string(),
            HttpStatus::InternalServerError => "Internal Server Error".to_string(),
            HttpStatus::BadRequest { message } => message.clone(),
            HttpStatus::Unauthorized => "Unauthorized".to_string(),
            HttpStatus::Forbidden => "Forbidden".to_string(),
        }
    }
    
    fn is_error(&self) -> bool {
        self.code() >= 400
    }
}
```

## 🎨 Derive макросы для енамов

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
enum Status {
    Pending,
    Processing { progress: u8 },
    Completed,
    Failed { error: String },
}

// Теперь можно:
let status1 = Status::Pending;
let status2 = status1.clone();  // Clone
println!("{:?}", status1);      // Debug
assert_eq!(status1, status2);   // PartialEq
```

## 💡 Лучшие практики

### 1. Используйте осмысленные имена
```rust
// ❌ Плохо
enum State {
    A,
    B,
    C,
}

// ✅ Хорошо
enum ConnectionState {
    Disconnected,
    Connecting,
    Connected,
}
```

### 2. Группируйте связанные данные
```rust
// ❌ Плохо - много отдельных енамов
enum UserType { Admin, User, Guest }
enum UserStatus { Active, Inactive, Banned }

// ✅ Лучше - один еном с данными
enum User {
    Admin { permissions: Vec<String> },
    RegularUser { status: Status },
    Guest,
}
```

### 3. Используйте Box для рекурсивных енамов
```rust
enum List {
    Empty,
    Node(i32, Box<List>),  // Box необходим для рекурсии
}
```

### 4. Предпочитайте енамы булевым флагам
```rust
// ❌ Много булевых флагов
struct Config {
    debug: bool,
    verbose: bool,
    quiet: bool,
}

// ✅ Еном для взаимоисключающих состояний
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

## 🔗 Связанные темы
- [[Pattern-Matching]] - Сопоставление с образцом
- [[Option-Result]] - Важные стандартные енамы
- [[Enum-Variants]] - Работа с вариантами енамов
- [[Match-Expressions]] - Match выражения

#enums #pattern-matching #variants #algebraic-types #intermediate