# 📝 Variables and Mutability

## 🎯 Основные концепции

### Неизменяемые переменные (по умолчанию)
```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    // x = 6; // ОШИБКА: cannot assign twice to immutable variable
}
```

### Изменяемые переменные
```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6; // OK
    println!("The value of x is: {}", x);
}
```

## 🔄 Shadowing (Затенение)

Shadowing позволяет объявить новую переменную с тем же именем:

```rust
fn main() {
    let x = 5;
    let x = x + 1; // Новая переменная x = 6
    
    {
        let x = x * 2; // Новая x в этой области = 12
        println!("Inner scope x: {}", x);
    }
    
    println!("Outer scope x: {}", x); // 6
}
```

### Преимущества shadowing над mut:
1. **Можно изменить тип**:
```rust
let spaces = "   ";
let spaces = spaces.len(); // Теперь это число
```

2. **Сохраняем неизменяемость**:
```rust
let x = 5;
let x = x + 1; // x все еще неизменяемая
```

## 📊 Константы

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
const MAX_POINTS: u32 = 100_000;
```

### Правила для констант:
- Всегда неизменяемые
- Тип должен быть указан явно
- Могут быть объявлены в любой области
- Вычисляются во время компиляции

## 🎭 Static переменные

```rust
static HELLO_WORLD: &str = "Hello, world!";
static mut COUNTER: u32 = 0;

fn add_to_counter(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}
```

### Static vs Const:
| Static | Const |
|--------|-------|
| Фиксированный адрес в памяти | Инлайнится при использовании |
| Может быть mut (unsafe) | Всегда неизменяемая |
| Единственный экземпляр | Копируется при использовании |

## 🔗 Связанные концепции

- [[01_Core/01_Basics/02_Data_Types|Data Types]] - типы переменных
- [[01_Core/02_Ownership/01_Ownership_Rules|Ownership]] - владение переменными
- [[01_Core/01_Basics/03_Functions|Functions]] - переменные в функциях

## 💻 Практические примеры

### Пример 1: Вычисления с переменными
```rust
fn calculate_area() {
    let width = 5;
    let height = 10;
    let area = width * height;
    
    println!("Area of {}x{} rectangle is {}", width, height, area);
}
```

### Пример 2: Изменяемый счетчик
```rust
fn count_to_ten() {
    let mut counter = 0;
    
    while counter < 10 {
        counter += 1;
        println!("Count: {}", counter);
    }
}
```

### Пример 3: Использование констант
```rust
const PI: f64 = 3.141592653589793;

fn circle_area(radius: f64) -> f64 {
    PI * radius * radius
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: В чем разница между let и let mut?
A: let создает неизменяемую переменную, let mut - изменяемую
<!--SR:!2024-01-15,3,250-->

#flashcard 
Q: Что такое shadowing в Rust?
A: Возможность объявить новую переменную с тем же именем, "затеняя" предыдущую
<!--SR:!2024-01-16,4,270-->

#flashcard 
Q: В чем разница между const и static?
A: const инлайнится при использовании, static имеет фиксированный адрес в памяти
<!--SR:!2024-01-17,3,250-->

## ⚠️ Частые ошибки

### Ошибка 1: Попытка изменить неизменяемую переменную
```rust
// ❌ ОШИБКА
let x = 5;
x = 10; // cannot assign twice to immutable variable

// ✅ Решение 1: Использовать mut
let mut x = 5;
x = 10;

// ✅ Решение 2: Использовать shadowing
let x = 5;
let x = 10;
```

### Ошибка 2: Неправильное использование констант
```rust
// ❌ ОШИБКА
const RESULT = calculate_value(); // не может вызывать функции

// ✅ Правильно
const RESULT: i32 = 5 * 10; // только константные выражения
```

## 📝 Упражнения

1. **Создайте программу с различными типами переменных**:
   - Неизменяемая переменная
   - Изменяемая переменная
   - Константа
   - Static переменная

2. **Используйте shadowing для преобразования типов**:
   - Преобразуйте строку в число
   - Измените тип переменной

3. **Напишите функцию-счетчик**:
   - Используйте изменяемую переменную
   - Добавьте константу для максимального значения

## 📚 Дополнительные ресурсы

- [Rust Book - Variables](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html)
- [[01_Core/01_Basics/00_Index|Basics Index]]
- [[Rust Cheatsheet|Cheatsheet]]

---
#rust #variables #mutability #basics #core
