# 🔧 Functions in Rust

## 🎯 Основы функций

### Объявление функций
```rust
fn main() {
    println!("Hello, world!");
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

### Параметры функций
```rust
fn print_value(x: i32) {
    println!("The value is: {}", x);
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

## ↩️ Возвращаемые значения

### Явный return
```rust
fn five() -> i32 {
    return 5;
}
```

### Неявный возврат (последнее выражение)
```rust
fn plus_one(x: i32) -> i32 {
    x + 1  // Без точки с запятой!
}
```

### Множественные возвращаемые значения
```rust
fn calculate(x: i32, y: i32) -> (i32, i32) {
    (x + y, x * y)
}

fn main() {
    let (sum, product) = calculate(5, 3);
    println!("Sum: {}, Product: {}", sum, product);
}
```

## 📋 Statements vs Expressions

### Statements (утверждения)
```rust
fn main() {
    let y = 6; // statement
    
    // Это НЕ работает:
    // let x = (let y = 6); // statement не возвращает значение
}
```

### Expressions (выражения)
```rust
fn main() {
    let y = {
        let x = 3;
        x + 1  // expression, возвращает 4
    };
    
    println!("The value of y is: {}", y);
}
```

## 🎭 Типы функций

### Обычные функции
```rust
fn regular_function(x: i32) -> i32 {
    x * 2
}
```

### Ассоциированные функции
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Ассоциированная функция (вызывается через ::)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
    
    // Метод (вызывается через .)
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

let rect = Rectangle::new(30, 50);
let area = rect.area();
```

### Замыкания (Closures)
```rust
fn main() {
    let add_one = |x: i32| -> i32 { x + 1 };
    let result = add_one(5);
    
    // Краткая форма
    let multiply = |x, y| x * y;
    let product = multiply(3, 4);
    
    // Захват переменных
    let factor = 2;
    let scale = |x| x * factor;
    println!("Scaled: {}", scale(5));
}
```

## 🔀 Функции высшего порядка

### Функции как параметры
```rust
fn apply_operation<F>(x: i32, f: F) -> i32
where
    F: Fn(i32) -> i32,
{
    f(x)
}

fn double(x: i32) -> i32 {
    x * 2
}

fn main() {
    let result = apply_operation(5, double);
    let result2 = apply_operation(5, |x| x * 3);
}
```

### Функции как возвращаемые значения
```rust
fn make_adder(y: i32) -> impl Fn(i32) -> i32 {
    move |x| x + y
}

fn main() {
    let add_5 = make_adder(5);
    let result = add_5(10); // 15
}
```

## 🔄 Рекурсия

```rust
fn factorial(n: u32) -> u32 {
    match n {
        0 => 1,
        _ => n * factorial(n - 1),
    }
}

// Хвостовая рекурсия
fn factorial_tail(n: u32, acc: u32) -> u32 {
    match n {
        0 => acc,
        _ => factorial_tail(n - 1, n * acc),
    }
}
```

## 🎯 Дженерик функции

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
}
```

## 🔗 Связанные концепции

- [[01_Core/01_Basics/01_Variables|Variables]] - переменные в функциях
- [[01_Core/01_Basics/04_Control_Flow|Control Flow]] - управление потоком
- [[01_Core/06_Traits/00_Index|Traits]] - trait bounds для функций
- [[01_Core/07_Generics/00_Index|Generics]] - обобщенные функции

## 💻 Практические примеры

### Пример 1: Функция валидации
```rust
fn validate_email(email: &str) -> Result<(), String> {
    if email.is_empty() {
        return Err("Email cannot be empty".to_string());
    }
    
    if !email.contains('@') {
        return Err("Email must contain @".to_string());
    }
    
    Ok(())
}
```

### Пример 2: Builder pattern
```rust
struct ConfigBuilder {
    host: Option<String>,
    port: Option<u16>,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            host: None,
            port: None,
        }
    }
    
    fn host(mut self, host: String) -> Self {
        self.host = Some(host);
        self
    }
    
    fn port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }
    
    fn build(self) -> Result<Config, String> {
        Ok(Config {
            host: self.host.ok_or("Host is required")?,
            port: self.port.unwrap_or(8080),
        })
    }
}
```

### Пример 3: Функциональный стиль
```rust
fn process_numbers(numbers: Vec<i32>) -> Vec<i32> {
    numbers
        .iter()
        .filter(|&&x| x > 0)
        .map(|&x| x * 2)
        .collect()
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: Как вернуть значение из функции без использования return?
A: Последнее выражение без точки с запятой является возвращаемым значением
<!--SR:!2024-01-22,5,280-->

#flashcard 
Q: В чем разница между statement и expression?
A: Statement не возвращает значение (например, let x = 5;), expression возвращает значение
<!--SR:!2024-01-23,4,265-->

#flashcard 
Q: Как объявить функцию с несколькими возвращаемыми значениями?
A: Использовать кортеж: fn func() -> (i32, i32) { (1, 2) }
<!--SR:!2024-01-24,3,250-->

## ⚠️ Частые ошибки

### Ошибка 1: Точка с запятой в возвращаемом выражении
```rust
// ❌ ОШИБКА
fn plus_one(x: i32) -> i32 {
    x + 1; // точка с запятой превращает в statement
}

// ✅ Правильно
fn plus_one(x: i32) -> i32 {
    x + 1  // без точки с запятой
}
```

### Ошибка 2: Отсутствие типов параметров
```rust
// ❌ ОШИБКА
fn print_value(x) {  // тип не указан
    println!("{}", x);
}

// ✅ Правильно
fn print_value(x: i32) {
    println!("{}", x);
}
```

### Ошибка 3: Несоответствие типа возврата
```rust
// ❌ ОШИБКА
fn get_value() -> i32 {
    if true {
        5
    }
    // не все пути возвращают значение
}

// ✅ Правильно
fn get_value() -> i32 {
    if true {
        5
    } else {
        0
    }
}
```

## 📝 Упражнения

1. **Напишите функцию проверки палиндрома**:
   - Принимает &str
   - Возвращает bool

2. **Создайте функцию высшего порядка**:
   - Принимает вектор и функцию
   - Применяет функцию ко всем элементам

3. **Реализуйте рекурсивную функцию**:
   - Вычисление чисел Фибоначчи
   - Добавьте мемоизацию

## 📚 Дополнительные ресурсы

- [Rust Book - Functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html)
- [[01_Core/01_Basics/00_Index|Basics Index]]
- [[Closures and Iterators|Advanced Functions]]

---
#rust #functions #basics #core
