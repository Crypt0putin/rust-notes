# 🎲 Игра "Угадай число"

Классическая игра для изучения основ Rust: ввод пользователя, генерация случайных чисел, циклы и обработка ошибок.

## 🎯 Цели проекта

### Изучаемые концепции
- [[01-Fundamentals/Variables-Mutability]] - Изменяемые переменные
- [[04-Functions-Control/Control-Flow]] - Условия и циклы
- [[07-Error-Handling/Result-Type]] - Обработка ошибок
- [[06-Collections/String-Operations]] - Работа со строками
- [[External-Crates]] - Использование внешних библиотек

### Навыки программирования
- Пользовательский ввод
- Генерация случайных чисел  
- Парсинг строк в числа
- Логика игры
- Обработка некорректного ввода

## 🎮 Описание игры

Программа генерирует случайное число от 1 до 100, и игрок должен его угадать. После каждой попытки программа сообщает, больше или меньше загаданное число.

## 🛠️ Реализация

### Базовая версия
```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Угадайте число!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Пожалуйста, введите предположение.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Не удалось прочитать строку");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("Вы предположили: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Слишком маленькое!"),
            Ordering::Greater => println!("Слишком большое!"),
            Ordering::Equal => {
                println!("Вы угадали!");
                break;
            }
        }
    }
}
```

### Cargo.toml
```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8"
```

## 📚 Пошаговое объяснение

### Шаг 1: Импорты и настройка
```rust
use rand::Rng;        // Трейт для генерации случайных чисел
use std::cmp::Ordering; // Enum для результата сравнения
use std::io;          // Модуль для ввода-вывода
```

### Шаг 2: Генерация случайного числа
```rust
let secret_number = rand::thread_rng().gen_range(1..=100);
//                  ^^^^^^^^^^^^^^^   ^^^^^^^^^^^^^^^^
//                  Генератор         Диапазон 1-100 включительно
```

### Шаг 3: Основной цикл игры
```rust
loop {
    // Цикл продолжается до break
}
```

### Шаг 4: Получение пользовательского ввода
```rust
let mut guess = String::new(); // Изменяемая строка

io::stdin()
    .read_line(&mut guess)     // Читаем в изменяемую ссылку
    .expect("Не удалось прочитать строку"); // Panic при ошибке
```

### Шаг 5: Парсинг строки в число
```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,        // Успешный парсинг
    Err(_) => continue,    // Ошибка - повторить цикл
};
```

**Объяснение:**
- `trim()` - убирает пробелы и символы новой строки
- `parse()` - пытается преобразовать строку в число
- `match` - обрабатывает `Result<u32, ParseIntError>`

### Шаг 6: Сравнение и вывод результата
```rust
match guess.cmp(&secret_number) {
    Ordering::Less => println!("Слишком маленькое!"),
    Ordering::Greater => println!("Слишком большое!"),
    Ordering::Equal => {
        println!("Вы угадали!");
        break; // Выходим из цикла
    }
}
```

## 🚀 Улучшения и расширения

### 1. Ограничение количества попыток
```rust
const MAX_ATTEMPTS: u32 = 10;

fn main() {
    // ... начальная настройка ...
    let mut attempts = 0;

    loop {
        attempts += 1;
        
        // ... получение ввода ...
        
        if attempts >= MAX_ATTEMPTS {
            println!("Вы исчерпали все {} попыток! Число было: {}", 
                    MAX_ATTEMPTS, secret_number);
            break;
        }
        
        // ... остальная логика ...
    }
}
```

### 2. Счётчик попыток и статистика
```rust
struct GameStats {
    attempts: u32,
    games_played: u32,
    games_won: u32,
}

impl GameStats {
    fn new() -> Self {
        GameStats {
            attempts: 0,
            games_played: 0,
            games_won: 0,
        }
    }
    
    fn reset_attempts(&mut self) {
        self.attempts = 0;
        self.games_played += 1;
    }
    
    fn win_game(&mut self) {
        self.games_won += 1;
    }
    
    fn show_stats(&self) {
        println!("Игр сыграно: {}", self.games_played);
        println!("Игр выиграно: {}", self.games_won);
        if self.games_played > 0 {
            let win_rate = (self.games_won as f64 / self.games_played as f64) * 100.0;
            println!("Процент побед: {:.1}%", win_rate);
        }
    }
}
```

### 3. Разные уровни сложности
```rust
enum Difficulty {
    Easy,   // 1-50
    Medium, // 1-100  
    Hard,   // 1-500
}

impl Difficulty {
    fn range(&self) -> (u32, u32) {
        match self {
            Difficulty::Easy => (1, 50),
            Difficulty::Medium => (1, 100),
            Difficulty::Hard => (1, 500),
        }
    }
    
    fn max_attempts(&self) -> u32 {
        match self {
            Difficulty::Easy => 8,
            Difficulty::Medium => 10,
            Difficulty::Hard => 12,
        }
    }
}

fn choose_difficulty() -> Difficulty {
    println!("Выберите уровень сложности:");
    println!("1. Лёгкий (1-50)");
    println!("2. Средний (1-100)");  
    println!("3. Сложный (1-500)");
    
    // ... логика выбора ...
}
```

### 4. Валидация ввода
```rust
fn get_user_input() -> Result<u32, String> {
    let mut input = String::new();
    
    io::stdin()
        .read_line(&mut input)
        .map_err(|_| "Не удалось прочитать ввод".to_string())?;
    
    let number = input.trim().parse::<u32>()
        .map_err(|_| "Пожалуйста, введите действительное число".to_string())?;
    
    if number == 0 {
        return Err("Число должно быть больше 0".to_string());
    }
    
    Ok(number)
}

// Использование:
match get_user_input() {
    Ok(guess) => {
        // Обработать число
    },
    Err(error) => {
        println!("Ошибка: {}", error);
        continue;
    }
}
```

### 5. Цветной вывод
```rust
// Добавить в Cargo.toml:
// colored = "2.0"

use colored::*;

fn print_result(result: Ordering) {
    match result {
        Ordering::Less => println!("{}", "Слишком маленькое!".red()),
        Ordering::Greater => println!("{}", "Слишком большое!".red()),
        Ordering::Equal => println!("{}", "Вы угадали!".green().bold()),
    }
}
```

## 🧪 Тестирование

### Модульные тесты
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_difficulty_ranges() {
        assert_eq!(Difficulty::Easy.range(), (1, 50));
        assert_eq!(Difficulty::Medium.range(), (1, 100));
        assert_eq!(Difficulty::Hard.range(), (1, 500));
    }

    #[test]  
    fn test_game_stats() {
        let mut stats = GameStats::new();
        assert_eq!(stats.games_played, 0);
        
        stats.reset_attempts();
        assert_eq!(stats.games_played, 1);
        
        stats.win_game();
        assert_eq!(stats.games_won, 1);
    }
}
```

## 🎯 Задания для практики

### Начальный уровень
1. **Реализуйте базовую версию** игры по инструкции
2. **Добавьте валидацию ввода** для некорректных чисел
3. **Ограничьте диапазон** принимаемых чисел (например, 1-100)

### Средний уровень
1. **Добавьте счётчик попыток** и ограничение
2. **Реализуйте разные уровни сложности**
3. **Добавьте возможность играть несколько раундов**

### Продвинутый уровень
1. **Сохраняйте статистику игр** в файл
2. **Добавьте систему подсказок** (например, чётное/нечётное)
3. **Создайте GUI версию** с помощью crate вроде `eframe`

## 🔍 Анализ ошибок

### Частые проблемы:
1. **Бесконечное зацикливание** при неправильной обработке ввода
2. **Panic при parse()** вместо обработки ошибки
3. **Неправильная работа с изменяемостью**

### Отладочные техники:
- Добавляйте `println!` для отслеживания значений
- Используйте `cargo run` с `RUST_BACKTRACE=1`
- Тестируйте граничные случаи

## 🔗 Связанные концепции
- [[04-Functions-Control/Match-Expressions]] - Pattern matching
- [[07-Error-Handling/README]] - Обработка ошибок  
- [[External-Crates]] - Работа с внешними библиотеками

#project #guessing-game #beginner #interactive #game