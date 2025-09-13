# 📊 Data Types in Rust

## 🎯 Обзор системы типов

Rust - статически типизированный язык. Компилятор должен знать типы всех переменных во время компиляции.

```rust
let guess: u32 = "42".parse().expect("Not a number!");
//        ^^^^ аннотация типа необходима
```

## 🔢 Скалярные типы

### Целочисленные типы (Integers)

| Размер | Знаковый | Беззнаковый |
|--------|----------|-------------|
| 8-bit  | i8       | u8          |
| 16-bit | i16      | u16         |
| 32-bit | i32      | u32         |
| 64-bit | i64      | u64         |
| 128-bit| i128     | u128        |
| arch   | isize    | usize       |

```rust
// Различные способы записи чисел
let decimal = 98_222;        // Десятичное
let hex = 0xff;             // Шестнадцатеричное
let octal = 0o77;           // Восьмеричное
let binary = 0b1111_0000;   // Двоичное
let byte = b'A';            // Байт (только u8)

// Суффиксы типов
let x = 5u32;               // u32
let y = 10_i64;             // i64
```

### Числа с плавающей точкой (Floating-Point)

```rust
let x = 2.0;      // f64 (по умолчанию)
let y: f32 = 3.0; // f32

// Специальные значения
let infinity = f64::INFINITY;
let neg_infinity = f64::NEG_INFINITY;
let nan = f64::NAN;

// Проверки
assert!(infinity.is_infinite());
assert!(nan.is_nan());
```

### Булев тип (Boolean)

```rust
let t = true;
let f: bool = false;

// Использование в условиях
if t {
    println!("It's true!");
}
```

### Символьный тип (Character)

```rust
let c = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';

// char - 4 байта, Unicode Scalar Value
let unicode: char = '\u{1F600}'; // 😀
```

## 📦 Составные типы

### Кортежи (Tuples)

```rust
// Объявление
let tup: (i32, f64, u8) = (500, 6.4, 1);

// Деструктуризация
let (x, y, z) = tup;
println!("The value of y is: {}", y);

// Доступ по индексу
let five_hundred = tup.0;
let six_point_four = tup.1;
let one = tup.2;

// Unit type - пустой кортеж
let unit: () = ();
```

### Массивы (Arrays)

```rust
// Фиксированный размер
let a: [i32; 5] = [1, 2, 3, 4, 5];

// Инициализация одинаковыми значениями
let a = [3; 5]; // [3, 3, 3, 3, 3]

// Доступ к элементам
let first = a[0];
let second = a[1];

// Срезы (Slices)
let slice: &[i32] = &a[1..3]; // [2, 3]
```

## 🎨 Пользовательские типы

### Type Aliases

```rust
type Kilometers = i32;
type Thunk = Box<dyn Fn() + Send + 'static>;

let x: Kilometers = 5;
let y: i32 = 5;
assert_eq!(x, y); // Одинаковые типы
```

### Never Type

```rust
fn bar() -> ! {
    panic!("This function never returns");
}

// Используется в match
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue, // continue имеет тип !
};
```

## 🔄 Преобразование типов

### Явное преобразование (Casting)

```rust
let decimal = 65.4321_f32;

// Явное преобразование
let integer = decimal as u8;
let character = integer as char;

// Безопасное преобразование с проверками
let parsed: i32 = "5".parse().unwrap();
let maybe_parsed = "abc".parse::<i32>(); // Result<i32, ParseIntError>
```

### From и Into traits

```rust
let my_str = "hello";
let my_string = String::from(my_str);

// Into - обратный From
let num = 5;
let num_string: String = num.to_string();
```

## 📊 Размеры типов

```rust
use std::mem;

println!("Size of i8: {} bytes", mem::size_of::<i8>());      // 1
println!("Size of i32: {} bytes", mem::size_of::<i32>());    // 4
println!("Size of i64: {} bytes", mem::size_of::<i64>());    // 8
println!("Size of f32: {} bytes", mem::size_of::<f32>());    // 4
println!("Size of f64: {} bytes", mem::size_of::<f64>());    // 8
println!("Size of bool: {} bytes", mem::size_of::<bool>());  // 1
println!("Size of char: {} bytes", mem::size_of::<char>());  // 4
```

## 🔗 Связанные концепции

- [[01_Core/01_Basics/01_Variables|Variables]] - объявление переменных
- [[01_Core/05_Structs_Enums/01_Structs|Structs]] - пользовательские структуры
- [[01_Core/05_Structs_Enums/02_Enums|Enums]] - перечисления
- [[01_Core/07_Generics/00_Index|Generics]] - обобщенные типы

## 💻 Практические примеры

### Работа с числами
```rust
fn number_operations() {
    // Арифметические операции
    let sum = 5 + 10;
    let difference = 95.5 - 4.3;
    let product = 4 * 30;
    let quotient = 56.7 / 32.2;
    let remainder = 43 % 5;
    
    // Переполнение
    let x: u8 = 255;
    // let y = x + 1; // panic в debug mode
    let y = x.wrapping_add(1); // 0
    let z = x.saturating_add(1); // 255
    let (result, overflowed) = x.overflowing_add(1); // (0, true)
}
```

### Работа с кортежами
```rust
fn tuple_operations() -> (i32, i32) {
    let point = (3, 5);
    let (x, y) = point;
    
    // Возврат нескольких значений
    (x * 2, y * 2)
}
```

### Работа с массивами
```rust
fn array_operations() {
    let months = ["January", "February", "March", "April", 
                  "May", "June", "July", "August", 
                  "September", "October", "November", "December"];
    
    for month in &months {
        println!("{}", month);
    }
    
    // Проверка границ
    let index = 10;
    if index < months.len() {
        println!("Month: {}", months[index]);
    }
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: Какой тип по умолчанию для целых чисел в Rust?
A: i32
<!--SR:!2024-01-18,4,270-->

#flashcard 
Q: Какой тип по умолчанию для чисел с плавающей точкой?
A: f64
<!--SR:!2024-01-19,5,280-->

#flashcard 
Q: Сколько байт занимает тип char в Rust?
A: 4 байта (Unicode Scalar Value)
<!--SR:!2024-01-20,3,250-->

#flashcard 
Q: В чем разница между массивом и срезом?
A: Массив имеет фиксированный размер, известный во время компиляции. Срез - это ссылка на часть массива с динамическим размером
<!--SR:!2024-01-21,4,265-->

## ⚠️ Частые ошибки

### Ошибка 1: Переполнение целых чисел
```rust
// ❌ Panic в debug mode
let x: u8 = 255;
let y = x + 1;

// ✅ Безопасные методы
let y = x.wrapping_add(1);    // Циклическое переполнение
let y = x.saturating_add(1);  // Насыщение на максимуме
let y = x.checked_add(1);      // Option<u8>
```

### Ошибка 2: Выход за границы массива
```rust
// ❌ Panic при выходе за границы
let arr = [1, 2, 3];
let element = arr[5];

// ✅ Безопасный доступ
let element = arr.get(5); // Option<&i32>
if let Some(val) = arr.get(5) {
    println!("Value: {}", val);
}
```

## 📝 Упражнения

1. **Создайте функцию для конвертации температур**:
   - Из Цельсия в Фаренгейт
   - Используйте f64

2. **Работа с кортежами**:
   - Создайте функцию, возвращающую min и max из массива
   - Используйте деструктуризацию

3. **Безопасная арифметика**:
   - Реализуйте калькулятор с проверкой переполнения
   - Используйте checked_* методы

## 📚 Дополнительные ресурсы

- [Rust Book - Data Types](https://doc.rust-lang.org/book/ch03-02-data-types.html)
- [[01_Core/01_Basics/00_Index|Basics Index]]
- [[01_Core/05_Structs_Enums/00_Index|Custom Types]]

---
#rust #types #data-types #basics #core
