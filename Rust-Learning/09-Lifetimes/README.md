# ⏰ Времена жизни (Lifetimes) в Rust

Lifetimes - это способ Rust гарантировать, что ссылки действительны столько, сколько нужно, предотвращая висячие указатели.

## 📋 Содержание

### Основы времён жизни
- [[Lifetime-Basics]] - Что такое lifetimes и зачем они нужны
- [[Lifetime-Syntax]] - Синтаксис аннотаций времени жизни
- [[Lifetime-Elision]] - Правила вывода времён жизни
- [[Lifetime-in-Structs]] - Lifetimes в структурах

### Продвинутые темы
- [[Lifetime-Bounds]] - Ограничения по времени жизни
- [[Higher-Ranked-Lifetimes]] - Lifetimes высшего ранга
- [[Static-Lifetime]] - Специальное время жизни 'static
- [[Lifetime-Subtyping]] - Субтипизация времён жизни

## 🎯 Что такое lifetimes?

Lifetimes - это аннотации, которые говорят компилятору, как долго должны жить ссылки по отношению друг к другу.

### Проблема, которую решают lifetimes
```rust
// Эта функция не скомпилируется без аннотаций lifetime:
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
// ❌ Error: missing lifetime specifier
```

**Проблема**: Компилятор не знает, с каким из входных параметров связано время жизни возвращаемого значения.

## 🔧 Синтаксис аннотаций

### Основной синтаксис
```rust
// Функция с аннотациями lifetime
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Чтение: "функция longest принимает две ссылки на строки,
// которые живут как минимум время 'a, и возвращает ссылку
// на строку, которая также живёт время 'a"
```

### Множественные lifetimes
```rust
fn announce_and_return_part<'a, 'b>(
    announcement: &'a str, 
    part: &'b str
) -> &'a str {
    println!("Attention please: {}", announcement);
    announcement
}

// 'a и 'b - независимые времена жизни
// Возвращаемое значение связано только с 'a
```

## 🏗️ Lifetimes в структурах

### Структура со ссылками
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

**Объяснение**: Структура `ImportantExcerpt` не может жить дольше ссылки в её поле `part`.

## ⚡ Правила вывода lifetimes

Rust может автоматически выводить lifetimes в простых случаях:

### Правило 1: Каждый параметр-ссылка получает свой lifetime
```rust
// Компилятор видит это:
fn first_word(s: &str) -> &str {

// Как это:
fn first_word<'a>(s: &'a str) -> &str {
```

### Правило 2: Если есть один входной lifetime, он применяется к выходу
```rust
// Компилятор видит это:
fn first_word<'a>(s: &'a str) -> &str {

// Как это:
fn first_word<'a>(s: &'a str) -> &'a str {
```

### Правило 3: Если есть `&self` или `&mut self`, их lifetime применяется к выходу
```rust
impl<'a> ImportantExcerpt<'a> {
    // Компилятор видит это:
    fn announce_and_return_part(&self, announcement: &str) -> &str {
    
    // Как это:
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        // Возвращаемое значение получает lifetime от &self
    }
}
```

## 🔍 Практические примеры

### Пример 1: Поиск самой длинной строки
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");
    
    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
    // result не может жить дольше этого блока,
    // потому что string2 уничтожается здесь
}
```

### Пример 2: Структура с методами
```rust
struct Parser<'a> {
    content: &'a str,
    position: usize,
}

impl<'a> Parser<'a> {
    fn new(content: &'a str) -> Self {
        Parser {
            content,
            position: 0,
        }
    }
    
    fn current_char(&self) -> Option<char> {
        self.content.chars().nth(self.position)
    }
    
    fn advance(&mut self) {
        self.position += 1;
    }
    
    fn parse_word(&mut self) -> &'a str {
        let start = self.position;
        
        while let Some(ch) = self.current_char() {
            if ch.is_alphabetic() {
                self.advance();
            } else {
                break;
            }
        }
        
        &self.content[start..self.position]
    }
}
```

## 🌟 Специальный lifetime: `'static`

### Что означает `'static`
```rust
// Строковые литералы имеют 'static lifetime
let s: &'static str = "I have a static lifetime.";

// Это означает, что ссылка действительна всё время работы программы
```

### Использование в функциях
```rust
fn returns_static() -> &'static str {
    "This is a string literal"
}

// Но не всё, что возвращает &'static, должно жить вечно!
fn make_static<T>(t: T) -> &'static T 
where 
    T: 'static 
{
    Box::leak(Box::new(t))  // "Утечка" в глобальную память
}
```

## 🚨 Частые ошибки и их решения

### Ошибка 1: Lifetime mismatch
```rust
// ❌ Не работает:
fn invalid_output<'a>() -> &'a str {
    let result = String::from("Hello");
    result.as_str() // Error: возвращаем ссылку на локальную переменную
}

// ✅ Исправлено:
fn valid_output() -> String {
    String::from("Hello") // Возвращаем владение, не ссылку
}

// Или:
fn valid_static_output() -> &'static str {
    "Hello" // Строковый литерал
}
```

### Ошибка 2: Borrowed value does not live long enough
```rust
// ❌ Не работает:
fn main() {
    let r;
    {
        let x = 5;
        r = &x; // x не живёт достаточно долго
    }
    println!("{}", r); // Error!
}

// ✅ Исправлено:
fn main() {
    let x = 5;
    let r = &x; // x и r в одной области видимости
    println!("{}", r); // OK
}
```

### Ошибка 3: Неправильные lifetime bounds
```rust
// ❌ Слишком строгое ограничение:
fn combine_strings<'a>(x: &'a str, y: &'a str) -> String {
    format!("{} {}", x, y)
}

// ✅ Lifetimes не нужны, так как возвращаем owned String:
fn combine_strings(x: &str, y: &str) -> String {
    format!("{} {}", x, y)
}
```

## 💡 Лучшие практики

### 1. Избегайте lifetimes когда возможно
```rust
// Вместо возврата ссылок:
fn process_string<'a>(s: &'a str) -> &'a str {
    // processing...
    s
}

// Лучше возвращать owned значения:
fn process_string(s: &str) -> String {
    // processing...
    s.to_string()
}
```

### 2. Используйте осмысленные имена lifetimes
```rust
// Вместо общих 'a, 'b:
fn parse_config<'a>(content: &'a str) -> Config<'a> {

// Лучше описательные имена:
fn parse_config<'content>(content: &'content str) -> Config<'content> {
```

### 3. Начинайте без аннотаций
```rust
// Сначала напишите код без lifetimes:
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}

// Добавляйте только когда компилятор требует:
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

## 🔧 Продвинутые техники

### Higher-Ranked Trait Bounds (HRTB)
```rust
fn call_with_any<F>(f: F) 
where 
    F: for<'a> Fn(&'a str) -> &'a str
{
    let s = String::from("hello");
    let result = f(&s);
    println!("{}", result);
}

// "для любого lifetime 'a, F может принять &'a str и вернуть &'a str"
```

### Lifetime bounds в generics
```rust
struct Ref<'a, T: 'a> {
    value: &'a T,
}

// T: 'a означает, что T должен жить как минимум время 'a
// Это нужно, если T может содержать ссылки
```

## 🎯 Упражнения для практики

### Начальный уровень
1. Напишите функцию, которая возвращает первое слово из строки
2. Создайте структуру, содержащую ссылку, и реализуйте для неё методы
3. Исправьте ошибки lifetime в предложенном коде

### Средний уровень
1. Реализуйте итератор для разбора строки по словам
2. Создайте структуру для кэширования результатов со ссылками
3. Напишите функцию для объединения нескольких ссылок

### Продвинутый уровень
1. Реализуйте простой парсер с lifetime аннотациями
2. Создайте структуру данных с несколькими lifetimes
3. Поэкспериментируйте с HRTB и lifetime bounds

## 🔗 Связанные темы
- [[02-Ownership/References]] - Основы ссылок
- [[08-Traits/README]] - Lifetimes в трейтах
- [[Compiler-Errors/E0623-Lifetime-Mismatch]] - Ошибки lifetimes

## 🧠 Ментальная модель

**Думайте о lifetimes как о "контрактах":**
- Функция говорит: "Если дашь мне ссылки, живущие время 'a, я верну ссылку, живущую не дольше 'a"
- Компилятор проверяет: "Действительно ли все ссылки живут достаточно долго?"
- Если контракт нарушается - ошибка компиляции

#lifetimes #references #memory-safety #advanced #borrow-checker