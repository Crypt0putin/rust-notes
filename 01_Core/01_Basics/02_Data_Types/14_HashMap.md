# 🗺️ HashMap - Ассоциативные массивы

## 📋 Обзор HashMap<K, V>

`HashMap<K, V>` - это коллекция пар ключ-значение, использующая хеш-таблицу для быстрого поиска.

```mermaid
graph TD
    A[HashMap Features] --> B[Key-Value Storage]
    A --> C[O(1) Average Access]
    A --> D[Hash-based]
    A --> E[Unordered]
    
    B --> B1[Unique Keys]
    C --> C1[Fast lookups]
    D --> D1[Requires Hash trait]
    E --> E1[No guaranteed order]
```

### Характеристики HashMap
- **Структура**: Хеш-таблица
- **Производительность**: O(1) в среднем для get/insert/remove
- **Порядок**: Не гарантирован
- **Ключи**: Должны реализовывать `Hash` и `Eq`
- **Память**: Больше overhead чем Vec

## 💻 Создание HashMap

### Импорт и создание
```rust
use std::collections::HashMap;

// Пустой HashMap
let mut map1: HashMap<String, i32> = HashMap::new();
let mut map2 = HashMap::<String, i32>::new();

// С предварительным выделением capacity
let mut map3 = HashMap::with_capacity(100);

// Из итератора кортежей
let map4: HashMap<_, _> = vec![
    ("apple", 3),
    ("banana", 2),
    ("orange", 5),
].into_iter().collect();

// Из двух векторов
let keys = vec!["a", "b", "c"];
let values = vec![1, 2, 3];
let map5: HashMap<_, _> = keys.into_iter()
    .zip(values.into_iter())
    .collect();

// С custom hasher
use std::collections::hash_map::RandomState;
let s = RandomState::new();
let mut map6 = HashMap::with_hasher(s);
```

### Макрос для создания (требует внешний крейт)
```rust
// С macro из maplit crate
// use maplit::hashmap;
// let map = hashmap!{
//     "a" => 1,
//     "b" => 2,
//     "c" => 3,
// };

// Собственный макрос
macro_rules! hashmap {
    ($($key:expr => $val:expr),* $(,)?) => {{
        let mut map = HashMap::new();
        $(map.insert($key, $val);)*
        map
    }};
}

let map = hashmap!{
    "one" => 1,
    "two" => 2,
    "three" => 3,
};
```

## 🔧 Основные операции

### Вставка и обновление
```rust
let mut scores = HashMap::new();

// Простая вставка
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 20);

// Вставка с проверкой
scores.entry(String::from("Yellow"))
    .or_insert(30);

// Обновление существующего значения
scores.entry(String::from("Blue"))
    .and_modify(|v| *v += 10)
    .or_insert(5);

// Вставка только если ключа нет
scores.entry(String::from("Green"))
    .or_default();  // Для типов с Default

// Сложная логика с entry API
let text = "hello world wonderful world";
let mut word_count = HashMap::new();

for word in text.split_whitespace() {
    let count = word_count.entry(word).or_insert(0);
    *count += 1;
}

// Bulk insert
scores.extend([
    (String::from("Purple"), 40),
    (String::from("Orange"), 50),
]);
```

### Получение значений
```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 20);

// Получение по ключу
let blue_score = scores.get("Blue");           // Option<&V>
let blue_score = scores.get_mut("Blue");       // Option<&mut V>

// С обработкой Option
match scores.get("Blue") {
    Some(score) => println!("Score: {}", score),
    None => println!("No score found"),
}

// Получение с default
let green_score = scores.get("Green").unwrap_or(&0);

// Получение пары ключ-значение
let pair = scores.get_key_value("Blue");       // Option<(&K, &V)>

// Прямой доступ (может паниковать)
// let score = scores["Blue"];  // Не рекомендуется!
```

### Удаление элементов
```rust
let mut map = HashMap::new();
map.insert("a", 1);
map.insert("b", 2);
map.insert("c", 3);

// Удаление по ключу
let removed = map.remove("b");         // Option<V>
let removed_pair = map.remove_entry("a"); // Option<(K, V)>

// Условное удаление
map.retain(|k, v| *v > 1);            // Оставить только где значение > 1

// Очистка
map.clear();                          // Удалить все элементы
```

## 🔄 Итерация

### Различные способы итерации
```rust
let mut book_reviews = HashMap::new();
book_reviews.insert("Rust Book", "Great");
book_reviews.insert("Design Patterns", "Good");
book_reviews.insert("The Little Prince", "Excellent");

// По парам ключ-значение
for (book, review) in &book_reviews {
    println!("{}: {}", book, review);
}

// Только по ключам
for book in book_reviews.keys() {
    println!("Book: {}", book);
}

// Только по значениям
for review in book_reviews.values() {
    println!("Review: {}", review);
}

// Изменяемая итерация по значениям
for review in book_reviews.values_mut() {
    *review = "Updated";
}

// Потребляющая итерация
for (book, review) in book_reviews {
    println!("{}: {}", book, review);
    // book_reviews больше не доступен
}

// Drain - удаление с возвратом
let drained: HashMap<_, _> = book_reviews.drain().collect();
```

## 🎨 Entry API

Entry API предоставляет эффективный способ работы с записями:

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Stats {
    count: i32,
    sum: i32,
}

impl Stats {
    fn new() -> Self {
        Stats { count: 0, sum: 0 }
    }
    
    fn add(&mut self, value: i32) {
        self.count += 1;
        self.sum += value;
    }
}

let mut stats_map: HashMap<String, Stats> = HashMap::new();
let data = vec![
    ("apple", 5),
    ("banana", 3),
    ("apple", 7),
    ("orange", 2),
    ("banana", 4),
];

for (key, value) in data {
    stats_map.entry(key.to_string())
        .or_insert_with(Stats::new)
        .add(value);
}

// Более сложный пример с match
use std::collections::hash_map::Entry;

match stats_map.entry("grape".to_string()) {
    Entry::Occupied(mut e) => {
        e.get_mut().add(10);
    }
    Entry::Vacant(e) => {
        let mut stats = Stats::new();
        stats.add(10);
        e.insert(stats);
    }
}
```

## 🔑 Требования к ключам

### Hash и Eq traits
```rust
use std::hash::{Hash, Hasher};

#[derive(Debug)]
struct Person {
    id: u32,
    name: String,
}

// Реализация Hash
impl Hash for Person {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.id.hash(state);
        // name не участвует в хешировании
    }
}

// Реализация PartialEq
impl PartialEq for Person {
    fn eq(&self, other: &Self) -> bool {
        self.id == other.id
        // name не участвует в сравнении
    }
}

impl Eq for Person {}

// Теперь Person можно использовать как ключ
let mut map = HashMap::new();
map.insert(Person { id: 1, name: "Alice".to_string() }, 100);
```

### Derive макросы
```rust
// Проще использовать derive
#[derive(Hash, Eq, PartialEq, Debug)]
struct Point {
    x: i32,
    y: i32,
}

let mut positions = HashMap::new();
positions.insert(Point { x: 0, y: 0 }, "origin");
positions.insert(Point { x: 1, y: 1 }, "diagonal");
```

## 📊 Производительность и оптимизация

### Capacity и load factor
```rust
let mut map = HashMap::with_capacity(100);

println!("Capacity: {}", map.capacity());  // >= 100
println!("Length: {}", map.len());         // 0

// HashMap автоматически увеличивается при заполнении
for i in 0..200 {
    map.insert(i, i * 2);
}

// Уменьшение неиспользуемой памяти
map.shrink_to_fit();

// Резервирование дополнительного места
map.reserve(50);  // Для минимум 50 дополнительных элементов
```

### Альтернативные hashers
```rust
use std::collections::HashMap;
use std::hash::BuildHasherDefault;

// FxHash - быстрый hasher для небольших ключей
// use fxhash::FxHasher;
// type FxHashMap<K, V> = HashMap<K, V, BuildHasherDefault<FxHasher>>;

// AHash - современный быстрый hasher
// use ahash::AHashMap;
// let mut map = AHashMap::new();
```

## ⚠️ Частые проблемы

### Проблема 1: Использование не-Hash типов
```rust
// ❌ f32/f64 не реализуют Hash
// let mut map = HashMap::new();
// map.insert(3.14, "pi");  // ОШИБКА

// ✅ Обертка или преобразование
use std::collections::HashMap;
use ordered_float::OrderedFloat;

let mut map = HashMap::new();
map.insert(OrderedFloat(3.14), "pi");

// Или конвертация в строку
let mut map = HashMap::new();
map.insert(3.14.to_string(), "pi");
```

### Проблема 2: Перемещение при вставке
```rust
let mut map = HashMap::new();
let key = String::from("key");
let value = String::from("value");

// ❌ key и value перемещены
map.insert(key, value);
// println!("{}", key);  // ОШИБКА: key moved

// ✅ Клонирование или использование ссылок
let key = String::from("key");
let value = String::from("value");
map.insert(key.clone(), value.clone());
println!("{}: {}", key, value);  // OK
```

### Проблема 3: Изменение ключей
```rust
// ❌ НИКОГДА не изменяйте ключи после вставки
#[derive(Hash, Eq, PartialEq)]
struct MutableKey {
    value: std::cell::RefCell<i32>,
}

// Это приведет к неопределенному поведению!
```

## 🎯 Best Practices

### 1. Используйте entry API для эффективных обновлений
```rust
// Плохо - двойной поиск
if !map.contains_key(&key) {
    map.insert(key, value);
}

// Хорошо - один поиск
map.entry(key).or_insert(value);
```

### 2. Предварительно выделяйте capacity
```rust
// Если знаете примерный размер
let mut map = HashMap::with_capacity(expected_size);
```

### 3. Используйте &str вместо String для поиска
```rust
let mut map = HashMap::new();
map.insert(String::from("key"), "value");

// Можно искать по &str благодаря Borrow trait
let value = map.get("key");  // Не нужно создавать String
```

## 🎯 Флеш-карточки

#flashcard 
Q: Какая средняя сложность операций get/insert/remove в HashMap?
A: O(1) - константная в среднем случае
<!--SR:!2024-03-10,4,270-->

#flashcard 
Q: Какие traits должен реализовывать тип для использования в качестве ключа HashMap?
A: Hash и Eq (полное равенство)
<!--SR:!2024-03-11,5,280-->

#flashcard 
Q: Что делает метод entry() в HashMap?
A: Возвращает Entry для эффективной работы с записью (вставка/обновление)
<!--SR:!2024-03-12,3,250-->

## 🔗 Связанные темы

- [[01_Core/01_Basics/02_Data_Types/15_HashSet|HashSet]] - множества
- [[01_Core/01_Basics/02_Data_Types/13_Vectors|Vectors]] - последовательные коллекции
- [[01_Core/06_Traits/00_Index|Traits]] - Hash и Eq traits
- [[BTreeMap]] - упорядоченная альтернатива

---
#rust #hashmap #collections #hash-table #key-value
