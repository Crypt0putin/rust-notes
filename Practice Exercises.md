# 🏋️ Rust Practice Exercises

## 📊 Уровни сложности

- 🟢 **Beginner** - Базовые концепции
- 🟡 **Intermediate** - Комбинация концепций
- 🔴 **Advanced** - Сложные задачи
- 🟣 **Expert** - Продвинутые техники

---

## 🟢 Beginner Exercises

### Exercise 1: FizzBuzz
**Концепции**: loops, conditions, match
```rust
// Напишите программу, которая выводит числа от 1 до 100
// Для чисел кратных 3 выведите "Fizz"
// Для чисел кратных 5 выведите "Buzz"
// Для чисел кратных и 3 и 5 выведите "FizzBuzz"

fn fizzbuzz(n: u32) -> String {
    // TODO: Implement
    todo!()
}

#[test]
fn test_fizzbuzz() {
    assert_eq!(fizzbuzz(3), "Fizz");
    assert_eq!(fizzbuzz(5), "Buzz");
    assert_eq!(fizzbuzz(15), "FizzBuzz");
    assert_eq!(fizzbuzz(7), "7");
}
```

### Exercise 2: Reverse String
**Концепции**: strings, iterators
```rust
// Разверните строку без использования встроенного reverse()

fn reverse_string(s: &str) -> String {
    // TODO: Implement
    todo!()
}

#[test]
fn test_reverse() {
    assert_eq!(reverse_string("hello"), "olleh");
    assert_eq!(reverse_string("rust"), "tsur");
}
```

### Exercise 3: Find Maximum
**Концепции**: slices, generics, traits
```rust
// Найдите максимальный элемент в срезе

fn find_max<T: PartialOrd>(slice: &[T]) -> Option<&T> {
    // TODO: Implement
    todo!()
}

#[test]
fn test_find_max() {
    assert_eq!(find_max(&[1, 3, 2]), Some(&3));
    assert_eq!(find_max(&["a", "z", "b"]), Some(&"z"));
    assert_eq!(find_max(&[]), None::<&i32>);
}
```

### Exercise 4: Count Words
**Концепции**: strings, hashmaps, iterators
```rust
use std::collections::HashMap;

// Подсчитайте частоту слов в тексте

fn count_words(text: &str) -> HashMap<String, usize> {
    // TODO: Implement
    todo!()
}

#[test]
fn test_count_words() {
    let mut expected = HashMap::new();
    expected.insert("hello".to_string(), 2);
    expected.insert("world".to_string(), 1);
    
    assert_eq!(count_words("hello world hello"), expected);
}
```

---

## 🟡 Intermediate Exercises

### Exercise 5: Custom Iterator
**Концепции**: traits, iterators, generics
```rust
// Создайте итератор для чисел Фибоначчи

struct Fibonacci {
    current: u64,
    next: u64,
}

impl Fibonacci {
    fn new() -> Self {
        // TODO: Implement
        todo!()
    }
}

impl Iterator for Fibonacci {
    type Item = u64;
    
    fn next(&mut self) -> Option<Self::Item> {
        // TODO: Implement
        todo!()
    }
}

#[test]
fn test_fibonacci() {
    let fib = Fibonacci::new();
    let first_10: Vec<u64> = fib.take(10).collect();
    assert_eq!(first_10, vec![0, 1, 1, 2, 3, 5, 8, 13, 21, 34]);
}
```

### Exercise 6: Binary Search Tree
**Концепции**: enums, Box, recursion, generics
```rust
// Реализуйте бинарное дерево поиска

#[derive(Debug)]
enum BST<T: Ord> {
    Empty,
    Node {
        value: T,
        left: Box<BST<T>>,
        right: Box<BST<T>>,
    },
}

impl<T: Ord> BST<T> {
    fn new() -> Self {
        // TODO: Implement
        todo!()
    }
    
    fn insert(&mut self, value: T) {
        // TODO: Implement
        todo!()
    }
    
    fn contains(&self, value: &T) -> bool {
        // TODO: Implement
        todo!()
    }
}

#[test]
fn test_bst() {
    let mut tree = BST::new();
    tree.insert(5);
    tree.insert(3);
    tree.insert(7);
    
    assert!(tree.contains(&5));
    assert!(tree.contains(&3));
    assert!(!tree.contains(&4));
}
```

### Exercise 7: Thread Pool
**Концепции**: threads, channels, Arc, Mutex
```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

// Создайте простой thread pool

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl ThreadPool {
    fn new(size: usize) -> Self {
        // TODO: Implement
        todo!()
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        // TODO: Implement
        todo!()
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        // TODO: Implement graceful shutdown
        todo!()
    }
}
```

### Exercise 8: JSON Parser
**Концепции**: enums, Result, parsing, recursion
```rust
use std::collections::HashMap;

// Простой JSON парсер

#[derive(Debug, PartialEq)]
enum JsonValue {
    Null,
    Bool(bool),
    Number(f64),
    String(String),
    Array(Vec<JsonValue>),
    Object(HashMap<String, JsonValue>),
}

fn parse_json(input: &str) -> Result<JsonValue, String> {
    // TODO: Implement basic JSON parsing
    todo!()
}

#[test]
fn test_json_parser() {
    assert_eq!(parse_json("null"), Ok(JsonValue::Null));
    assert_eq!(parse_json("true"), Ok(JsonValue::Bool(true)));
    assert_eq!(parse_json("42"), Ok(JsonValue::Number(42.0)));
    assert_eq!(parse_json("\"hello\""), Ok(JsonValue::String("hello".to_string())));
}
```

---

## 🔴 Advanced Exercises

### Exercise 9: Async Web Scraper
**Концепции**: async/await, tokio, error handling
```rust
use tokio;
use reqwest;

// Асинхронный веб-скрапер

async fn scrape_urls(urls: Vec<String>) -> Vec<Result<String, String>> {
    // TODO: Fetch all URLs concurrently
    // Return page titles or errors
    todo!()
}

#[tokio::test]
async fn test_scraper() {
    let urls = vec![
        "https://www.rust-lang.org".to_string(),
        "https://docs.rs".to_string(),
    ];
    
    let results = scrape_urls(urls).await;
    assert_eq!(results.len(), 2);
}
```

### Exercise 10: Lock-Free Queue
**Концепции**: unsafe, atomics, concurrency
```rust
use std::sync::atomic::{AtomicPtr, Ordering};
use std::ptr;

// Lock-free односвязная очередь

struct Node<T> {
    data: T,
    next: AtomicPtr<Node<T>>,
}

struct LockFreeQueue<T> {
    head: AtomicPtr<Node<T>>,
    tail: AtomicPtr<Node<T>>,
}

unsafe impl<T: Send> Send for LockFreeQueue<T> {}
unsafe impl<T: Send> Sync for LockFreeQueue<T> {}

impl<T> LockFreeQueue<T> {
    fn new() -> Self {
        // TODO: Implement
        todo!()
    }
    
    fn enqueue(&self, data: T) {
        // TODO: Implement lock-free enqueue
        todo!()
    }
    
    fn dequeue(&self) -> Option<T> {
        // TODO: Implement lock-free dequeue
        todo!()
    }
}
```

### Exercise 11: Custom Derive Macro
**Концепции**: procedural macros, syn, quote
```rust
// Создайте derive макрос для автоматической генерации builder pattern

#[derive(Builder)]
struct User {
    name: String,
    age: u32,
    email: Option<String>,
}

// Должно генерировать:
// impl User {
//     fn builder() -> UserBuilder { ... }
// }
// 
// struct UserBuilder { ... }
// impl UserBuilder {
//     fn name(mut self, name: String) -> Self { ... }
//     fn age(mut self, age: u32) -> Self { ... }
//     fn email(mut self, email: String) -> Self { ... }
//     fn build(self) -> User { ... }
// }
```

---

## 🟣 Expert Exercises

### Exercise 12: Type-Safe State Machine
**Концепции**: phantom types, zero-cost abstractions
```rust
use std::marker::PhantomData;

// Type-safe state machine для TCP соединения

struct Closed;
struct Listen;
struct SynReceived;
struct Established;

struct TcpConnection<State> {
    _state: PhantomData<State>,
}

impl TcpConnection<Closed> {
    fn new() -> Self {
        todo!()
    }
    
    fn listen(self) -> TcpConnection<Listen> {
        todo!()
    }
}

impl TcpConnection<Listen> {
    fn accept(self) -> TcpConnection<SynReceived> {
        todo!()
    }
}

// И так далее...
// Невозможные переходы должны быть ошибками компиляции
```

### Exercise 13: Custom Allocator
**Концепции**: unsafe, memory management, GlobalAlloc
```rust
use std::alloc::{GlobalAlloc, Layout};

// Простой bump allocator

struct BumpAllocator {
    // TODO: Define fields
}

unsafe impl GlobalAlloc for BumpAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        // TODO: Implement
        todo!()
    }
    
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        // TODO: Implement (или оставить пустым для bump allocator)
        todo!()
    }
}
```

---

## 📝 Шаблон для решения

```rust
// Problem: [Название задачи]
// Difficulty: [Уровень]
// Concepts: [Концепции]
// Time limit: [Время]

// Your solution:
fn solution() {
    // TODO: Write your code here
}

// Tests:
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_solution() {
        // TODO: Add test cases
    }
}

// Notes:
// - Какие подходы вы пробовали?
// - Какие были сложности?
// - Что нового узнали?
```

## 🎯 Челленджи

### 30-Day Challenge
- **День 1-10**: По 3 beginner задачи в день
- **День 11-20**: По 2 intermediate задачи в день
- **День 21-30**: По 1 advanced задаче в день

### Weekly Projects
- **Неделя 1**: CLI приложение
- **Неделя 2**: Web сервер
- **Неделя 3**: Async приложение
- **Неделя 4**: Systems programming

## 🔗 Ресурсы для практики

- [[03_Projects/00_Index|Projects]] - полноценные проекты
- [[Code Snippets]] - готовые решения
- [[04_Flashcards/00_Index|Flashcards]] - повторение теории
- [Exercism](https://exercism.io/tracks/rust) - онлайн задачи
- [Rustlings](https://github.com/rust-lang/rustlings) - упражнения

---
#rust #exercises #practice #challenges #coding
