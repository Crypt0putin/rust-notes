# 🔄 Паттерны итераторов

Полезные паттерны и техники работы с итераторами в Rust.

## 🎯 Базовые трансформации

### Map и collect
```rust
let numbers = vec![1, 2, 3, 4, 5];

// Удвоение всех чисел
let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();

// Преобразование типов
let strings: Vec<String> = numbers.iter().map(|x| x.to_string()).collect();

// В HashMap
use std::collections::HashMap;
let map: HashMap<i32, String> = numbers
    .iter()
    .map(|&x| (x, format!("value_{}", x)))
    .collect();
```

### Filter и условия
```rust
let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Фильтрация чётных чисел
let even: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();

// Комбинирование filter и map
let even_doubled: Vec<i32> = numbers
    .iter()
    .filter(|&&x| x % 2 == 0)
    .map(|x| x * 2)
    .collect();

// Filter map (фильтрация с трансформацией)
let parsed: Vec<i32> = vec!["1", "2", "not_a_number", "4"]
    .iter()
    .filter_map(|s| s.parse().ok())
    .collect();
```

## 📊 Агрегация данных

### Суммирование и произведение
```rust
let numbers = vec![1, 2, 3, 4, 5];

// Сумма
let sum: i32 = numbers.iter().sum();

// Произведение
let product: i32 = numbers.iter().product();

// Сумма квадратов
let sum_of_squares: i32 = numbers.iter().map(|x| x * x).sum();

// Средее арифметическое
let average = numbers.iter().sum::<i32>() as f64 / numbers.len() as f64;
```

### Fold и reduce
```rust
let numbers = vec![1, 2, 3, 4, 5];

// Fold с начальным значением
let sum = numbers.iter().fold(0, |acc, x| acc + x);

// Поиск максимума с fold
let max = numbers.iter().fold(i32::MIN, |acc, &x| acc.max(x));

// Reduce без начального значения
let product = numbers.iter().reduce(|acc, x| acc * x); // Option<&i32>

// Создание строки
let concatenated = vec!["hello", "world", "rust"]
    .iter()
    .fold(String::new(), |mut acc, &s| {
        if !acc.is_empty() {
            acc.push(' ');
        }
        acc.push_str(s);
        acc
    });
```

## 🔍 Поиск и проверки

### Find и position
```rust
let numbers = vec![10, 20, 30, 40, 50];

// Поиск первого элемента больше 25
let found = numbers.iter().find(|&&x| x > 25); // Some(&30)

// Поиск позиции
let position = numbers.iter().position(|&x| x > 25); // Some(2)

// Поиск с индексом
let (index, value) = numbers
    .iter()
    .enumerate()
    .find(|(_, &x)| x > 25)
    .unwrap(); // (2, &30)
```

### Any и all
```rust
let numbers = vec![2, 4, 6, 8];

// Проверка условий
let all_even = numbers.iter().all(|&x| x % 2 == 0); // true
let any_greater_5 = numbers.iter().any(|&x| x > 5); // true
let has_zero = numbers.iter().any(|&x| x == 0); // false

// Проверка пустоты через any
let is_not_empty = numbers.iter().any(|_| true);
```

## 🔀 Группировка и партиционирование

### Partition
```rust
let numbers = vec![1, 2, 3, 4, 5, 6];

// Разделение на чётные и нечётные
let (even, odd): (Vec<i32>, Vec<i32>) = numbers
    .into_iter()
    .partition(|&x| x % 2 == 0);

// Partition с более сложным условием
let words = vec!["hello", "world", "rust", "is", "awesome"];
let (short, long): (Vec<&str>, Vec<&str>) = words
    .into_iter()
    .partition(|word| word.len() <= 4);
```

### Group by (с внешней библиотекой itertools)
```rust
// Cargo.toml: itertools = "0.11"
use itertools::Itertools;

let data = vec![("A", 1), ("B", 2), ("A", 3), ("B", 4), ("A", 5)];

// Группировка по ключу
let grouped: Vec<(char, Vec<i32>)> = data
    .into_iter()
    .into_group_map();

// Или ручная группировка с fold
use std::collections::HashMap;
let grouped: HashMap<char, Vec<i32>> = data
    .into_iter()
    .fold(HashMap::new(), |mut acc, (key, value)| {
        acc.entry(key).or_insert_with(Vec::new).push(value);
        acc
    });
```

## 🔗 Работа с вложенными структурами

### Flatten
```rust
let nested = vec![vec![1, 2], vec![3, 4, 5], vec![6]];

// Развёртывание вложенных векторов
let flat: Vec<i32> = nested.iter().flatten().cloned().collect();
// [1, 2, 3, 4, 5, 6]

// Flat map = map + flatten
let doubled_flat: Vec<i32> = nested
    .iter()
    .flat_map(|inner| inner.iter().map(|x| x * 2))
    .collect();
// [2, 4, 6, 8, 10, 12]
```

### Работа с Option в итераторах
```rust
let maybe_numbers: Vec<Option<i32>> = vec![Some(1), None, Some(3), Some(4), None];

// Фильтрация Some значений
let numbers: Vec<i32> = maybe_numbers.iter().filter_map(|&x| x).collect();
// [1, 3, 4]

// Или с flatten
let numbers: Vec<i32> = maybe_numbers.iter().flatten().cloned().collect();
```

## 🎮 Продвинутые техники

### Take while и skip while
```rust
let numbers = vec![1, 2, 3, 4, 5, 4, 3, 2, 1];

// Брать пока условие выполняется
let ascending: Vec<&i32> = numbers.iter().take_while(|&&x| x < 5).collect();
// [&1, &2, &3, &4]

// Пропускать пока условие выполняется
let after_peak: Vec<&i32> = numbers.iter().skip_while(|&&x| x < 5).collect();
// [&5, &4, &3, &2, &1]
```

### Cycle и repeat
```rust
// Бесконечный цикл
let pattern: Vec<i32> = vec![1, 2, 3]
    .iter()
    .cycle()
    .take(7)
    .cloned()
    .collect();
// [1, 2, 3, 1, 2, 3, 1]

// Repeat значения
let repeated: Vec<i32> = std::iter::repeat(42)
    .take(5)
    .collect();
// [42, 42, 42, 42, 42]
```

### Scan (аккумулятор с состоянием)
```rust
let numbers = vec![1, 2, 3, 4, 5];

// Накопительные суммы
let cumsum: Vec<i32> = numbers
    .iter()
    .scan(0, |acc, &x| {
        *acc += x;
        Some(*acc)
    })
    .collect();
// [1, 3, 6, 10, 15]

// Скользящее среднее (упрощённо)
let values = vec![10, 20, 30, 40, 50];
let moving_avg: Vec<f64> = values
    .windows(3)
    .map(|window| window.iter().sum::<i32>() as f64 / 3.0)
    .collect();
```

## 📝 Работа со строками

### Разбор и обработка текста
```rust
let text = "hello world rust programming";

// Подсчёт слов по длине
use std::collections::HashMap;
let word_lengths: HashMap<usize, usize> = text
    .split_whitespace()
    .map(|word| word.len())
    .fold(HashMap::new(), |mut acc, len| {
        *acc.entry(len).or_insert(0) += 1;
        acc
    });

// Самые длинные слова
let long_words: Vec<&str> = text
    .split_whitespace()
    .filter(|word| word.len() > 4)
    .collect();

// Создание предложения
let capitalized = text
    .split_whitespace()
    .map(|word| {
        let mut chars: Vec<char> = word.chars().collect();
        if !chars.is_empty() {
            chars[0] = chars[0].to_uppercase().next().unwrap_or(chars[0]);
        }
        chars.iter().collect::<String>()
    })
    .collect::<Vec<String>>()
    .join(" ");
```

## 🔢 Числовые последовательности

### Математические операции
```rust
// Числа Фибоначчи
let fibonacci: Vec<u64> = (0..10)
    .scan((0, 1), |state, _| {
        let next = state.0 + state.1;
        *state = (state.1, next);
        Some(state.0)
    })
    .collect();

// Факториалы
let factorials: Vec<u64> = (1..=10)
    .scan(1, |acc, n| {
        *acc *= n;
        Some(*acc)
    })
    .collect();

// Простые числа (упрощённо)
let primes: Vec<i32> = (2..100)
    .filter(|&n| (2..=((n as f64).sqrt() as i32)).all(|i| n % i != 0))
    .collect();
```

## 🎯 Практические применения

### Обработка CSV данных (концептуально)
```rust
let csv_lines = vec![
    "Name,Age,City",
    "Alice,30,New York", 
    "Bob,25,London",
    "Charlie,35,Tokyo",
];

// Парсинг данных
let people: Vec<(&str, u32, &str)> = csv_lines
    .iter()
    .skip(1) // Пропускаем заголовок
    .filter_map(|line| {
        let parts: Vec<&str> = line.split(',').collect();
        if parts.len() == 3 {
            parts[1].parse().ok().map(|age| (parts[0], age, parts[2]))
        } else {
            None
        }
    })
    .collect();

// Статистика
let avg_age = people.iter().map(|(_, age, _)| age).sum::<u32>() as f64 / people.len() as f64;
let cities: Vec<&str> = people.iter().map(|(_, _, city)| *city).collect();
```

### Lazy evaluation
```rust
// Итераторы ленивые - вычисления происходят только при потреблении
let expensive_computation = (0..1_000_000)
    .map(|x| x * x) // Ещё не выполняется!
    .filter(|&x| x % 1000 == 0) // И это тоже!
    .take(5); // И это!

// Только здесь происходят вычисления
let results: Vec<i32> = expensive_computation.collect();
```

#snippet #iterators #functional #data-processing #intermediate