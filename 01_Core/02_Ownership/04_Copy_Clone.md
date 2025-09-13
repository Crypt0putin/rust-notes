# 📋 Copy and Clone Traits

## 🎯 Обзор

Copy и Clone - это трейты для копирования данных в Rust. Они определяют, как значения дублируются.

## 📊 Copy Trait

### Что такое Copy?

Copy trait позволяет типу копироваться побитово при присваивании. Это "дешевое" копирование на стеке.

```rust
// Copy типы
let x = 5;
let y = x; // x копируется в y
println!("x = {}, y = {}", x, y); // Оба доступны
```

### Типы, реализующие Copy

```rust
// Все целочисленные типы
let a: i32 = 42;
let b: u64 = 100;

// Числа с плавающей точкой
let f1: f32 = 3.14;
let f2: f64 = 2.718;

// Булевы значения
let t = true;
let f = false;

// Символы
let c = 'z';

// Кортежи из Copy типов
let point = (3, 5);
let copy = point; // Copy

// Массивы из Copy типов
let arr = [1, 2, 3, 4];
let arr_copy = arr; // Copy
```

### Реализация Copy для своих типов

```rust
#[derive(Debug, Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Self {
        Point { x, y }
    }
}

let p1 = Point::new(10, 20);
let p2 = p1; // Copy
println!("p1: {:?}, p2: {:?}", p1, p2); // Оба доступны
```

### Ограничения Copy

```rust
// ❌ Нельзя реализовать Copy для типов с non-Copy полями
struct Container {
    data: String, // String не Copy
}

// impl Copy for Container {} // ОШИБКА!

// ❌ Copy и Drop несовместимы
struct Resource {
    handle: i32,
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Cleaning up resource");
    }
}

// impl Copy for Resource {} // ОШИБКА!
```

## 🔄 Clone Trait

### Что такое Clone?

Clone trait для явного глубокого копирования. Может быть "дорогой" операцией.

```rust
pub trait Clone {
    fn clone(&self) -> Self;
    
    fn clone_from(&mut self, source: &Self) {
        *self = source.clone()
    }
}
```

### Использование Clone

```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // Глубокое копирование
println!("s1 = {}, s2 = {}", s1, s2); // Оба доступны

let v1 = vec![1, 2, 3];
let v2 = v1.clone(); // Копирует весь вектор
```

### Реализация Clone

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    friends: Vec<String>,
}

// Автоматическая реализация
#[derive(Clone)]
struct AutoPerson {
    name: String,
    age: u32,
}

// Ручная реализация
impl Clone for Person {
    fn clone(&self) -> Self {
        Person {
            name: self.name.clone(),
            age: self.age, // u32 is Copy
            friends: self.friends.clone(),
        }
    }
}

// Оптимизированная реализация
impl Clone for Person {
    fn clone(&self) -> Self {
        Person {
            name: self.name.clone(),
            age: self.age,
            friends: Vec::with_capacity(self.friends.len())
                .extend_from_slice(&self.friends),
        }
    }
}
```

## 🆚 Copy vs Clone

| Аспект | Copy | Clone |
|--------|------|-------|
| Скорость | Быстро (побитовое) | Может быть медленно |
| Память | Только стек | Может выделять на куче |
| Вызов | Неявный | Явный `.clone()` |
| Реализация | Маркерный трейт | Требует метод `clone()` |
| Ограничения | Только для простых типов | Для любых типов |

## 💻 Практические примеры

### Пример 1: Оптимизация с Clone
```rust
use std::rc::Rc;

#[derive(Clone)]
struct ExpensiveData {
    data: Vec<u8>,
}

// Дорогое клонирование
fn expensive_clone(data: &ExpensiveData) -> ExpensiveData {
    data.clone() // Копирует весь вектор
}

// Дешевое "клонирование" с Rc
fn cheap_clone(data: &Rc<ExpensiveData>) -> Rc<ExpensiveData> {
    Rc::clone(data) // Только увеличивает счетчик ссылок
}
```

### Пример 2: Селективное клонирование
```rust
#[derive(Debug)]
struct Config {
    name: String,
    settings: HashMap<String, String>,
    cache: Option<Vec<u8>>, // Может быть большим
}

impl Config {
    // Клонирует только важные части
    fn light_clone(&self) -> Self {
        Config {
            name: self.name.clone(),
            settings: self.settings.clone(),
            cache: None, // Не клонируем кэш
        }
    }
}
```

### Пример 3: Clone with Cow
```rust
use std::borrow::Cow;

#[derive(Clone)]
struct Document {
    title: String,
    content: Cow<'static, str>,
}

impl Document {
    fn new(title: String) -> Self {
        Document {
            title,
            content: Cow::Borrowed("Default content"),
        }
    }
    
    fn set_content(&mut self, content: String) {
        self.content = Cow::Owned(content);
    }
}
```

## 🎯 Паттерны использования

### ToOwned для обобщения
```rust
fn process<T: ToOwned>(data: &T) -> T::Owned {
    data.to_owned()
}

let s = "hello";
let owned = process(&s); // String

let v = [1, 2, 3];
let owned_vec = process(&v[..]); // Vec<i32>
```

### Clone on Write (CoW)
```rust
use std::borrow::Cow;

fn process_string(s: Cow<str>) -> Cow<str> {
    if s.contains("replace") {
        let mut owned = s.into_owned();
        owned = owned.replace("replace", "with");
        Cow::Owned(owned)
    } else {
        s // Не клонируем если не нужно
    }
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: В чем главное отличие Copy от Clone?
A: Copy - неявное побитовое копирование на стеке, Clone - явное глубокое копирование
<!--SR:!2024-02-03,4,270-->

#flashcard 
Q: Какие типы могут реализовать Copy?
A: Только типы, все поля которых также Copy, и которые не реализуют Drop
<!--SR:!2024-02-04,5,280-->

#flashcard 
Q: Почему String не может быть Copy?
A: String владеет данными на куче, копирование указателя привело бы к двойному освобождению
<!--SR:!2024-02-05,3,250-->

## ⚠️ Частые ошибки

### Ошибка 1: Попытка Copy для сложных типов
```rust
// ❌ ОШИБКА
#[derive(Copy, Clone)]
struct MyStruct {
    data: Vec<i32>, // Vec не Copy!
}

// ✅ Правильно: только Clone
#[derive(Clone)]
struct MyStruct {
    data: Vec<i32>,
}
```

### Ошибка 2: Забытый clone()
```rust
let s1 = String::from("hello");
// ❌ ОШИБКА
let s2 = s1; // Move, не Copy
// println!("{}", s1); // Ошибка!

// ✅ Правильно
let s2 = s1.clone();
println!("{}", s1); // OK
```

### Ошибка 3: Ненужное клонирование
```rust
// ❌ Неэффективно
fn process(s: String) { // Принимает владение
    println!("{}", s);
}

let my_string = String::from("hello");
process(my_string.clone()); // Ненужный clone
println!("{}", my_string);

// ✅ Эффективно
fn process(s: &str) { // Принимает ссылку
    println!("{}", s);
}

process(&my_string); // Без клонирования
```

## 📝 Упражнения

1. **Реализуйте Copy и Clone**:
   - Создайте структуру Point3D
   - Реализуйте оба трейта
   - Проверьте поведение

2. **Оптимизация клонирования**:
   - Создайте структуру с дорогими полями
   - Реализуйте "умное" клонирование
   - Используйте Rc для оптимизации

3. **Clone vs ToOwned**:
   - Исследуйте разницу
   - Напишите обобщенные функции
   - Используйте с разными типами

## 📚 Дополнительные ресурсы

- [Rust Book - Copy](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [Rust Book - Clone](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [[01_Core/02_Ownership/03_Move_Semantics|Move Semantics]]
- [[02_Advanced/03_Smart_Pointers/06_Cow|Clone on Write]]

---
#rust #copy #clone #traits #ownership
