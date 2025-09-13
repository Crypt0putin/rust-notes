# E0502: Нельзя заимствовать как изменяемое

**Категория**: Borrowing Error  
**Частота**: ⭐⭐⭐⭐ Очень частая  
**Сложность**: 🟡 Средняя

## 🚫 Описание ошибки

Эта ошибка возникает, когда вы пытаетесь создать изменяемое заимствование, пока существуют активные неизменяемые заимствования того же значения.

## 💥 Примеры ошибок

### Базовый случай
```rust
let mut s = String::from("hello");
let r1 = &s;        // неизменяемое заимствование
let r2 = &mut s;    // ❌ Error: cannot borrow as mutable
println!("{} {}", r1, r2);
```

**Сообщение компилятора:**
```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:3:14
  |
2 |     let r1 = &s;
  |              -- immutable borrow occurs here
3 |     let r2 = &mut s;
  |              ^^^^^^ mutable borrow occurs here
4 |     println!("{} {}", r1, r2);
  |                       -- immutable borrow later used here
```

### В функциях
```rust
fn process_string(s: &mut String) {
    let immutable_ref = &*s;        // неизменяемое заимствование
    s.push_str(" world");           // ❌ Error: пытаемся изменить через s
    println!("{}", immutable_ref);
}
```

### В циклах
```rust
let mut vec = vec![1, 2, 3, 4, 5];
let first = &vec[0];               // неизменяемое заимствование

for item in &mut vec {             // ❌ Error: изменяемое заимствование
    *item *= 2;
}
println!("First: {}", first);      // неизменяемое заимствование всё ещё используется
```

### С методами
```rust
let mut s = String::from("hello");
let len = s.len();                 // неизменяемое заимствование (&self)
s.push_str(" world");              // ❌ Error: изменяемое заимствование (&mut self)
println!("Length was: {}", len);
```

## ✅ Решения

### 1. Разделение областей видимости
```rust
let mut s = String::from("hello");

// Используем неизменяемые ссылки в ограниченной области
{
    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
} // r1 и r2 выходят из области видимости

// Теперь можно создать изменяемую ссылку
let r3 = &mut s;
r3.push_str(" world");
println!("{}", r3);
```

### 2. Использование Non-Lexical Lifetimes (NLL)
```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2); // Последнее использование r1 и r2

// ✅ OK: r1 и r2 больше не используются
let r3 = &mut s;
r3.push_str(" world");
println!("{}", r3);
```

### 3. Клонирование для независимых копий
```rust
let mut s = String::from("hello");
let r1 = s.clone();             // Создаём копию
let r2 = &mut s;                // ✅ OK: изменяем оригинал
r2.push_str(" world");
println!("Original: {}, Copy: {}", r2, r1);
```

### 4. Реструктуризация кода
```rust
// Вместо:
fn bad_example(s: &mut String) {
    let len = s.len();          // неизменяемое заимствование
    s.push_str(" world");       // ❌ конфликт
    println!("Length was: {}", len);
}

// Лучше:
fn good_example(s: &mut String) {
    let len = s.len();          // получаем длину
    println!("Length was: {}", len); // используем сразу
    s.push_str(" world");       // теперь можно изменять
}
```

### 5. Использование методов, возвращающих owned значения
```rust
let mut vec = vec![1, 2, 3, 4, 5];
let first_copy = vec[0];        // копируем значение, не берём ссылку

for item in &mut vec {          // ✅ OK: нет активных заимствований
    *item *= 2;
}
println!("First was: {}", first_copy);
```

## 🤔 Почему это происходит?

### Правила заимствования Rust:
1. **Либо одна изменяемая ссылка**
2. **Либо любое количество неизменяемых ссылок**  
3. **Но НЕ одновременно**

### Причины этого правила:
- **Предотвращение data races** во время компиляции
- **Защита от инвалидации итераторов** 
- **Гарантия thread safety**

### Примеры проблем без этого правила:
```rust
// Если бы это было возможно:
let mut vec = vec![1, 2, 3];
let first = &vec[0];           // указывает на первый элемент
vec.clear();                   // очищает вектор
println!("{}", first);         // ❌ использование после освобождения!
```

## 🔍 Диагностика

### Как найти проблему:
1. **Найдите неизменяемое заимствование** - строка с `&value`
2. **Найдите изменяемое заимствование** - строка с `&mut value`
3. **Проследите области использования** - где ссылки используются в последний раз

### Полезные вопросы:
- Нужны ли мне обе ссылки одновременно?
- Могу ли я изменить порядок операций?
- Можно ли использовать owned значения вместо ссылок?
- Можно ли разбить функцию на части?

## 💡 Лучшие практики

### 1. Минимизируйте время жизни заимствований
```rust
// Плохо - долгоживущие ссылки:
fn process_data(data: &mut Vec<i32>) {
    let first = &data[0];       // долгоживущая ссылка
    // ... много кода ...
    data.push(42);              // ❌ конфликт
    println!("First: {}", first);
}

// Лучше - короткие ссылки:
fn process_data(data: &mut Vec<i32>) {
    {
        let first = &data[0];   
        println!("First: {}", first); // используем сразу
    }
    data.push(42);              // ✅ OK
}
```

### 2. Предпочитайте owned значения ссылкам когда возможно
```rust
// Вместо хранения ссылок:
struct BadExample<'a> {
    name: &'a str,
    data: &'a mut Vec<i32>,
}

// Лучше owned значения:
struct GoodExample {
    name: String,
    data: Vec<i32>,
}
```

### 3. Разделяйте чтение и запись
```rust
// Вместо смешивания:
fn mixed_operations(data: &mut Vec<i32>) {
    let len = data.len();           // чтение
    data.push(42);                  // запись
    let first = data.first();       // ❌ потенциальный конфликт
}

// Лучше группировать:
fn separated_operations(data: &mut Vec<i32>) -> Option<i32> {
    // Сначала все чтения
    let len = data.len();
    let first = data.first().copied();
    
    // Потом все записи
    data.push(42);
    
    first
}
```

## 🎯 Конкретные сценарии

### Работа с HashMap
```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("key1".to_string(), "value1".to_string());

// ❌ Проблема:
let value = map.get("key1");        // неизменяемое заимствование
map.insert("key2".to_string(), "value2".to_string()); // изменяемое заимствование

// ✅ Решение:
let value = map.get("key1").cloned(); // получаем owned копию
map.insert("key2".to_string(), "value2".to_string()); // теперь OK
```

### Обновление элементов вектора
```rust
let mut numbers = vec![1, 2, 3, 4, 5];

// ❌ Проблема:
let first = &numbers[0];
for num in &mut numbers {
    *num *= 2;
}
println!("First was: {}", first);

// ✅ Решение 1 - копирование:
let first = numbers[0];             // копируем значение
for num in &mut numbers {
    *num *= 2;
}
println!("First was: {}", first);

// ✅ Решение 2 - разделение:
println!("First is: {}", numbers[0]); // используем сразу
for num in &mut numbers {
    *num *= 2;
}
```

## 🧪 Практические упражнения

### Упражнение 1: Исправьте функцию
```rust
fn append_and_return_first(vec: &mut Vec<String>) -> &String {
    let first = &vec[0];        // ❌ Исправьте эту функцию
    vec.push("new".to_string());
    first
}
```

### Упражнение 2: Обработка строк
```rust
fn process_string(s: &mut String) {
    let len = s.len();
    let chars: Vec<char> = s.chars().collect(); // ❌ Исправьте
    s.push_str(&format!(" (length: {})", len));
}
```

## 🔗 Связанные ошибки
- [[E0382-Borrow-Of-Moved-Value]] - Использование после move
- [[E0499-Cannot-Borrow-Mutably-Multiple-Times]] - Множественные изменяемые заимствования
- [[E0597-Borrowed-Value-Does-Not-Live-Long-Enough]] - Недостаточное время жизни

## 📚 Дополнительное чтение
- [[02-Ownership/Borrowing]] - Правила заимствования
- [[02-Ownership/Reference-Rules]] - Детальные правила ссылок
- [[Non-Lexical-Lifetimes]] - NLL и современная проверка времени жизни

#error-E0502 #borrowing #mutable #immutable #common-error #intermediate