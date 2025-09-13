# 📌 References and Borrowing

## 🎯 Что такое заимствование?

**Заимствование** (Borrowing) - это механизм, который позволяет использовать значение без передачи владения.

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // &s1 - заимствование
    println!("The length of '{}' is {}.", s1, len); // s1 все еще валидна!
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // s выходит из области видимости, но не владеет значением, поэтому ничего не происходит
```

## 📊 Правила заимствования

### Правило 1: Либо одна изменяемая ссылка, либо любое количество неизменяемых

```rust
let mut s = String::from("hello");

// ✅ Несколько неизменяемых ссылок - ОК
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2);

// ❌ Изменяемая и неизменяемая одновременно - ОШИБКА
let r3 = &s; // неизменяемая
let r4 = &mut s; // ОШИБКА: нельзя заимствовать как изменяемое
```

### Правило 2: Ссылки должны быть всегда валидными

```rust
// ❌ Висячая ссылка - ОШИБКА
fn dangle() -> &String {
    let s = String::from("hello");
    &s // ОШИБКА: s будет удалена, ссылка станет невалидной
}

// ✅ Правильный вариант
fn no_dangle() -> String {
    let s = String::from("hello");
    s // передаем владение
}
```

## 🔗 Связи с другими концепциями

```mermaid
graph LR
    A[Borrowing] --> B[[[01_Core/02_Ownership/01_Ownership_Rules|Ownership]]]
    A --> C[[[01_Core/04_Lifetimes/01_Lifetime_Basics|Lifetimes]]]
    A --> D[[[01_Core/03_Borrowing/02_Mutable_References|Mutable Refs]]]
    A --> E[[[01_Core/03_Borrowing/03_Slices|Slices]]]
    B --> F[[[01_Core/02_Ownership/03_Move_Semantics|Move]]]
    C --> G[[[01_Core/04_Lifetimes/02_Lifetime_Annotations|Annotations]]]
```

## 💻 Примеры кода

### Неизменяемые ссылки
```rust
fn main() {
    let s = String::from("hello");
    
    // Создаем несколько неизменяемых ссылок
    let r1 = &s;
    let r2 = &s;
    let r3 = &s;
    
    println!("r1: {}, r2: {}, r3: {}", r1, r2, r3);
}
```

### Изменяемые ссылки
```rust
fn main() {
    let mut s = String::from("hello");
    
    change(&mut s);
    
    println!("{}", s); // Выведет "hello, world"
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### Область видимости ссылок
```rust
fn main() {
    let mut s = String::from("hello");
    
    {
        let r1 = &mut s;
        r1.push_str(", world");
    } // r1 выходит из области видимости
    
    let r2 = &mut s; // Теперь можно создать новую изменяемую ссылку
    r2.push_str("!");
    
    println!("{}", s); // "hello, world!"
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: Сколько изменяемых ссылок может быть на одно значение одновременно?
A: Только одна изменяемая ссылка в одной области видимости
<!--SR:!2024-01-20,5,280-->

#flashcard 
Q: Можно ли иметь неизменяемую и изменяемую ссылку одновременно?
A: Нет, нельзя иметь изменяемую ссылку, пока существуют неизменяемые
<!--SR:!2024-01-22,4,265-->

#flashcard 
Q: Что такое заимствование в Rust?
A: Механизм использования значения через ссылку без передачи владения
<!--SR:!2024-01-25,6,290-->

## ⚠️ Частые ошибки и решения

### Ошибка 1: Множественные изменяемые ссылки
```rust
// ❌ ОШИБКА
let mut s = String::from("hello");
let r1 = &mut s;
let r2 = &mut s; // ОШИБКА: вторая изменяемая ссылка
println!("{}, {}", r1, r2);
```

**Решение**: Использовать области видимости
```rust
// ✅ Правильно
let mut s = String::from("hello");
{
    let r1 = &mut s;
    r1.push_str(", world");
} // r1 выходит из области видимости
let r2 = &mut s;
r2.push_str("!");
```

### Ошибка 2: Смешивание изменяемых и неизменяемых ссылок
```rust
// ❌ ОШИБКА
let mut s = String::from("hello");
let r1 = &s;     // неизменяемая
let r2 = &s;     // неизменяемая
let r3 = &mut s; // ОШИБКА: изменяемая при наличии неизменяемых
println!("{}, {}, and {}", r1, r2, r3);
```

**Решение**: Завершить использование неизменяемых ссылок
```rust
// ✅ Правильно
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2);
// r1 и r2 больше не используются

let r3 = &mut s;
r3.push_str(", world");
```

## 📝 Практические задания

1. **Задание 1**: Исправьте код
```rust
fn main() {
    let s = String::from("hello");
    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

2. **Задание 2**: Реализуйте функцию, которая принимает две ссылки и возвращает более длинную

3. **Задание 3**: Создайте структуру с методом, который заимствует `self` неизменяемо и изменяемо

## 📚 Полезные ссылки

- [[01_Core/02_Ownership/01_Ownership_Rules|Ownership Rules]] - основы владения
- [[01_Core/03_Borrowing/02_Mutable_References|Mutable References]] - детали изменяемых ссылок
- [[01_Core/04_Lifetimes/01_Lifetime_Basics|Lifetimes]] - время жизни ссылок
- [[Common Borrowing Patterns]] - паттерны заимствования

---
#rust #borrowing #references #core
