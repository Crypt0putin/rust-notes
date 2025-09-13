# 🔧 Методы структур

Методы позволяют определять функции, связанные со структурами, обеспечивая инкапсуляцию и удобство использования.

## 🎯 Основы методов

### Блоки impl
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Метод с &self (неизменяемое заимствование)
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // Метод с &mut self (изменяемое заимствование)
    fn double_size(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
    
    // Метод с self (потребляющий метод)
    fn into_square(self) -> Rectangle {
        let size = std::cmp::max(self.width, self.height);
        Rectangle {
            width: size,
            height: size,
        }
    }
}
```

### Использование методов
```rust
fn main() {
    let mut rect = Rectangle {
        width: 30,
        height: 50,
    };
    
    // Вызов метода с &self
    println!("Area: {}", rect.area());
    
    // Вызов метода с &mut self
    rect.double_size();
    
    // Вызов метода с self (rect становится недоступен)
    let square = rect.into_square();
}
```

## 🏗️ Ассоциированные функции

### Функции-конструкторы
```rust
impl Rectangle {
    // Ассоциированная функция (нет self)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
    
    // Конструктор квадрата
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
    
    // Конструктор по умолчанию
    fn default() -> Rectangle {
        Rectangle {
            width: 1,
            height: 1,
        }
    }
}

// Использование ассоциированных функций
let rect1 = Rectangle::new(30, 50);
let rect2 = Rectangle::square(25);
let rect3 = Rectangle::default();
```

## 🎭 Типы параметров self

### &self - неизменяемое заимствование
```rust
impl Rectangle {
    fn width(&self) -> u32 {
        self.width
    }
    
    fn height(&self) -> u32 {
        self.height
    }
    
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

// Использование
let rect1 = Rectangle::new(30, 50);
let rect2 = Rectangle::new(10, 40);

println!("Width: {}", rect1.width());
println!("Can hold: {}", rect1.can_hold(&rect2));
// rect1 и rect2 остаются доступными
```

### &mut self - изменяемое заимствование
```rust
impl Rectangle {
    fn set_width(&mut self, width: u32) {
        self.width = width;
    }
    
    fn scale(&mut self, factor: f64) {
        self.width = (self.width as f64 * factor) as u32;
        self.height = (self.height as f64 * factor) as u32;
    }
    
    fn reset(&mut self) {
        self.width = 0;
        self.height = 0;
    }
}

// Использование
let mut rect = Rectangle::new(30, 50);
rect.set_width(40);
rect.scale(1.5);
// rect остаётся доступным, но изменённым
```

### self - потребляющие методы
```rust
impl Rectangle {
    fn into_tuple(self) -> (u32, u32) {
        (self.width, self.height)
    }
    
    fn combine_with(self, other: Rectangle) -> Rectangle {
        Rectangle {
            width: self.width + other.width,
            height: self.height + other.height,
        }
    }
}

// Использование
let rect1 = Rectangle::new(30, 50);
let rect2 = Rectangle::new(20, 30);
let combined = rect1.combine_with(rect2);
// rect1 и rect2 больше не доступны!
```

## 🔄 Множественные impl блоки

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

// Первый impl блок - основные методы
impl Rectangle {
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
    
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

// Второй impl блок - методы сравнения
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
    
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

// Третий impl блок - методы изменения
impl Rectangle {
    fn set_width(&mut self, width: u32) {
        self.width = width;
    }
    
    fn set_height(&mut self, height: u32) {
        self.height = height;
    }
}
```

## 🎯 Практические паттерны

### Builder паттерн
```rust
struct User {
    name: String,
    email: String,
    age: Option<u32>,
    active: bool,
}

impl User {
    fn new(name: String, email: String) -> Self {
        User {
            name,
            email,
            age: None,
            active: true,
        }
    }
    
    fn with_age(mut self, age: u32) -> Self {
        self.age = Some(age);
        self
    }
    
    fn inactive(mut self) -> Self {
        self.active = false;
        self
    }
}

// Использование builder паттерна
let user = User::new("Alice".to_string(), "alice@example.com".to_string())
    .with_age(30)
    .inactive();
```

### Fluent interface (цепочка методов)
```rust
struct Calculator {
    value: f64,
}

impl Calculator {
    fn new(value: f64) -> Self {
        Calculator { value }
    }
    
    fn add(mut self, x: f64) -> Self {
        self.value += x;
        self
    }
    
    fn multiply(mut self, x: f64) -> Self {
        self.value *= x;
        self
    }
    
    fn divide(mut self, x: f64) -> Self {
        self.value /= x;
        self
    }
    
    fn result(self) -> f64 {
        self.value
    }
}

// Использование
let result = Calculator::new(10.0)
    .add(5.0)
    .multiply(2.0)
    .divide(3.0)
    .result();
```

### Методы валидации
```rust
struct Email {
    value: String,
}

impl Email {
    fn new(email: String) -> Result<Self, String> {
        if Self::is_valid(&email) {
            Ok(Email { value: email })
        } else {
            Err("Invalid email format".to_string())
        }
    }
    
    fn is_valid(email: &str) -> bool {
        email.contains('@') && email.contains('.')
    }
    
    fn domain(&self) -> &str {
        self.value.split('@').nth(1).unwrap_or("")
    }
    
    fn local_part(&self) -> &str {
        self.value.split('@').next().unwrap_or("")
    }
}
```

## 🚀 Продвинутые техники

### Методы с дженериками
```rust
struct Container<T> {
    value: T,
}

impl<T> Container<T> {
    fn new(value: T) -> Self {
        Container { value }
    }
    
    fn get(&self) -> &T {
        &self.value
    }
    
    fn set(&mut self, value: T) {
        self.value = value;
    }
    
    fn map<U, F>(self, f: F) -> Container<U>
    where
        F: FnOnce(T) -> U,
    {
        Container {
            value: f(self.value),
        }
    }
}

// Специализация для конкретных типов
impl Container<String> {
    fn len(&self) -> usize {
        self.value.len()
    }
    
    fn is_empty(&self) -> bool {
        self.value.is_empty()
    }
}
```

### Методы с ограничениями трейтов
```rust
impl<T> Container<T>
where
    T: Clone,
{
    fn duplicate(&self) -> Container<T> {
        Container {
            value: self.value.clone(),
        }
    }
}

impl<T> Container<T>
where
    T: std::fmt::Display,
{
    fn print(&self) {
        println!("Container contains: {}", self.value);
    }
}
```

## 💡 Лучшие практики

### 1. Группируйте связанные методы
```rust
impl Rectangle {
    // Конструкторы
    fn new(width: u32, height: u32) -> Self { /* ... */ }
    fn square(size: u32) -> Self { /* ... */ }
    
    // Геттеры
    fn width(&self) -> u32 { /* ... */ }
    fn height(&self) -> u32 { /* ... */ }
    
    // Вычисления
    fn area(&self) -> u32 { /* ... */ }
    fn perimeter(&self) -> u32 { /* ... */ }
    
    // Мутации
    fn set_width(&mut self, width: u32) { /* ... */ }
    fn scale(&mut self, factor: f64) { /* ... */ }
}
```

### 2. Предпочитайте &self когда возможно
```rust
impl Rectangle {
    // ✅ Хорошо - не изменяет состояние
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // ❌ Плохо - без необходимости берёт владение
    fn area_bad(self) -> u32 {
        self.width * self.height
    }
}
```

### 3. Используйте Self вместо имени типа
```rust
impl Rectangle {
    // ✅ Хорошо
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
    
    // ❌ Повторение имени типа
    fn new_verbose(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
}
```

### 4. Документируйте публичные методы
```rust
impl Rectangle {
    /// Creates a new rectangle with the given dimensions.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let rect = Rectangle::new(30, 50);
    /// assert_eq!(rect.area(), 1500);
    /// ```
    pub fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
    
    /// Calculates the area of the rectangle.
    pub fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

## 🔗 Связанные темы
- [[Struct-Definition]] - Определение структур
- [[Associated-Functions]] - Ассоциированные функции
- [[Impl-Blocks]] - Множественные impl блоки
- [[Self-Types]] - Типы Self и self

#struct-methods #impl #associated-functions #encapsulation #intermediate