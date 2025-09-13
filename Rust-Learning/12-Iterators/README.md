# 🔄 Итераторы в Rust

Итераторы - мощный и идиоматичный способ обработки коллекций данных в Rust. Они ленивые, эффективные и функциональные.

## 📋 Содержание

### Основы итераторов
- [[Iterator-Trait]] - Трейт Iterator и его методы
- [[Iterator-Creation]] - Создание итераторов
- [[Iterator-Consumption]] - Потребляющие методы
- [[Iterator-Adaptation]] - Адаптирующие методы

### Продвинутые техники
- [[Custom-Iterators]] - Создание собственных итераторов
- [[Iterator-Performance]] - Производительность итераторов
- [[Parallel-Iterators]] - Параллельные итераторы с rayon

## 🎯 Основные концепции

### Создание итераторов
```rust
let vec = vec![1, 2, 3, 4, 5];

// Три способа получить итератор:
let iter1 = vec.iter();        // Итератор по ссылкам &T
let iter2 = vec.into_iter();   // Итератор по значениям T (потребляет вектор)
let iter3 = vec.iter_mut();    // Итератор по изменяемым ссылкам &mut T (только для mut vec)
```

### Ленивость итераторов
```rust
let vec = vec![1, 2, 3, 4, 5];
let iter = vec.iter().map(|x| x * 2); // Ничего не вычисляется!

// Итератор начинает работу только при потреблении:
let doubled: Vec<i32> = iter.collect(); // Теперь вычисляется
```

## 🔧 Методы итераторов

### Адаптеры (возвращают новый итератор)

#### `map` - преобразование элементов
```rust
let numbers = vec![1, 2, 3, 4];
let doubled: Vec<i32> = numbers
    .iter()
    .map(|x| x * 2)
    .collect();
// [2, 4, 6, 8]
```

#### `filter` - фильтрация элементов
```rust
let numbers = vec![1, 2, 3, 4, 5, 6];
let even: Vec<&i32> = numbers
    .iter()
    .filter(|&x| x % 2 == 0)
    .collect();
// [2, 4, 6]
```

#### `enumerate` - добавление индексов
```rust
let names = vec!["Alice", "Bob", "Charlie"];
let indexed: Vec<(usize, &str)> = names
    .iter()
    .enumerate()
    .collect();
// [(0, "Alice"), (1, "Bob"), (2, "Charlie")]
```

#### `zip` - объединение итераторов
```rust
let names = vec!["Alice", "Bob", "Charlie"];
let ages = vec![30, 25, 35];
let people: Vec<(&str, &i32)> = names
    .iter()
    .zip(ages.iter())
    .collect();
// [("Alice", 30), ("Bob", 25), ("Charlie", 35)]
```

#### `take` и `skip` - взятие и пропуск элементов
```rust
let numbers: Vec<i32> = (0..10)
    .skip(3)        // Пропускаем первые 3
    .take(4)        // Берём следующие 4
    .collect();
// [3, 4, 5, 6]
```

#### `chain` - соединение итераторов
```rust
let first = vec![1, 2, 3];
let second = vec![4, 5, 6];
let combined: Vec<i32> = first
    .iter()
    .chain(second.iter())
    .cloned()
    .collect();
// [1, 2, 3, 4, 5, 6]
```

### Потребители (возвращают значения)

#### `collect` - сбор в коллекцию
```rust
let numbers: Vec<i32> = (0..5).collect(); // [0, 1, 2, 3, 4]
let map: std::collections::HashMap<i32, String> = (0..3)
    .map(|i| (i, format!("value{}", i)))
    .collect();
```

#### `reduce` - свёртка с явным начальным значением
```rust
let numbers = vec![1, 2, 3, 4, 5];
let sum = numbers.iter().fold(0, |acc, x| acc + x); // 15

// Или с reduce (без начального значения):
let product = numbers.iter().reduce(|acc, x| acc * x); // Some(120)
```

#### `find` - поиск элемента
```rust
let numbers = vec![1, 2, 3, 4, 5];
let found = numbers.iter().find(|&&x| x > 3); // Some(&4)
let not_found = numbers.iter().find(|&&x| x > 10); // None
```

#### `any` и `all` - проверки условий
```rust
let numbers = vec![2, 4, 6, 8];
let all_even = numbers.iter().all(|&x| x % 2 == 0); // true
let has_big = numbers.iter().any(|&x| x > 5); // true
```

#### `count` - подсчёт элементов
```rust
let numbers = vec![1, 2, 3, 4, 5];
let count_big = numbers.iter().filter(|&&x| x > 3).count(); // 2
```

## 🏗️ Цепочки методов

### Классический пример: обработка данных
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    salary: u32,
}

fn main() {
    let people = vec![
        Person { name: "Alice".to_string(), age: 30, salary: 70000 },
        Person { name: "Bob".to_string(), age: 25, salary: 50000 },
        Person { name: "Charlie".to_string(), age: 35, salary: 80000 },
        Person { name: "Diana".to_string(), age: 28, salary: 65000 },
    ];

    // Найти средний возраст людей с зарплатой выше 60000
    let avg_age = people
        .iter()
        .filter(|person| person.salary > 60000)
        .map(|person| person.age)
        .sum::<u32>() as f64
        / people.iter().filter(|person| person.salary > 60000).count() as f64;
    
    println!("Средний возраст: {:.1}", avg_age); // 31.0
}
```

### Оптимизированная версия
```rust
fn main() {
    let people = vec![
        Person { name: "Alice".to_string(), age: 30, salary: 70000 },
        Person { name: "Bob".to_string(), age: 25, salary: 50000 },
        Person { name: "Charlie".to_string(), age: 35, salary: 80000 },
        Person { name: "Diana".to_string(), age: 28, salary: 65000 },
    ];

    // Более эффективный способ - один проход
    let (total_age, count) = people
        .iter()
        .filter(|person| person.salary > 60000)
        .map(|person| person.age)
        .fold((0, 0), |(sum, count), age| (sum + age, count + 1));
    
    if count > 0 {
        let avg_age = total_age as f64 / count as f64;
        println!("Средний возраст: {:.1}", avg_age);
    }
}
```

## 🎯 Продвинутые техники

### Собственный итератор
```rust
struct Counter {
    current: usize,
    max: usize,
}

impl Counter {
    fn new(max: usize) -> Counter {
        Counter { current: 0, max }
    }
}

impl Iterator for Counter {
    type Item = usize;

    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}

// Использование:
let counter = Counter::new(3);
let numbers: Vec<usize> = counter.collect(); // [0, 1, 2]
```

### Итератор по диапазонам
```rust
// Встроенные итераторы диапазонов
let range1: Vec<i32> = (0..5).collect();        // [0, 1, 2, 3, 4]
let range2: Vec<i32> = (0..=5).collect();       // [0, 1, 2, 3, 4, 5]
let range3: Vec<i32> = (0..10).step_by(2).collect(); // [0, 2, 4, 6, 8]
```

### Работа с `Result` в итераторах
```rust
fn parse_numbers(strings: Vec<&str>) -> Result<Vec<i32>, std::num::ParseIntError> {
    strings.iter().map(|s| s.parse::<i32>()).collect()
}

// Или с фильтрацией ошибок:
fn parse_numbers_safe(strings: Vec<&str>) -> Vec<i32> {
    strings
        .iter()
        .filter_map(|s| s.parse().ok()) // Игнорируем ошибки
        .collect()
}

// Использование:
let strings = vec!["1", "2", "not_a_number", "4"];
let numbers = parse_numbers_safe(strings); // [1, 2, 4]
```

## ⚡ Производительность

### Итераторы vs циклы
```rust
// Традиционный цикл
let mut sum = 0;
for i in 0..1_000_000 {
    sum += i;
}

// Итератор (часто быстрее благодаря оптимизациям!)
let sum: usize = (0..1_000_000).sum();
```

### Zero-cost abstractions
```rust
// Этот код:
let vec = vec![1, 2, 3, 4, 5];
let sum: i32 = vec.iter().map(|x| x * 2).sum();

// Компилируется примерно в то же самое, что и:
let vec = vec![1, 2, 3, 4, 5];
let mut sum = 0;
for x in &vec {
    sum += x * 2;
}
```

## 🚀 Параллельные итераторы с Rayon

```rust
// Добавьте в Cargo.toml: rayon = "1.7"
use rayon::prelude::*;

fn main() {
    let numbers: Vec<i32> = (0..1_000_000).collect();
    
    // Последовательная обработка
    let sum: i32 = numbers.iter().map(|x| x * x).sum();
    
    // Параллельная обработка (автоматическое распараллеливание!)
    let par_sum: i32 = numbers.par_iter().map(|x| x * x).sum();
    
    println!("Sequential: {}, Parallel: {}", sum, par_sum);
}
```

## 💡 Лучшие практики

### 1. Предпочитайте итераторы циклам
```rust
// Вместо:
let mut results = Vec::new();
for item in &items {
    if item.is_valid() {
        results.push(item.process());
    }
}

// Лучше:
let results: Vec<_> = items
    .iter()
    .filter(|item| item.is_valid())
    .map(|item| item.process())
    .collect();
```

### 2. Используйте `collect()` осознанно
```rust
// Неэффективно - лишние аллокации:
let vec1: Vec<_> = items.iter().map(|x| x * 2).collect();
let vec2: Vec<_> = vec1.iter().filter(|&x| x > &10).collect();

// Лучше - одна цепочка:
let result: Vec<_> = items
    .iter()
    .map(|x| x * 2)
    .filter(|&x| x > 10)
    .collect();
```

### 3. Избегайте ненужного клонирования
```rust
// Неэффективно:
let strings = vec!["hello".to_string(), "world".to_string()];
let lengths: Vec<usize> = strings.iter().cloned().map(|s| s.len()).collect();

// Лучше:
let lengths: Vec<usize> = strings.iter().map(|s| s.len()).collect();
```

## 🧪 Практические примеры

### Группировка данных
```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Sale {
    product: String,
    amount: f64,
}

fn group_sales_by_product(sales: Vec<Sale>) -> HashMap<String, f64> {
    sales
        .into_iter()
        .fold(HashMap::new(), |mut acc, sale| {
            *acc.entry(sale.product).or_insert(0.0) += sale.amount;
            acc
        })
}
```

### Поиск дубликатов
```rust
use std::collections::HashSet;

fn find_duplicates<T: Clone + Eq + std::hash::Hash>(items: &[T]) -> Vec<T> {
    let mut seen = HashSet::new();
    let mut duplicates = HashSet::new();
    
    for item in items {
        if !seen.insert(item.clone()) {
            duplicates.insert(item.clone());
        }
    }
    
    duplicates.into_iter().collect()
}
```

### Обработка вложенных итераторов
```rust
let nested = vec![vec![1, 2], vec![3, 4, 5], vec![6]];

// flatten - разворачивание вложенных итераторов
let flat: Vec<i32> = nested.iter().flatten().cloned().collect();
// [1, 2, 3, 4, 5, 6]

// flat_map - map + flatten
let doubled_flat: Vec<i32> = nested
    .iter()
    .flat_map(|inner| inner.iter().map(|x| x * 2))
    .collect();
// [2, 4, 6, 8, 10, 12]
```

## 🎯 Упражнения для практики

### Начальный уровень
1. Создайте итератор для чисел Фибоначчи
2. Найдите все слова в тексте длиннее 5 символов
3. Посчитайте сумму квадратов чётных чисел от 1 до 100

### Средний уровень
1. Реализуйте функцию для группировки элементов по ключу
2. Создайте итератор, который возвращает скользящее окно размера N
3. Найдите топ-10 самых частых слов в тексте

### Продвинутый уровень
1. Реализуйте свой аналог `collect()` для пользовательских типов
2. Создайте итератор для обхода дерева в глубину
3. Реализуйте ленивый итератор для чтения больших файлов по строкам

## 🔗 Связанные темы
- [[11-Closures/README]] - Замыкания в итераторах
- [[06-Collections/README]] - Коллекции и итераторы
- [[Performance-Optimization]] - Оптимизация с итераторами

#iterators #functional-programming #collections #performance #intermediate