# 🔄 Замыкания (Closures)

Замыкания - анонимные функции, которые могут захватывать переменные из окружающей области видимости.

## 📋 Содержание

### Основы замыканий
- [[Closure-Syntax]] - Синтаксис замыканий
- [[Closure-Capture]] - Способы захвата переменных
- [[Move-Closures]] - Перемещающие замыкания
- [[Fn-Traits]] - Трейты Fn, FnMut, FnOnce

### Практическое применение
- [[Iterators-Closures]] - Замыкания с итераторами
- [[Higher-Order-Functions]] - Функции высшего порядка
- [[Lazy-Evaluation]] - Ленивые вычисления

## 🔧 Основы замыканий

### Базовый синтаксис
```rust
fn main() {
    // Простое замыкание
    let add_one = |x| x + 1;
    println!("5 + 1 = {}", add_one(5));

    // С явными типами
    let add_one_v2 = |x: i32| -> i32 { x + 1 };
    
    // Многострочное
    let expensive_calculation = |num| {
        println!("calculating slowly...");
        std::thread::sleep(std::time::Duration::from_secs(2));
        num
    };
}
```

### Сравнение с функциями
```rust
// Функция
fn add_one_v1(x: u32) -> u32 { x + 1 }

// Замыкания - типы выводятся автоматически
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

## 🎯 Захват переменных

### Захват по ссылке
```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x; // Захватывает x по ссылке

    let y = 4;
    assert!(equal_to_x(y));
    
    println!("x is still accessible: {}", x); // x всё ещё доступна
}
```

### Move замыкания
```rust
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x; // Перемещает x в замыкание

    // println!("can't use x here: {:?}", x); // ❌ Ошибка!

    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```

### Изменение захваченных переменных
```rust
fn main() {
    let mut s = String::from("hello");
    
    let mut update_string = |str| {
        s.push_str(str);
        println!("{}", s);
    };
    
    update_string(" world");
    update_string("!");
    // s теперь содержит "hello world!"
}
```

## 🏷️ Трейты Fn

### FnOnce - вызывается один раз
```rust
fn call_once<F>(closure: F) 
where
    F: FnOnce(),
{
    closure();
}

fn main() {
    let x = vec![1, 2, 3];
    
    let consume_x = move || {
        println!("x: {:?}", x);
        // x потребляется здесь
    };
    
    call_once(consume_x);
    // consume_x больше нельзя использовать
}
```

### FnMut - может изменять захваченные переменные
```rust
fn call_mut<F>(mut closure: F) 
where
    F: FnMut(),
{
    closure();
    closure(); // Можно вызвать несколько раз
}

fn main() {
    let mut s = String::from("hello");
    
    let mut add_world = || {
        s.push_str(" world");
    };
    
    call_mut(add_world);
}
```

### Fn - может вызываться много раз без изменений
```rust
fn call_many_times<F>(closure: F) 
where
    F: Fn(),
{
    closure();
    closure();
    closure();
}

fn main() {
    let x = 10;
    
    let print_x = || println!("x: {}", x);
    
    call_many_times(print_x);
}
```

## 🔄 Замыкания с итераторами

### Map и filter
```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];
    
    let v2: Vec<_> = v1.iter()
        .map(|x| x + 1)  // Замыкание в map
        .collect();
    
    assert_eq!(v2, vec![2, 3, 4]);
    
    // Фильтрация с замыканием
    let v3: Vec<_> = v1.iter()
        .filter(|&x| *x > 1)  // Замыкание в filter
        .collect();
    
    assert_eq!(v3, vec![&2, &3]);
}
```

### Захват внешних переменных в итераторах
```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)  // Захватывает shoe_size
        .collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 13, style: String::from("sandal") },
            Shoe { size: 10, style: String::from("boot") },
        ];

        let in_my_size = shoes_in_my_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe { size: 10, style: String::from("sneaker") },
                Shoe { size: 10, style: String::from("boot") },
            ]
        );
    }
}
```

## 🚀 Продвинутые техники

### Возврат замыканий
```rust
fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    move |y| x + y
}

fn main() {
    let add_5 = make_adder(5);
    println!("5 + 3 = {}", add_5(3)); // 8
}
```

### Замыкания как параметры
```rust
fn apply_operation<F>(numbers: Vec<i32>, op: F) -> Vec<i32>
where
    F: Fn(i32) -> i32,
{
    numbers.iter().map(|&x| op(x)).collect()
}

fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    let doubled = apply_operation(numbers.clone(), |x| x * 2);
    let squared = apply_operation(numbers.clone(), |x| x * x);
    
    println!("Doubled: {:?}", doubled); // [2, 4, 6, 8, 10]
    println!("Squared: {:?}", squared); // [1, 4, 9, 16, 25]
}
```

### Кэширование с замыканиями
```rust
use std::collections::HashMap;

struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    value: HashMap<u32, u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: HashMap::new(),
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value.get(&arg) {
            Some(v) => *v,
            None => {
                let v = (self.calculation)(arg);
                self.value.insert(arg, v);
                v
            }
        }
    }
}

fn main() {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        std::thread::sleep(std::time::Duration::from_secs(2));
        num
    });

    println!("First call: {}", expensive_result.value(10));  // Вычисляется
    println!("Second call: {}", expensive_result.value(10)); // Из кэша
}
```

## 💡 Практические применения

### Обработка конфигурации
```rust
fn configure_app<F>(setup: F) 
where
    F: FnOnce(&mut AppConfig),
{
    let mut config = AppConfig::default();
    setup(&mut config);
    // Используем настроенную конфигурацию
}

#[derive(Default)]
struct AppConfig {
    debug: bool,
    port: u16,
}

fn main() {
    configure_app(|config| {
        config.debug = true;
        config.port = 8080;
    });
}
```

### Event handlers
```rust
struct Button {
    label: String,
}

impl Button {
    fn on_click<F>(&self, handler: F) 
    where
        F: Fn(&str),
    {
        println!("Button '{}' was clicked!", self.label);
        handler(&self.label);
    }
}

fn main() {
    let button = Button {
        label: "Save".to_string(),
    };
    
    button.on_click(|label| {
        println!("Handling click for: {}", label);
        // Логика сохранения
    });
}
```

## 🎯 Сравнение производительности

### Замыкания vs функции
```rust
// Замыкания часто инлайнятся компилятором
let add_one = |x| x + 1;
let numbers: Vec<i32> = (0..1000000).map(add_one).collect();

// Эквивалентно функции по производительности
fn add_one_fn(x: i32) -> i32 { x + 1 }
let numbers: Vec<i32> = (0..1000000).map(add_one_fn).collect();
```

## ✅ Чек-лист изучения
- [ ] Понимаю синтаксис замыканий
- [ ] Различаю способы захвата переменных (по ссылке vs move)
- [ ] Использую замыкания с итераторами
- [ ] Понимаю трейты Fn, FnMut, FnOnce
- [ ] Могу передавать замыкания как параметры
- [ ] Возвращаю замыкания из функций

#closures #functional-programming #capture #higher-order-functions #intermediate