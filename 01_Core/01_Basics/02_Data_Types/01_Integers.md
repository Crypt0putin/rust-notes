# 🔢 Integers - Целочисленные типы

## 📋 Обзор целочисленных типов

Rust предоставляет целочисленные типы разных размеров и знаковости:

### Знаковые целые (Signed)
| Тип | Размер | Минимум | Максимум |
|-----|--------|---------|----------|
| `i8` | 8 бит | -128 | 127 |
| `i16` | 16 бит | -32,768 | 32,767 |
| `i32` | 32 бит | -2,147,483,648 | 2,147,483,647 |
| `i64` | 64 бит | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807 |
| `i128` | 128 бит | -(2^127) | 2^127 - 1 |
| `isize` | arch | зависит от архитектуры | зависит от архитектуры |

### Беззнаковые целые (Unsigned)
| Тип | Размер | Минимум | Максимум |
|-----|--------|---------|----------|
| `u8` | 8 бит | 0 | 255 |
| `u16` | 16 бит | 0 | 65,535 |
| `u32` | 32 бит | 0 | 4,294,967,295 |
| `u64` | 64 бит | 0 | 18,446,744,073,709,551,615 |
| `u128` | 128 бит | 0 | 2^128 - 1 |
| `usize` | arch | 0 | зависит от архитектуры |

## 💻 Объявление и инициализация

### Базовое использование
```rust
// Явное указание типа
let x: i32 = 42;
let y: u8 = 255;

// Вывод типа (по умолчанию i32)
let z = 100; // i32

// Суффиксы типов
let a = 57u8;
let b = 1_000_000i64;

// Разделители для читаемости
let big_number = 1_000_000;
let hex = 0xDEAD_BEEF;
```

### Литералы в разных системах счисления
```rust
let decimal = 98_222;        // Десятичная
let hex = 0xff;             // Шестнадцатеричная
let octal = 0o77;           // Восьмеричная
let binary = 0b1111_0000;   // Двоичная
let byte = b'A';            // Байт (только u8)
```

## 🔄 Преобразование типов

### Явное приведение (as)
```rust
let x: i32 = 100;
let y: i64 = x as i64;        // Расширение
let z: i16 = x as i16;        // Сужение (может потерять данные)
let unsigned: u32 = x as u32;  // Изменение знаковости
```

### Безопасное преобразование
```rust
use std::convert::TryFrom;

let x: i32 = 100;

// TryFrom для безопасного преобразования
match i8::try_from(x) {
    Ok(val) => println!("Converted: {}", val),
    Err(e) => println!("Conversion failed: {}", e),
}

// Методы проверки
let y: u32 = 200;
let result = y.checked_add(100);  // Some(300)
let overflow = u8::MAX.checked_add(1);  // None
```

## ⚙️ Арифметические операции

### Базовые операции
```rust
let a = 10;
let b = 3;

let sum = a + b;      // 13
let diff = a - b;     // 7
let product = a * b;  // 30
let quotient = a / b; // 3 (целочисленное деление)
let remainder = a % b; // 1 (остаток)
```

### Проверяемые операции (Checked)
```rust
let x: u8 = 250;

// Проверка на переполнение
let checked = x.checked_add(10);        // None (переполнение)
let safe = x.checked_sub(10);           // Some(240)
let mul = x.checked_mul(2);             // None
let div = x.checked_div(0);             // None (деление на 0)

// Saturating операции (ограничение значениями min/max)
let sat_add = x.saturating_add(10);     // 255 (u8::MAX)
let sat_sub = 0u8.saturating_sub(10);   // 0 (u8::MIN)

// Wrapping операции (циклическое переполнение)
let wrap_add = x.wrapping_add(10);      // 4 (250 + 10 - 256)
let wrap_sub = 0u8.wrapping_sub(1);     // 255

// Overflowing операции (возвращает кортеж)
let (result, overflowed) = x.overflowing_add(10); // (4, true)
```

## 🔬 Битовые операции

```rust
let a: u8 = 0b1010_1010;  // 170
let b: u8 = 0b0101_0101;  // 85

// Битовые операции
let and = a & b;          // 0b0000_0000 = 0
let or = a | b;           // 0b1111_1111 = 255
let xor = a ^ b;          // 0b1111_1111 = 255
let not = !a;             // 0b0101_0101 = 85

// Сдвиги
let left_shift = a << 1;  // 0b0101_0100 = 84 (с переполнением)
let right_shift = a >> 1; // 0b0101_0101 = 85

// Подсчет битов
let ones = a.count_ones();      // 4
let zeros = a.count_zeros();    // 4
let leading = a.leading_zeros(); // 0
let trailing = a.trailing_zeros(); // 1
```

## 🎯 Методы и свойства

```rust
let x: i32 = -42;
let y: u32 = 42;

// Проверки
println!("Is positive: {}", x.is_positive());  // false
println!("Is negative: {}", x.is_negative());  // true
println!("Is power of 2: {}", y.is_power_of_two()); // false

// Абсолютное значение
let abs = x.abs();                    // 42
let unsigned_abs = x.unsigned_abs();  // 42u32

// Min/Max
let min = i32::MIN;  // -2147483648
let max = i32::MAX;  // 2147483647

// Математические функции
let pow = 2i32.pow(10);          // 1024
let sqrt = 16u32.integer_sqrt(); // 4 (только для unsigned, nightly)

// Swap bytes (для сетевых операций)
let swapped = 0x1234u16.swap_bytes(); // 0x3412
let be = 0x1234u16.to_be();          // big-endian
let le = 0x1234u16.to_le();          // little-endian
```

## ⚠️ Частые ошибки и решения

### Переполнение в debug режиме
```rust
// ❌ Паника в debug режиме
let x: u8 = 255;
let y = x + 1; // panic in debug, wraps in release

// ✅ Используйте проверяемые операции
if let Some(result) = x.checked_add(1) {
    println!("Result: {}", result);
} else {
    println!("Overflow would occur");
}
```

### Неявное преобразование
```rust
// ❌ Rust не делает неявных преобразований
let x: i32 = 10;
let y: i64 = 20;
// let z = x + y; // ОШИБКА: несовпадение типов

// ✅ Явное приведение
let z = x as i64 + y; // OK
```

### Деление на ноль
```rust
// ❌ Паника при делении на 0
let x = 10;
let y = 0;
// let z = x / y; // panic!

// ✅ Проверка перед делением
if y != 0 {
    let z = x / y;
} else {
    println!("Division by zero");
}

// Или используйте checked_div
let safe_div = x.checked_div(y); // None
```

## 🎯 Флеш-карточки

#flashcard 
Q: Какой тип целых чисел используется по умолчанию в Rust?
A: i32
<!--SR:!2024-03-01,3,250-->

#flashcard 
Q: Что произойдет при переполнении u8::MAX + 1 в debug режиме?
A: Паника (panic) с сообщением об overflow
<!--SR:!2024-03-02,4,270-->

#flashcard 
Q: Какой метод использовать для безопасного сложения с проверкой переполнения?
A: checked_add() - возвращает Option<T>
<!--SR:!2024-03-03,5,280-->

## 📚 Практические примеры

### Пример 1: Безопасный калькулятор
```rust
fn safe_calculator(a: i32, b: i32, op: char) -> Option<i32> {
    match op {
        '+' => a.checked_add(b),
        '-' => a.checked_sub(b),
        '*' => a.checked_mul(b),
        '/' => {
            if b != 0 {
                a.checked_div(b)
            } else {
                None
            }
        }
        _ => None,
    }
}
```

### Пример 2: Работа с битовыми флагами
```rust
// Битовые флаги для permissions
const READ: u8 = 0b0000_0100;    // 4
const WRITE: u8 = 0b0000_0010;   // 2
const EXECUTE: u8 = 0b0000_0001; // 1

fn check_permissions(perms: u8) {
    if perms & READ != 0 {
        println!("Can read");
    }
    if perms & WRITE != 0 {
        println!("Can write");
    }
    if perms & EXECUTE != 0 {
        println!("Can execute");
    }
}

let full_perms = READ | WRITE | EXECUTE; // 7
check_permissions(full_perms);
```

## 🔗 Связанные темы

- [[01_Core/01_Basics/02_Data_Types/02_Floating_Point|Floating Point]] - числа с плавающей точкой
- [[01_Core/01_Basics/02_Data_Types/16_Type_Conversion|Type Conversion]] - преобразование типов
- [[01_Core/08_Error_Handling/00_Index|Error Handling]] - обработка ошибок переполнения
- [[Common Errors|Common Errors]] - частые ошибки с числами

---
#rust #integers #types #arithmetic #bitwise
