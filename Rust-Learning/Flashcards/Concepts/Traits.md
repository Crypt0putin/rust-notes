# 🎭 Флеш-карточки: Трейты

## Определение трейта

Как определить трейт `Drawable` с методом `draw`?
?
```rust
trait Drawable {
    fn draw(&self);
}
```

#flashcard #traits #syntax #beginner

---

## Реализация трейта

Как реализовать трейт `Display` для структуры `Point`?
?
```rust
impl Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

#flashcard #traits #implementation #beginner

---

## Trait bounds

Как написать функцию, принимающую любой тип, реализующий `Debug` и `Clone`?
?
```rust
fn process<T: Debug + Clone>(item: T) {
    println!("{:?}", item);
    let copy = item.clone();
}
```

#flashcard #traits #generics #intermediate

---

## Where clause

Как переписать сложные trait bounds используя `where`?
?
```rust
// Вместо:
fn func<T: Display + Clone, U: Debug + PartialEq>(t: T, u: U) { }

// Лучше:
fn func<T, U>(t: T, u: U) 
where 
    T: Display + Clone,
    U: Debug + PartialEq,
{ }
```

#flashcard #traits #where-clause #intermediate

---

## Trait objects

Как создать trait object для трейта `Draw`?
?
```rust
let drawable: Box<dyn Draw> = Box::new(Circle { radius: 5 });
// или
let drawables: Vec<Box<dyn Draw>> = vec![
    Box::new(Circle { radius: 5 }),
    Box::new(Rectangle { width: 10, height: 5 }),
];
```

#flashcard #traits #trait-objects #advanced

---

## Associated types

В чём разница между associated types и generic параметрами?
?
- **Associated types**: одна реализация на тип, естественная связь
- **Generic параметры**: множественные реализации возможны

```rust
// Associated type
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

// Generic parameter  
trait Convert<T> {
    fn convert(&self) -> T;
}
```

#flashcard #traits #associated-types #advanced

---

## Derive макрос

Какие трейты можно автоматически реализовать с помощью `#[derive]`?
?
Основные: `Debug`, `Clone`, `Copy`, `PartialEq`, `Eq`, `PartialOrd`, `Ord`, `Hash`, `Default`

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point { x: i32, y: i32 }
```

#flashcard #traits #derive #beginner

---

## Orphan rule

Можно ли реализовать внешний трейт для внешнего типа?
?
Нет, нельзя. Можно реализовать:
- Свой трейт для любого типа
- Любой трейт для своего типа
- Но НЕ внешний трейт для внешнего типа

Решение: newtype pattern
```rust
struct MyVec(Vec<i32>);
impl Display for MyVec { ... }
```

#flashcard #traits #orphan-rule #advanced

---

## Default methods

Как определить метод по умолчанию в трейте?
?
```rust
trait Greet {
    fn name(&self) -> &str;
    
    // Метод по умолчанию
    fn greet(&self) -> String {
        format!("Hello, {}!", self.name())
    }
}
```

#flashcard #traits #default-methods #intermediate

---

## Conditional implementations

Как реализовать трейт только для типов с определёнными свойствами?
?
```rust
impl<T: Display> ToString for T {
    fn to_string(&self) -> String {
        format!("{}", self)
    }
}
```
Трейт `ToString` реализуется для всех типов, реализующих `Display`.

#flashcard #traits #conditional-impl #advanced