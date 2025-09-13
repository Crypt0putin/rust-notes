# E0382: Использование перемещённого значения

**Категория**: Ownership Error  
**Частота**: ⭐⭐⭐⭐⭐ Очень частая  
**Сложность**: 🟡 Средняя

## 🚫 Описание ошибки

Эта ошибка возникает, когда вы пытаетесь использовать значение после того, как владение им было передано (moved) в другое место.

## 💥 Примеры ошибок

### Базовый случай
```rust
let s1 = String::from("hello");
let s2 = s1; // Владение переходит к s2
println!("{}", s1); // ❌ Error: borrow of moved value: `s1`
```

**Сообщение компилятора:**
```
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:3:20
  |
1 | let s1 = String::from("hello");
  |     -- move occurs because `s1` has type `String`
2 | let s2 = s1;
  |          -- value moved here  
3 | println!("{}", s1);
  |                ^^ value borrowed here after move
```

### Передача в функцию
```rust
fn take_ownership(s: String) {
    println!("{}", s);
} // s выходит из области видимости и удаляется

fn main() {
    let s = String::from("hello");
    take_ownership(s); // s перемещается в функцию
    println!("{}", s); // ❌ Error: использование после move
}
```

### В циклах и итераторах
```rust
let strings = vec![String::from("hello"), String::from("world")];
for s in strings { // strings перемещается в цикл
    println!("{}", s);
}
println!("{:?}", strings); // ❌ Error: использование после move
```

## ✅ Решения

### 1. Клонирование
```rust
let s1 = String::from("hello");
let s2 = s1.clone(); // Создаём копию данных
println!("{} {}", s1, s2); // ✅ OK: s1 всё ещё доступна
```

### 2. Заимствование
```rust
let s1 = String::from("hello");
let s2 = &s1; // Заимствуем, не перемещаем
println!("{} {}", s1, s2); // ✅ OK: оба доступны
```

### 3. Возврат владения из функции
```rust
fn take_and_return(s: String) -> String {
    println!("{}", s);
    s // Возвращаем владение
}

fn main() {
    let s = String::from("hello");
    let s = take_and_return(s); // Получаем владение обратно
    println!("{}", s); // ✅ OK
}
```

### 4. Использование ссылок в функциях
```rust
fn borrow_string(s: &String) {
    println!("{}", s);
}

fn main() {
    let s = String::from("hello");
    borrow_string(&s); // Передаём ссылку
    println!("{}", s); // ✅ OK: s всё ещё доступна
}
```

### 5. Итерирование по ссылкам
```rust
let strings = vec![String::from("hello"), String::from("world")];
for s in &strings { // Итерируем по ссылкам
    println!("{}", s);
}
println!("{:?}", strings); // ✅ OK: strings не перемещён
```

## 🤔 Когда происходит move?

### Move происходит:
- При присваивании: `let b = a;`
- При передаче в функцию: `function(value)`  
- При возврате из функции: `return value;`
- В pattern matching: `match value { ... }`
- При вставке в коллекции: `vec.push(value)`

### Move НЕ происходит для Copy типов:
```rust
let x = 5; // i32 реализует Copy
let y = x; // Копируется, не перемещается
println!("{} {}", x, y); // ✅ OK: x всё ещё доступна
```

**Copy типы:**
- Все целочисленные типы (i32, u64, и т.д.)
- Булев тип (bool)
- Все типы с плавающей точкой (f64, f32)
- Символьный тип (char)
- Кортежи, если все элементы имеют Copy

## 🔍 Диагностика

### Как найти место move:
1. Внимательно читайте сообщение об ошибке
2. Ищите строку с комментарием `value moved here`
3. Проследите использование переменной от объявления до ошибки

### Вопросы для самопроверки:
- Где переменная была перемещена?
- Нужно ли мне значение после перемещения?
- Могу ли я использовать заимствование вместо владения?
- Подходит ли клонирование для этого случая?

## 💡 Лучшие практики

### 1. Предпочитайте заимствование владению
```rust
// Вместо этого:
fn process_string(s: String) -> String {
    // обработка
    s
}

// Лучше так:
fn process_string(s: &str) {
    // обработка
}
```

### 2. Используйте методы, не потребляющие self
```rust
// Потребляет self
let result = vec.into_iter().collect();

// Не потребляет
let result: Vec<_> = vec.iter().cloned().collect();
```

### 3. Структурируйте код для минимизации moves
```rust
// Неэффективно: много клонирований
let data = expensive_computation();
process_data_1(data.clone());
process_data_2(data.clone());
process_data_3(data);

// Лучше: используйте ссылки
let data = expensive_computation();
process_data_1(&data);
process_data_2(&data);  
process_data_3(&data);
```

## 🎯 Практические упражнения

### Упражнение 1: Исправьте код
```rust
fn main() {
    let s = String::from("test");
    let len = calculate_length(s);
    println!("Length of '{}' is {}", s, len); // ❌ Исправьте
}

fn calculate_length(s: String) -> usize {
    s.len()
}
```

### Упражнение 2: Работа с векторами
```rust
fn main() {
    let v = vec![1, 2, 3];
    let first = v[0];
    let v2 = v; // ❌ Что здесь не так?
    println!("First: {}, Vector: {:?}", first, v);
}
```

## 🔗 Связанные ошибки
- [[E0384-Cannot-Assign-Twice]] - Повторное присваивание
- [[E0502-Cannot-Borrow-Mutably]] - Конфликт заимствований
- [[E0597-Borrowed-Value-Does-Not-Live-Long-Enough]] - Время жизни ссылок

## 📚 Дополнительное чтение
- [[02-Ownership/Move-Semantics]] - Семантика перемещения
- [[02-Ownership/Borrowing]] - Заимствование данных
- [[Clone-Copy-Traits]] - Трейты Copy и Clone

#error-E0382 #ownership #move-semantics #common-error #beginner