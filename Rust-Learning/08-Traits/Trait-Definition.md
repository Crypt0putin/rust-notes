# 🎯 Определение трейтов

Трейты определяют общее поведение, которое типы могут реализовывать. Это основа полиморфизма в Rust.

## 🎯 Базовый синтаксис

### Простой трейт
```rust
trait Drawable {
    fn draw(&self);
}

// Реализация для структуры
struct Circle {
    radius: f64,
}

impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing a circle with radius {}", self.radius);
    }
}
```

### Трейт с множественными методами
```rust
trait Shape {
    fn area(&self) -> f64;
    fn perimeter(&self) -> f64;
    
    // Методы могут иметь параметры
    fn scale(&mut self, factor: f64);
    
    // Могут возвращать Self
    fn clone_shape(&self) -> Self
    where
        Self: Sized;
}
```

## 🔧 Методы по умолчанию

### Базовые методы по умолчанию
```rust
trait Greet {
    fn name(&self) -> &str;
    
    // Метод по умолчанию
    fn greet(&self) -> String {
        format!("Hello, my name is {}", self.name())
    }
    
    // Метод по умолчанию, использующий другой метод
    fn introduce(&self) -> String {
        format!("{}. Nice to meet you!", self.greet())
    }
}

struct Person {
    name: String,
}

impl Greet for Person {
    fn name(&self) -> &str {
        &self.name
    }
    
    // greet() и introduce() наследуются автоматически
    // Но можно переопределить:
    fn greet(&self) -> String {
        format!("Hi there! I'm {}", self.name())
    }
}
```

### Методы по умолчанию с логикой
```rust
trait Iterator {
    type Item;
    
    // Обязательный метод
    fn next(&mut self) -> Option<Self::Item>;
    
    // Методы по умолчанию
    fn count(mut self) -> usize 
    where 
        Self: Sized,
    {
        let mut count = 0;
        while let Some(_) = self.next() {
            count += 1;
        }
        count
    }
    
    fn collect<B: FromIterator<Self::Item>>(self) -> B
    where
        Self: Sized,
    {
        FromIterator::from_iter(self)
    }
}
```

## 🏷️ Ассоциированные типы

### Определение ассоциированных типов
```rust
trait Container {
    type Item;
    
    fn get(&self, index: usize) -> Option<&Self::Item>;
    fn set(&mut self, index: usize, item: Self::Item) -> bool;
    fn len(&self) -> usize;
}

// Реализация
struct IntVec {
    data: Vec<i32>,
}

impl Container for IntVec {
    type Item = i32;  // Конкретизируем ассоциированный тип
    
    fn get(&self, index: usize) -> Option<&Self::Item> {
        self.data.get(index)
    }
    
    fn set(&mut self, index: usize, item: Self::Item) -> bool {
        if index < self.data.len() {
            self.data[index] = item;
            true
        } else {
            false
        }
    }
    
    fn len(&self) -> usize {
        self.data.len()
    }
}
```

### Ассоциированные типы vs Дженерики
```rust
// С ассоциированными типами - одна реализация на тип
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

// С дженериками - множественные реализации возможны
trait Convert<T> {
    fn convert(&self) -> T;
}

// Можно реализовать Convert несколько раз:
impl Convert<String> for i32 { /* ... */ }
impl Convert<f64> for i32 { /* ... */ }

// Но Iterator только один раз для каждого типа
```

## 🌟 Продвинутые возможности

### Трейты с ограничениями
```rust
trait Summarizable {
    fn summary(&self) -> String;
}

trait Printable: Summarizable {  // Printable требует Summarizable
    fn print(&self) {
        println!("{}", self.summary());
    }
}

// Для реализации Printable нужно сначала реализовать Summarizable
struct Article {
    title: String,
    content: String,
}

impl Summarizable for Article {
    fn summary(&self) -> String {
        format!("{}: {}", self.title, &self.content[..50])
    }
}

impl Printable for Article {
    // print() наследуется, но можно переопределить
}
```

### Трейты с ассоциированными константами
```rust
trait MathConstants {
    const PI: f64;
    const E: f64 = 2.71828;  // Значение по умолчанию
    
    fn circle_area(radius: f64) -> f64 {
        Self::PI * radius * radius
    }
}

struct MyMath;

impl MathConstants for MyMath {
    const PI: f64 = 3.14159;
    // E наследуется со значением по умолчанию
}

// Использование
let area = MyMath::circle_area(5.0);
println!("PI value: {}", MyMath::PI);
```

## 🎯 Полезные паттерны

### Трейт для конвертации
```rust
trait FromString {
    type Error;
    
    fn from_string(s: &str) -> Result<Self, Self::Error>
    where
        Self: Sized;
}

#[derive(Debug)]
struct ParseIntError;

impl FromString for i32 {
    type Error = ParseIntError;
    
    fn from_string(s: &str) -> Result<Self, Self::Error> {
        s.parse().map_err(|_| ParseIntError)
    }
}
```

### Builder трейт
```rust
trait Builder {
    type Item;
    
    fn new() -> Self;
    fn build(self) -> Self::Item;
}

trait ConfigBuilder: Builder {
    fn with_host(self, host: &str) -> Self;
    fn with_port(self, port: u16) -> Self;
}

struct ServerConfigBuilder {
    host: Option<String>,
    port: Option<u16>,
}

impl Builder for ServerConfigBuilder {
    type Item = ServerConfig;
    
    fn new() -> Self {
        ServerConfigBuilder {
            host: None,
            port: None,
        }
    }
    
    fn build(self) -> Self::Item {
        ServerConfig {
            host: self.host.unwrap_or_else(|| "localhost".to_string()),
            port: self.port.unwrap_or(8080),
        }
    }
}

impl ConfigBuilder for ServerConfigBuilder {
    fn with_host(mut self, host: &str) -> Self {
        self.host = Some(host.to_string());
        self
    }
    
    fn with_port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }
}
```

### Visitor паттерн
```rust
trait Visitor<T> {
    type Output;
    
    fn visit(&mut self, item: &T) -> Self::Output;
}

trait Visitable {
    fn accept<V, T>(&self, visitor: &mut V) -> V::Output
    where
        V: Visitor<T>,
        Self: AsRef<T>;
}

// Пример использования
struct PrintVisitor;

impl Visitor<String> for PrintVisitor {
    type Output = ();
    
    fn visit(&mut self, item: &String) -> Self::Output {
        println!("Visiting: {}", item);
    }
}
```

## 🏗️ Marker трейты

### Пустые трейты для маркировки
```rust
// Marker trait - нет методов
trait ThreadSafe {}

// Автоматическая реализация для безопасных типов
struct SafeData {
    value: i32,
}

impl ThreadSafe for SafeData {}

// Функция, принимающая только thread-safe типы
fn process_safely<T: ThreadSafe>(data: T) {
    // Можем быть уверены в thread safety
}

// Или с Phantom types
use std::marker::PhantomData;

trait State {}
struct Initialized;
struct Uninitialized;

impl State for Initialized {}
impl State for Uninitialized {}

struct Connection<S: State> {
    _state: PhantomData<S>,
}

impl Connection<Uninitialized> {
    fn new() -> Self {
        Connection { _state: PhantomData }
    }
    
    fn initialize(self) -> Connection<Initialized> {
        Connection { _state: PhantomData }
    }
}

impl Connection<Initialized> {
    fn send_data(&self, data: &str) {
        println!("Sending: {}", data);
    }
}
```

## 💡 Лучшие практики

### 1. Предпочитайте маленькие, сфокусированные трейты
```rust
// ✅ Хорошо - один концепт
trait Draw {
    fn draw(&self);
}

trait Resize {
    fn resize(&mut self, scale: f64);
}

// ❌ Плохо - слишком много ответственности
trait GraphicsObject {
    fn draw(&self);
    fn resize(&mut self, scale: f64);
    fn rotate(&mut self, angle: f64);
    fn translate(&mut self, x: f64, y: f64);
    fn get_bounds(&self) -> Rectangle;
    fn set_color(&mut self, color: Color);
}
```

### 2. Используйте ассоциированные типы для естественных связей
```rust
// ✅ Хорошо - естественная связь
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

// ❌ Менее естественно
trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

### 3. Предоставляйте полезные методы по умолчанию
```rust
trait Container {
    type Item;
    
    fn get(&self, index: usize) -> Option<&Self::Item>;
    fn len(&self) -> usize;
    
    // Полезные методы по умолчанию
    fn is_empty(&self) -> bool {
        self.len() == 0
    }
    
    fn first(&self) -> Option<&Self::Item> {
        self.get(0)
    }
    
    fn last(&self) -> Option<&Self::Item> {
        if self.is_empty() {
            None
        } else {
            self.get(self.len() - 1)
        }
    }
}
```

### 4. Документируйте трейты хорошо
```rust
/// A trait for objects that can be serialized to a string format.
/// 
/// # Examples
/// 
/// ```
/// struct Person { name: String }
/// 
/// impl Serialize for Person {
///     fn serialize(&self) -> String {
///         format!("Person({})", self.name)
///     }
/// }
/// ```
trait Serialize {
    /// Converts the object to its string representation.
    /// 
    /// # Returns
    /// 
    /// A `String` containing the serialized form of the object.
    fn serialize(&self) -> String;
}
```

## 🔗 Связанные темы
- [[Trait-Implementation]] - Реализация трейтов
- [[Trait-Bounds]] - Ограничения трейтов
- [[Associated-Types]] - Подробнее об ассоциированных типах
- [[Default-Methods]] - Методы по умолчанию

#trait-definition #interfaces #polymorphism #associated-types #intermediate