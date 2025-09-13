# 🏗️ Определение структур

Структуры в Rust позволяют группировать связанные данные и создавать пользовательские типы.

## 🎯 Синтаксис определения

### Базовая структура
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

### Создание экземпляра
```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

## 📝 Типы структур

### 1. Именованные поля (Named fields)
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

let rect = Rectangle {
    width: 30,
    height: 50,
};

// Доступ к полям
println!("Width: {}", rect.width);
```

### 2. Кортежные структуры (Tuple structs)
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);

// Доступ по индексу
println!("Red: {}", black.0);
println!("X: {}", origin.0);
```

### 3. Единичные структуры (Unit structs)
```rust
struct AlwaysEqual;

let subject = AlwaysEqual;

// Полезны для реализации трейтов без данных
impl SomeTrait for AlwaysEqual {
    // методы трейта
}
```

## 🔧 Работа с полями

### Сокращённый синтаксис инициализации
```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,      // Эквивалентно email: email
        username,   // Эквивалентно username: username
        active: true,
        sign_in_count: 1,
    }
}
```

### Синтаксис обновления структур
```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1  // Остальные поля копируются из user1
};

// ⚠️ После этого user1 частично перемещён!
// Поля active и sign_in_count ещё доступны (они Copy)
// Но email и username перемещены в user2
```

## 🔒 Изменяемость

### Изменяемые экземпляры
```rust
let mut user = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

// Изменение поля
user.email = String::from("anotheremail@example.com");
user.sign_in_count += 1;
```

### Вся структура изменяемая или неизменяемая
```rust
// ❌ Нельзя сделать отдельные поля изменяемыми
struct User {
    username: String,
    email: String,
    sign_in_count: u64,  // Нельзя сделать только это поле mut
    active: bool,
}

// ✅ Изменяемость всего экземпляра
let mut user = User { /* ... */ };
```

## 🎨 Автоматические реализации

### Derive макросы
```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 1, y: 2 };
let p2 = p1.clone();  // Клонирование
println!("{:?}", p1); // Debug вывод
assert_eq!(p1, p2);   // Сравнение
```

### Доступные derive трейты
```rust
#[derive(
    Debug,          // {:?} форматирование
    Clone,          // .clone() метод
    Copy,           // Копирование вместо перемещения
    PartialEq,      // == и != операторы
    Eq,             // Полное равенство
    PartialOrd,     // <, >, <=, >= операторы
    Ord,            // Полное упорядочивание
    Hash,           // Хеширование для HashMap
    Default,        // Default::default()
)]
struct Point {
    x: i32,
    y: i32,
}
```

## 🏗️ Обобщённые структуры

### Структуры с типами-параметрами
```rust
struct Point<T> {
    x: T,
    y: T,
}

let integer_point = Point { x: 5, y: 10 };
let float_point = Point { x: 1.0, y: 4.0 };
```

### Множественные типы-параметры
```rust
struct Rectangle<T, U> {
    width: T,
    height: U,
}

let rect = Rectangle { 
    width: 30.0,     // f64
    height: 50       // i32
};
```

## 🌟 Продвинутые примеры

### Вложенные структуры
```rust
struct Address {
    street: String,
    city: String,
    country: String,
}

struct Person {
    name: String,
    age: u32,
    address: Address,
}

let person = Person {
    name: String::from("Alice"),
    age: 30,
    address: Address {
        street: String::from("123 Main St"),
        city: String::from("Anytown"),
        country: String::from("USA"),
    },
};

// Доступ к вложенным полям
println!("Lives in: {}", person.address.city);
```

### Структуры с функциями
```rust
struct Calculator {
    result: f64,
}

impl Calculator {
    fn new() -> Self {
        Calculator { result: 0.0 }
    }
    
    fn add(&mut self, value: f64) -> &mut Self {
        self.result += value;
        self  // Возвращаем self для цепочки вызовов
    }
    
    fn multiply(&mut self, value: f64) -> &mut Self {
        self.result *= value;
        self
    }
    
    fn get_result(&self) -> f64 {
        self.result
    }
}

// Использование
let mut calc = Calculator::new();
let result = calc.add(5.0).multiply(2.0).get_result();
println!("Result: {}", result); // 10.0
```

### Структуры с временами жизни
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

## 💡 Лучшие практики

### 1. Используйте осмысленные имена
```rust
// ❌ Плохо
struct Data {
    a: i32,
    b: String,
    c: bool,
}

// ✅ Хорошо
struct UserAccount {
    user_id: i32,
    username: String,
    is_active: bool,
}
```

### 2. Группируйте связанные данные
```rust
// ❌ Много отдельных переменных
let user_name = String::from("Alice");
let user_email = String::from("alice@example.com");
let user_age = 30;

// ✅ Структура
struct User {
    name: String,
    email: String,
    age: u32,
}

let user = User {
    name: String::from("Alice"),
    email: String::from("alice@example.com"),
    age: 30,
};
```

### 3. Используйте derive для стандартных трейтов
```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

### 4. Рассмотрите Copy для простых структур
```rust
#[derive(Debug, Clone, Copy, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

// Теперь Point копируется автоматически
let p1 = Point { x: 1, y: 2 };
let p2 = p1;  // Копирование, не перемещение
println!("{:?} {:?}", p1, p2);  // Оба доступны
```

## 🔗 Связанные темы
- [[Struct-Methods]] - Методы структур
- [[Impl-Blocks]] - Блоки реализации
- [[Struct-Fields]] - Работа с полями
- [[Tuple-Structs]] - Кортежные структуры

#structs #definition #data-types #custom-types #beginner