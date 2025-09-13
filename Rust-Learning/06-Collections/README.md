# 📦 Коллекции в Rust

Стандартные коллекции для хранения и работы с группами данных.

## 📋 Содержание

### Основные коллекции
- [[Vector-Vec]] - Динамические массивы Vec<T>
- [[String-Type]] - Строки и String
- [[HashMap-BTreeMap]] - Ассоциативные массивы
- [[HashSet-BTreeSet]] - Множества

### Специальные коллекции
- [[VecDeque]] - Двусторонняя очередь
- [[LinkedList]] - Связанный список
- [[BinaryHeap]] - Двоичная куча

## 🔢 Векторы (Vec<T>)

### Создание векторов
```rust
// Пустой вектор
let mut v: Vec<i32> = Vec::new();

// С начальными значениями
let v = vec![1, 2, 3, 4, 5];

// С заданной ёмкостью
let mut v = Vec::with_capacity(10);
```

### Операции с векторами
```rust
let mut v = Vec::new();

// Добавление элементов
v.push(5);
v.push(6);
v.push(7);
v.push(8);

// Доступ к элементам
let third: &i32 = &v[2];
match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}

// Итерирование
for i in &v {
    println!("{}", i);
}

// Изменение элементов
for i in &mut v {
    *i += 50;
}
```

## 🔤 Строки (String)

### Создание строк
```rust
// Пустая строка
let mut s = String::new();

// Из строкового литерала
let s = "initial contents".to_string();
let s = String::from("initial contents");

// UTF-8 данные
let hello = String::from("Здравствуй");
```

### Операции со строками
```rust
let mut s = String::from("foo");
s.push_str("bar"); // Добавить строку
s.push('!');       // Добавить символ

// Конкатенация
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 перемещается!

// С format!
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{}-{}-{}", s1, s2, s3);

// Итерирование
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

## 🗺️ HashMap<K, V>

### Создание HashMap
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// Из векторов
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
let scores: HashMap<_, _> = teams.into_iter().zip(initial_scores.into_iter()).collect();
```

### Операции с HashMap
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

// Доступ к значениям
let team_name = String::from("Blue");
let score = scores.get(&team_name);

// Итерирование
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// Обновление значений
scores.insert(String::from("Blue"), 25); // Перезапись

// Вставка только если ключа нет
scores.entry(String::from("Red")).or_insert(50);

// Обновление на основе старого значения
let text = "hello world wonderful world";
let mut map = HashMap::new();
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```

## 🔍 HashSet<T>

### Основные операции
```rust
use std::collections::HashSet;

let mut books = HashSet::new();

// Добавление элементов
books.insert("A Game of Thrones");
books.insert("To Kill a Mockingbird");
books.insert("The Great Gatsby");

// Проверка наличия
if !books.contains("The Winds of Winter") {
    println!("We have {} books, but The Winds of Winter ain't one.",
             books.len());
}

// Удаление
books.remove("The Great Gatsby");

// Итерирование
for book in &books {
    println!("{}", book);
}
```

### Операции над множествами
```rust
use std::collections::HashSet;

let a: HashSet<i32> = vec![1, 2, 3].into_iter().collect();
let b: HashSet<i32> = vec![4, 2, 3, 4].into_iter().collect();

// Пересечение
for x in a.intersection(&b) {
    println!("{}", x); // 2, 3
}

// Разность
for x in a.difference(&b) {
    println!("{}", x); // 1
}

// Объединение
for x in a.union(&b) {
    println!("{}", x); // 1, 2, 3, 4
}

// Симметричная разность
for x in a.symmetric_difference(&b) {
    println!("{}", x); // 1, 4
}
```

## 📊 Практические примеры

### Подсчёт слов
```rust
use std::collections::HashMap;

fn count_words(text: &str) -> HashMap<String, usize> {
    let mut word_count = HashMap::new();
    
    for word in text.split_whitespace() {
        let count = word_count.entry(word.to_lowercase()).or_insert(0);
        *count += 1;
    }
    
    word_count
}

fn main() {
    let text = "hello world hello rust world";
    let counts = count_words(text);
    
    for (word, count) in counts {
        println!("{}: {}", word, count);
    }
}
```

### Группировка данных
```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

fn group_by_city(people: Vec<Person>) -> HashMap<String, Vec<Person>> {
    let mut groups = HashMap::new();
    
    for person in people {
        groups.entry(person.city.clone())
              .or_insert_with(Vec::new)
              .push(person);
    }
    
    groups
}
```

## 💡 Выбор коллекции

### Когда использовать что?

| Коллекция | Использовать когда |
|-----------|-------------------|
| `Vec<T>` | Нужен список элементов с доступом по индексу |
| `HashMap<K, V>` | Нужны пары ключ-значение |
| `HashSet<T>` | Нужна коллекция уникальных элементов |
| `BTreeMap<K, V>` | Нужен отсортированный HashMap |
| `BTreeSet<T>` | Нужен отсортированный HashSet |
| `VecDeque<T>` | Нужно добавлять/удалять с обеих сторон |

## ✅ Чек-лист изучения
- [ ] Создаю и использую векторы
- [ ] Работаю со строками и их методами
- [ ] Использую HashMap для ассоциативных данных
- [ ] Применяю HashSet для уникальных элементов
- [ ] Понимаю владение в коллекциях
- [ ] Выбираю подходящую коллекцию для задачи

#collections #vector #string #hashmap #hashset #data-structures #intermediate