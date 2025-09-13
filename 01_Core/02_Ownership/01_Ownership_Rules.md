# 🔑 Ownership Rules

## 📋 Основные правила владения

### Правило 1: Каждое значение имеет владельца
```rust
let x = 5; // x владеет значением 5
let s = String::from("hello"); // s владеет строкой
```

### Правило 2: У значения может быть только один владелец
```rust
let s1 = String::from("hello");
let s2 = s1; // s1 больше не валидна! Владение перешло к s2
// println!("{}", s1); // Ошибка компиляции!
```

### Правило 3: Когда владелец выходит из области видимости, значение удаляется
```rust
{
    let s = String::from("hello"); // s валидна с этого момента
    // работаем с s
} // здесь s выходит из области видимости и память освобождается
```

## 🔗 Связанные концепции

- [[01_Core/02_Ownership/02_Memory_Stack_Heap|Stack vs Heap]] - где хранятся данные
- [[01_Core/02_Ownership/03_Move_Semantics|Move семантика]] - как передается владение
- [[01_Core/03_Borrowing/01_References|Заимствование]] - использование без владения
- [[01_Core/02_Ownership/04_Copy_Clone|Copy и Clone]] - копирование данных

## 💡 Важные моменты

### Drop trait
Когда переменная выходит из области видимости, Rust автоматически вызывает функцию `drop`:

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}
```

### Владение и функции
```rust
fn takes_ownership(some_string: String) { // some_string входит в область видимости
    println!("{}", some_string);
} // Здесь some_string выходит из области видимости и `drop` вызывается

fn makes_copy(some_integer: i32) { // some_integer входит в область видимости
    println!("{}", some_integer);
} // Здесь some_integer выходит из области видимости. Ничего особенного не происходит
```

## 🎯 Флеш-карточки

#flashcard 
Q: Что происходит, когда владелец выходит из области видимости?
A: Rust автоматически вызывает функцию `drop`, и память освобождается
<!--SR:!2024-01-15,3,250-->

#flashcard 
Q: Сколько владельцев может быть у значения в Rust?
A: Только один владелец в каждый момент времени
<!--SR:!2024-01-18,4,270-->

## 📝 Практические примеры

### Пример 1: Передача владения
```rust
fn main() {
    let s = String::from("hello");  // s приходит в область видимости
    
    takes_ownership(s);             // значение s перемещается в функцию
                                    // и больше не валидно здесь
    
    let x = 5;                      // x приходит в область видимости
    
    makes_copy(x);                  // x перемещается в функцию,
                                    // но i32 - Copy, так что можно
                                    // использовать x после
}
```

### Пример 2: Возврат владения
```rust
fn gives_ownership() -> String {
    let some_string = String::from("hello");
    some_string // владение передается вызывающей функции
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string // владение возвращается
}
```

## ⚠️ Частые ошибки

### Ошибка: использование после перемещения
```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{}, world!", s1); // ОШИБКА! s1 больше не валидна
```

**Решение**: использовать клонирование или заимствование
```rust
// Вариант 1: Клонирование
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2); // Работает!

// Вариант 2: Заимствование
let s1 = String::from("hello");
let s2 = &s1;
println!("s1 = {}, s2 = {}", s1, s2); // Работает!
```

## 📚 Дополнительные ресурсы

- [The Rust Book - Chapter 4](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [[Rust Memory Model]] - детальное объяснение модели памяти
- [[Common Ownership Patterns]] - паттерны работы с владением

---
#rust #ownership #core #memory
