# 🔀 Control Flow in Rust

## 🎯 Условные конструкции

### if выражения
```rust
fn main() {
    let number = 7;
    
    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

### else if цепочки
```rust
fn main() {
    let number = 6;
    
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

### if как выражение
```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };
    
    println!("The value of number is: {}", number);
    
    // Можно использовать в более сложных выражениях
    let result = if number > 3 {
        number * 2
    } else {
        number + 10
    };
}
```

## 🔄 Циклы

### loop - бесконечный цикл
```rust
fn main() {
    let mut counter = 0;
    
    let result = loop {
        counter += 1;
        
        if counter == 10 {
            break counter * 2; // Возвращаем значение
        }
    };
    
    println!("The result is {}", result);
}
```

### Метки циклов
```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;
        
        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up; // Выход из внешнего цикла
            }
            remaining -= 1;
        }
        
        count += 1;
    }
    println!("End count = {}", count);
}
```

### while - условный цикл
```rust
fn main() {
    let mut number = 3;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("LIFTOFF!!!");
}
```

### for - итерация
```rust
fn main() {
    // Итерация по массиву
    let a = [10, 20, 30, 40, 50];
    
    for element in a {
        println!("the value is: {}", element);
    }
    
    // Итерация по диапазону
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
    
    // Итерация с индексом
    for (index, value) in a.iter().enumerate() {
        println!("Index {}: {}", index, value);
    }
}
```

## 🎲 Pattern Matching

### match выражение
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### match с привязкой значений
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!("Move to x: {}, y: {}", x, y);
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!("Change color to red: {}, green: {}, blue: {}", r, g, b)
        }
    }
}
```

### Guards в match
```rust
fn main() {
    let num = Some(4);
    
    match num {
        Some(x) if x < 5 => println!("less than five: {}", x),
        Some(x) => println!("{}", x),
        None => (),
    }
}
```

### Подстановочный символ и игнорирование
```rust
fn main() {
    let some_value = 3u8;
    
    match some_value {
        1 => println!("one"),
        3 => println!("three"),
        5 => println!("five"),
        7 => println!("seven"),
        _ => (), // Игнорируем остальные значения
    }
    
    // Игнорирование частей значения
    let numbers = (2, 4, 8, 16, 32);
    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {}, {}, {}", first, third, fifth)
        }
    }
}
```

## 🎯 if let и while let

### if let для упрощения match
```rust
fn main() {
    let config_max = Some(3u8);
    
    // Вместо match
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
    
    // Используем if let
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
    
    // С else
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
}
```

### while let для циклов
```rust
fn main() {
    let mut stack = Vec::new();
    
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
}
```

## 🎨 Продвинутые паттерны

### Деструктуризация
```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    
    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
    
    // Сокращенная форма
    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

### Диапазоны в match
```rust
fn main() {
    let x = 5;
    
    match x {
        1..=5 => println!("one through five"),
        6..=10 => println!("six through ten"),
        _ => println!("something else"),
    }
    
    let x = 'c';
    
    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
}
```

### @ привязки
```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
    } => println!("Found an id in range: {}", id_variable),
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    }
    Message::Hello { id } => println!("Found some other id: {}", id),
}
```

## 🔗 Связанные концепции

- [[01_Core/01_Basics/03_Functions|Functions]] - функции и выражения
- [[01_Core/05_Structs_Enums/02_Enums|Enums]] - перечисления для match
- [[01_Core/05_Structs_Enums/03_Pattern_Matching|Pattern Matching]] - детали паттернов
- [[01_Core/08_Error_Handling/00_Index|Error Handling]] - обработка Option и Result

## 💻 Практические примеры

### Пример 1: FizzBuzz
```rust
fn fizzbuzz(n: u32) {
    for i in 1..=n {
        match (i % 3, i % 5) {
            (0, 0) => println!("FizzBuzz"),
            (0, _) => println!("Fizz"),
            (_, 0) => println!("Buzz"),
            _ => println!("{}", i),
        }
    }
}
```

### Пример 2: Обработка команд
```rust
enum Command {
    Move { x: f64, y: f64 },
    Rotate(f64),
    Scale(f64),
    Quit,
}

fn execute_command(cmd: Command) {
    match cmd {
        Command::Move { x, y } if x > 0.0 && y > 0.0 => {
            println!("Moving to positive coordinates: {}, {}", x, y);
        }
        Command::Move { x, y } => {
            println!("Moving to: {}, {}", x, y);
        }
        Command::Rotate(angle) => {
            println!("Rotating by {} degrees", angle);
        }
        Command::Scale(factor) if factor > 0.0 => {
            println!("Scaling by {}", factor);
        }
        Command::Scale(_) => {
            println!("Invalid scale factor");
        }
        Command::Quit => {
            println!("Quitting...");
        }
    }
}
```

### Пример 3: Итераторы с фильтрацией
```rust
fn process_data() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Функциональный стиль
    let even_squares: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .collect();
    
    println!("Even squares: {:?}", even_squares);
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: Как выйти из вложенного цикла во внешний?
A: Использовать метки циклов: 'label: loop { ... break 'label; }
<!--SR:!2024-01-25,4,270-->

#flashcard 
Q: В чем разница между if и match?
A: if проверяет булевы условия, match сопоставляет паттерны и должен быть исчерпывающим
<!--SR:!2024-01-26,5,280-->

#flashcard 
Q: Как вернуть значение из loop?
A: Использовать break со значением: break value;
<!--SR:!2024-01-27,3,250-->

## ⚠️ Частые ошибки

### Ошибка 1: Неисчерпывающий match
```rust
// ❌ ОШИБКА
enum Color {
    Red,
    Green,
    Blue,
}

let color = Color::Red;
match color {
    Color::Red => println!("Red"),
    Color::Green => println!("Green"),
    // Забыли Blue!
}

// ✅ Правильно
match color {
    Color::Red => println!("Red"),
    Color::Green => println!("Green"),
    Color::Blue => println!("Blue"),
}
// Или
match color {
    Color::Red => println!("Red"),
    _ => println!("Not red"),
}
```

### Ошибка 2: Разные типы в ветках if
```rust
// ❌ ОШИБКА
let number = if condition {
    5
} else {
    "six" // Другой тип!
};

// ✅ Правильно
let number = if condition {
    5
} else {
    6
};
```

## 📝 Упражнения

1. **Реализуйте игру "Камень-ножницы-бумага"**:
   - Используйте enum для вариантов
   - match для определения победителя

2. **Напишите функцию поиска в массиве**:
   - Используйте for с enumerate
   - Верните Option<usize> с индексом

3. **Создайте конечный автомат**:
   - enum для состояний
   - match для переходов

## 📚 Дополнительные ресурсы

- [Rust Book - Control Flow](https://doc.rust-lang.org/book/ch03-05-control-flow.html)
- [[01_Core/01_Basics/00_Index|Basics Index]]
- [[01_Core/05_Structs_Enums/03_Pattern_Matching|Pattern Matching Deep Dive]]

---
#rust #control-flow #loops #match #basics #core
