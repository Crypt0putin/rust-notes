# 🔄 Move Semantics in Rust

## 🎯 Что такое Move?

**Move** - это передача владения значением от одной переменной к другой. После move исходная переменная становится невалидной.

```rust
let s1 = String::from("hello");
let s2 = s1; // Move происходит здесь

// println!("{}", s1); // ОШИБКА: value borrowed here after move
println!("{}", s2); // OK: s2 теперь владеет данными
```

## 📊 Визуализация Move

### До Move:
```
Stack                      Heap
s1: String                ┌─────────────┐
  ptr ──────────────────> │  "hello"    │
  len: 5                  └─────────────┘
  capacity: 5
```

### После Move:
```
Stack                      Heap
s1: (невалидна)           ┌─────────────┐
s2: String                │  "hello"    │
  ptr ──────────────────> │             │
  len: 5                  └─────────────┘
  capacity: 5
```

## 🔀 Когда происходит Move?

### 1. Присваивание
```rust
let x = vec![1, 2, 3];
let y = x; // Move
// x больше не доступен
```

### 2. Передача в функцию
```rust
fn take_ownership(some_string: String) {
    println!("{}", some_string);
} // some_string выходит из области видимости и освобождается

let s = String::from("hello");
take_ownership(s); // s перемещается в функцию
// println!("{}", s); // ОШИБКА: s больше не валидна
```

### 3. Возврат из функции
```rust
fn gives_ownership() -> String {
    let some_string = String::from("hello");
    some_string // Владение передается вызывающему коду
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string // Владение возвращается
}

let s1 = gives_ownership();
let s2 = String::from("hello");
let s3 = takes_and_gives_back(s2); // s2 moved, s3 получает владение
```

### 4. В структурах и енумах
```rust
struct Container {
    data: String,
}

let s = String::from("hello");
let container = Container { data: s }; // s moved в структуру
// println!("{}", s); // ОШИБКА
```

## 🎭 Move vs Copy

### Типы, реализующие Copy
```rust
let x = 5;
let y = x; // Copy, не Move
println!("x = {}, y = {}", x, y); // Оба доступны

// Copy типы:
// - Все целочисленные типы
// - bool
// - char
// - f32, f64
// - Кортежи из Copy типов
// - Массивы из Copy типов
```

### Типы, использующие Move
```rust
// String
let s1 = String::from("hello");
let s2 = s1; // Move

// Vec
let v1 = vec![1, 2, 3];
let v2 = v1; // Move

// Box
let b1 = Box::new(5);
let b2 = b1; // Move

// Любые структуры без Copy trait
struct Point {
    x: i32,
    y: i32,
    label: String, // String не Copy, поэтому Point тоже
}
```

## 🔧 Управление Move

### Клонирование вместо Move
```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // Глубокое копирование
println!("s1 = {}, s2 = {}", s1, s2); // Оба доступны
```

### Заимствование вместо Move
```rust
fn calculate_length(s: &String) -> usize { // s - ссылка
    s.len()
} // s выходит из области, но не владеет данными

let s1 = String::from("hello");
let len = calculate_length(&s1); // Заимствование
println!("The length of '{}' is {}.", s1, len); // s1 все еще доступна
```

### Partial Move
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

let person = Person {
    name: String::from("Alice"),
    age: 30,
};

let Person { name, age } = person;
// person больше не доступен целиком
// name moved, age copied (u32 implements Copy)
println!("Name: {}, Age: {}", name, age);
// println!("{:?}", person); // ОШИБКА
```

## 🎯 Move в циклах

### Проблема с Move в цикле
```rust
let v = vec![1, 2, 3];

// ❌ ОШИБКА: v перемещается на первой итерации
// for i in v {
//     println!("{}", i);
// }
// for i in v { // v уже moved!
//     println!("{}", i);
// }

// ✅ Решение 1: Заимствование
for i in &v {
    println!("{}", i);
}
for i in &v { // OK, v не перемещена
    println!("{}", i);
}

// ✅ Решение 2: Клонирование
for i in v.clone() {
    println!("{}", i);
}
// v все еще доступна
```

## 🔄 Move и замыкания

### Захват по Move
```rust
let s = String::from("hello");

// Move замыкание
let closure = move || {
    println!("Inside closure: {}", s);
};

closure();
// println!("{}", s); // ОШИБКА: s moved в замыкание
```

### Частичный захват
```rust
let mut list = vec![1, 2, 3];

let mut borrows_mutably = || {
    list.push(4);
};

// list недоступен здесь для изменения
borrows_mutably();
// Теперь можно использовать list
```

## 💻 Практические примеры

### Пример 1: Builder Pattern с Move
```rust
struct Request {
    url: String,
    method: String,
    body: Option<String>,
}

struct RequestBuilder {
    url: Option<String>,
    method: Option<String>,
    body: Option<String>,
}

impl RequestBuilder {
    fn new() -> Self {
        RequestBuilder {
            url: None,
            method: None,
            body: None,
        }
    }
    
    fn url(mut self, url: String) -> Self {
        self.url = Some(url);
        self // Move self обратно
    }
    
    fn method(mut self, method: String) -> Self {
        self.method = Some(method);
        self
    }
    
    fn build(self) -> Result<Request, String> {
        Ok(Request {
            url: self.url.ok_or("URL required")?,
            method: self.method.unwrap_or_else(|| "GET".to_string()),
            body: self.body,
        })
    }
}
```

### Пример 2: Swap без копирования
```rust
use std::mem;

fn swap_ownership() {
    let mut s1 = String::from("hello");
    let mut s2 = String::from("world");
    
    // Обмен владением без копирования
    mem::swap(&mut s1, &mut s2);
    
    println!("s1: {}, s2: {}", s1, s2); // s1: world, s2: hello
}
```

### Пример 3: Option для временного Move
```rust
struct Container {
    data: Option<String>,
}

impl Container {
    fn process(&mut self) {
        // Временно забираем владение
        if let Some(mut data) = self.data.take() {
            data.push_str(" processed");
            // Возвращаем владение
            self.data = Some(data);
        }
    }
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: Что происходит с исходной переменной после Move?
A: Она становится невалидной и больше не может использоваться
<!--SR:!2024-01-31,4,270-->

#flashcard 
Q: Какие типы используют Move вместо Copy?
A: String, Vec, Box и любые структуры, содержащие non-Copy поля
<!--SR:!2024-02-01,5,280-->

#flashcard 
Q: Как передать значение в функцию без Move?
A: Использовать заимствование (&) или клонирование (.clone())
<!--SR:!2024-02-02,3,250-->

## ⚠️ Частые ошибки

### Ошибка 1: Использование после Move
```rust
// ❌ ОШИБКА
let s = String::from("hello");
let s2 = s;
println!("{}", s); // value borrowed here after move

// ✅ Решения
// Вариант 1: Clone
let s = String::from("hello");
let s2 = s.clone();
println!("{}", s);

// Вариант 2: Заимствование
let s = String::from("hello");
let s2 = &s;
println!("{}", s);
```

### Ошибка 2: Move в цикле
```rust
// ❌ ОШИБКА
let v = vec![String::from("a"), String::from("b")];
for s in v { // v moved здесь
    println!("{}", s);
}
// v больше недоступен

// ✅ Правильно
for s in &v { // Заимствование
    println!("{}", s);
}
// v все еще доступен
```

### Ошибка 3: Частичный Move структуры
```rust
struct Container {
    name: String,
    value: i32,
}

let c = Container {
    name: String::from("test"),
    value: 42,
};

// ❌ ОШИБКА
let name = c.name; // Partial move
// println!("{:?}", c); // ОШИБКА: c частично moved

// ✅ Решение
let name = c.name.clone(); // Или использовать ссылки
```

## 📝 Упражнения

1. **Реализуйте функцию swap**:
   - Без использования std::mem::swap
   - Обменивайте владение между переменными

2. **Создайте "умный" контейнер**:
   - Который может временно отдавать владение
   - И забирать его обратно

3. **Builder с проверками**:
   - Реализуйте builder, который consume self
   - Добавьте валидацию на этапе build

## 📚 Дополнительные ресурсы

- [Rust Book - Ownership](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [[01_Core/02_Ownership/01_Ownership_Rules|Ownership Rules]]
- [[01_Core/02_Ownership/04_Copy_Clone|Copy and Clone]]
- [[01_Core/03_Borrowing/01_References|Borrowing]]

---
#rust #move #ownership #semantics #memory
