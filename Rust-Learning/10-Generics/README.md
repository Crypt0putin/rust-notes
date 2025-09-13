# 🔧 Обобщённые типы (Generics) в Rust

Generics позволяют писать код, который работает с разными типами, сохраняя при этом безопасность типов и производительность.

## 📋 Содержание

### Основы дженериков
- [[Generic-Functions]] - Обобщённые функции
- [[Generic-Structs]] - Обобщённые структуры
- [[Generic-Enums]] - Обобщённые перечисления
- [[Generic-Methods]] - Методы с дженериками

### Продвинутые техники
- [[Associated-Types-vs-Generics]] - Когда использовать что
- [[Generic-Constants]] - Обобщённые константы
- [[Generic-Lifetime-Bounds]] - Дженерики с lifetimes
- [[Higher-Kinded-Types]] - Типы высшего порядка

## 🎯 Зачем нужны дженерики?

### Проблема без дженериков
```rust
// Функция только для i32
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// Та же логика для char
fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// Дублирование кода!
```

### Решение с дженериками
```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// Теперь одна функция работает с любым типом T,
// который можно сравнивать (PartialOrd) и копировать (Copy)
```

## 🔧 Обобщённые функции

### Базовый синтаксис
```rust
// T - параметр типа (type parameter)
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>());
}

fn main() {
    print_type_of(&42);      // i32
    print_type_of(&"hello"); // &str
    print_type_of(&vec![1, 2, 3]); // alloc::vec::Vec<i32>
}
```

### Множественные параметры типа
```rust
fn combine<T, U>(first: T, second: U) -> (T, U) {
    (first, second)
}

fn main() {
    let result = combine(42, "hello");
    println!("{:?}", result); // (42, "hello")
}
```

### С trait bounds (ограничениями)
```rust
use std::fmt::Display;

fn compare_and_display<T: PartialOrd + Display>(x: T, y: T) {
    if x >= y {
        println!("The largest member is {}", x);
    } else {
        println!("The largest member is {}", y);
    }
}
```

## 🏗️ Обобщённые структуры

### Структура с одним типом
```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }
}

// Специализированная реализация для f64
impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let integer_point = Point::new(5, 10);
    let float_point = Point::new(1.0, 4.0);
    
    println!("Distance: {}", float_point.distance_from_origin());
}
```

### Структура с множественными типами
```rust
struct Rectangle<T, U> {
    width: T,
    height: U,
}

impl<T, U> Rectangle<T, U> {
    fn width(&self) -> &T {
        &self.width
    }
    
    fn height(&self) -> &U {
        &self.height
    }
}

// Смешивание типов в методах
impl<T, U> Rectangle<T, U> {
    fn mixup<V, W>(self, other: Rectangle<V, W>) -> Rectangle<T, W> {
        Rectangle {
            width: self.width,
            height: other.height,
        }
    }
}
```

## 📦 Обобщённые перечисления

### Классические примеры из стандартной библиотеки
```rust
// Option<T> - может содержать значение типа T или ничего
enum Option<T> {
    Some(T),
    None,
}

// Result<T, E> - успешный результат T или ошибка E  
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Пользовательские обобщённые enums
```rust
#[derive(Debug)]
enum Message<T> {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    Payload(T),
}

impl<T> Message<T> {
    fn process(self) -> Option<T> {
        match self {
            Message::Payload(data) => Some(data),
            _ => None,
        }
    }
}

fn main() {
    let msg1: Message<i32> = Message::Payload(42);
    let msg2: Message<String> = Message::Write("Hello".to_string());
    
    if let Some(data) = msg1.process() {
        println!("Payload: {}", data);
    }
}
```

## 🎯 Trait bounds и where clauses

### Inline trait bounds
```rust
fn notify<T: Display + Clone>(item: T) {
    println!("Breaking news! {}", item);
    let _copy = item.clone();
}
```

### Where clause для сложных ограничений
```rust
fn some_function<T, U>(t: T, u: U) -> String
where
    T: Display + Clone,
    U: Clone + Debug,
{
    format!("t: {}, u: {:?}", t, u.clone())
}
```

### Conditional implementations
```rust
struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// Метод доступен только если T реализует Display + PartialOrd
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## 🔬 Продвинутые техники

### Associated types vs Generics
```rust
// С дженериками - можно иметь множественные реализации
trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}

// С associated types - одна реализация на тип
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

// Associated types используются когда есть естественная связь
// между типом и его "продуктом"
```

### Phantom types
```rust
use std::marker::PhantomData;

struct RawPointer<T> {
    ptr: *const u8,
    _phantom: PhantomData<T>,
}

impl<T> RawPointer<T> {
    fn new(ptr: *const T) -> Self {
        RawPointer {
            ptr: ptr as *const u8,
            _phantom: PhantomData,
        }
    }
}

// PhantomData<T> позволяет структуре "знать" о типе T
// без фактического хранения значения типа T
```

### Higher-Ranked Trait Bounds (HRTB)
```rust
fn apply_to_all<F>(f: F) 
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    let strings = vec!["hello", "world"];
    for s in strings {
        println!("{}", f(s));
    }
}
```

## 📊 Практические примеры

### Обобщённая коллекция
```rust
#[derive(Debug)]
struct MyVec<T> {
    items: Vec<T>,
}

impl<T> MyVec<T> {
    fn new() -> Self {
        MyVec {
            items: Vec::new(),
        }
    }
    
    fn push(&mut self, item: T) {
        self.items.push(item);
    }
    
    fn pop(&mut self) -> Option<T> {
        self.items.pop()
    }
    
    fn len(&self) -> usize {
        self.items.len()
    }
}

// Дополнительные методы только для типов с Clone
impl<T: Clone> MyVec<T> {
    fn duplicate_last(&mut self) {
        if let Some(last) = self.items.last() {
            self.items.push(last.clone());
        }
    }
}

// Дополнительные методы только для типов с Display  
impl<T: Display> MyVec<T> {
    fn print_all(&self) {
        for item in &self.items {
            println!("{}", item);
        }
    }
}
```

### Обобщённый кэш
```rust
use std::collections::HashMap;
use std::hash::Hash;

struct Cache<K, V> 
where
    K: Eq + Hash,
{
    storage: HashMap<K, V>,
}

impl<K, V> Cache<K, V> 
where
    K: Eq + Hash,
{
    fn new() -> Self {
        Cache {
            storage: HashMap::new(),
        }
    }
    
    fn get(&self, key: &K) -> Option<&V> {
        self.storage.get(key)
    }
    
    fn insert(&mut self, key: K, value: V) {
        self.storage.insert(key, value);
    }
    
    fn contains_key(&self, key: &K) -> bool {
        self.storage.contains_key(key)
    }
}

// Специализированные методы для String ключей
impl<V> Cache<String, V> {
    fn get_by_str(&self, key: &str) -> Option<&V> {
        self.storage.get(key)
    }
}
```

### Builder pattern с дженериками
```rust
#[derive(Debug)]
struct Config<T> {
    name: String,
    value: T,
    enabled: bool,
}

struct ConfigBuilder<T> {
    name: Option<String>,
    value: Option<T>,
    enabled: bool,
}

impl<T> ConfigBuilder<T> {
    fn new() -> Self {
        ConfigBuilder {
            name: None,
            value: None,
            enabled: false,
        }
    }
    
    fn name(mut self, name: impl Into<String>) -> Self {
        self.name = Some(name.into());
        self
    }
    
    fn value(mut self, value: T) -> Self {
        self.value = Some(value);
        self
    }
    
    fn enabled(mut self, enabled: bool) -> Self {
        self.enabled = enabled;
        self
    }
    
    fn build(self) -> Result<Config<T>, String> {
        Ok(Config {
            name: self.name.ok_or("Name is required")?,
            value: self.value.ok_or("Value is required")?,
            enabled: self.enabled,
        })
    }
}

fn main() -> Result<(), String> {
    let config: Config<i32> = ConfigBuilder::new()
        .name("max_connections")
        .value(100)
        .enabled(true)
        .build()?;
        
    println!("{:?}", config);
    Ok(())
}
```

## ⚡ Производительность дженериков

### Monomorphization
```rust
// Этот код:
fn print_value<T: Display>(value: T) {
    println!("{}", value);
}

fn main() {
    print_value(5);
    print_value("hello");
}

// Компилируется примерно в это:
fn print_value_i32(value: i32) {
    println!("{}", value);
}

fn print_value_str(value: &str) {
    println!("{}", value);
}

fn main() {
    print_value_i32(5);
    print_value_str("hello");
}
```

**Zero-cost abstraction**: Дженерики не добавляют runtime накладных расходов!

## 💡 Лучшие практики

### 1. Используйте осмысленные имена типов
```rust
// Вместо общих T, U, V:
fn process<T, U>(input: T) -> U { ... }

// Лучше описательные имена:
fn process<Input, Output>(input: Input) -> Output { ... }

// Или в контексте:
fn serialize<Data, Format>(data: Data) -> Format { ... }
```

### 2. Минимизируйте trait bounds
```rust
// Не добавляйте лишние ограничения:
fn store<T: Clone + Debug + Display>(value: T) { 
    // если используется только Debug
}

// Лучше:
fn store<T: Debug>(value: T) {
    println!("Storing: {:?}", value);
}
```

### 3. Предпочитайте associated types для "естественных" связей
```rust
// Когда есть естественная связь между типом и его продуктом:
trait Collect {
    type Item;
    type Output;
    fn collect(items: Vec<Self::Item>) -> Self::Output;
}

// Вместо множественных generic параметров:
trait Collect<Item, Output> {
    fn collect(items: Vec<Item>) -> Output;
}
```

## 🚨 Частые ошибки

### Неправильные trait bounds
```rust
// ❌ Слишком строгие ограничения
fn process<T: Clone + Debug + Display + PartialEq>(value: T) {
    println!("{:?}", value); // Используется только Debug!
}

// ✅ Минимальные необходимые ограничения
fn process<T: Debug>(value: T) {
    println!("{:?}", value);
}
```

### Неправильное использование lifetimes с дженериками
```rust
// ❌ Неправильно:
fn get_default<T>() -> &T {
    // Нельзя вернуть ссылку без источника
}

// ✅ Правильно:
fn get_default<T: Default>() -> T {
    T::default()
}
```

## 🎯 Упражнения для практики

### Начальный уровень
1. Создайте обобщённую структуру `Pair<T>` с методами `new` и `get_values`
2. Напишите функцию `swap<T>` для обмена двух значений
3. Реализуйте обобщённый `Stack<T>` с методами `push`, `pop`, `is_empty`

### Средний уровень
1. Создайте обобщённый `Result<T, E>` с методами map, and_then, unwrap_or
2. Реализуйте trait `Reduce<T>` для сворачивания коллекций
3. Напишите обобщённую структуру `Tree<T>` с базовыми операциями

### Продвинутый уровень
1. Реализуйте type-safe builder pattern с состояниями
2. Создайте обобщённый парсер с trait bounds
3. Экспериментируйте с Higher-Ranked Trait Bounds

## 🔗 Связанные темы
- [[08-Traits/README]] - Трейты и дженерики
- [[09-Lifetimes/README]] - Lifetimes в дженериках
- [[Associated-Types]] - Ассоциированные типы

#generics #type-system #zero-cost #advanced #polymorphism