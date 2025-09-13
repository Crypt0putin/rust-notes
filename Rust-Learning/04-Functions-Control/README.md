# 🔧 Функции и управление потоком

Функции и конструкции управления потоком - основа структурированного программирования в Rust.

## 📋 Содержание

### Функции
- [[Function-Definition]] - Определение функций
- [[Function-Parameters]] - Параметры и аргументы
- [[Return-Values]] - Возвращаемые значения
- [[Associated-Functions]] - Ассоциированные функции

### Управление потоком
- [[If-Expressions]] - Условные выражения
- [[Loops]] - Циклы: loop, while, for
- [[Match-Expressions]] - Сопоставление с образцом
- [[Pattern-Matching]] - Продвинутое сопоставление

## 🔧 Основы функций

### Определение функции
```rust
fn main() {
    println!("Hello, world!");
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

### Параметры функций
```rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

### Возвращаемые значения
```rust
fn five() -> i32 {
    5 // Выражение без точки с запятой
}

fn plus_one(x: i32) -> i32 {
    x + 1 // Последнее выражение возвращается
}

fn explicit_return(x: i32) -> i32 {
    return x + 1; // Явный return
}
```

## 🎯 Управление потоком

### If выражения
```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
    
    // if в выражении
    let condition = true;
    let number = if condition { 5 } else { 6 };
    println!("The value of number is: {}", number);
}
```

### Циклы
```rust
fn main() {
    // Бесконечный цикл
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2; // Возврат значения из loop
        }
    };
    
    // while цикл
    let mut number = 3;
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    // for цикл
    let a = [10, 20, 30, 40, 50];
    for element in a.iter() {
        println!("the value is: {}", element);
    }
    
    // for с диапазоном
    for number in (1..4).rev() {
        println!("{}!", number);
    }
}
```

### Match выражения
```rust
#[derive(Debug)]
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {
    let coin = Coin::Quarter;
    println!("Value: {}", value_in_cents(coin));
}
```

## 💡 Продвинутые техники

### Замыкания (краткий обзор)
```rust
use std::thread;
use std::time::Duration;

fn main() {
    let expensive_closure = |num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    println!("Result: {}", expensive_closure(5));
}
```

### Функции высшего порядка
```rust
fn apply_to_3<F>(f: F) -> i32
where
    F: Fn(i32) -> i32,
{
    f(3)
}

fn main() {
    let double = |x| x * 2;
    println!("Result: {}", apply_to_3(double)); // 6
}
```

## ✅ Чек-лист изучения
- [ ] Умею определять функции с параметрами
- [ ] Понимаю разницу между выражениями и операторами
- [ ] Использую if как выражение
- [ ] Работаю с разными типами циклов
- [ ] Применяю match для сопоставления с образцом

#functions #control-flow #loops #match #beginner