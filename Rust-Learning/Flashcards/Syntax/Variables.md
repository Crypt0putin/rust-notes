# 🔤 Флеш-карточки: Синтаксис переменных

## Неизменяемая переменная

Как объявить неизменяемую переменную `x` со значением 5?
?
```rust
let x = 5;
```

#flashcard #syntax #variables #beginner

---

## Изменяемая переменная

Как объявить изменяемую переменную `count` со значением 0?
?
```rust
let mut count = 0;
```

#flashcard #syntax #variables #beginner

---

## Константа

Как объявить константу `MAX_POINTS` со значением 100_000?
?
```rust
const MAX_POINTS: u32 = 100_000;
```

#flashcard #syntax #constants #beginner

---

## Затенение переменных

Что происходит в этом коде?
```rust
let x = 5;
let x = x + 1;
let x = x * 2;
```
?
Происходит затенение (shadowing). Каждое новое `let` создаёт новую переменную с тем же именем. Финальное значение `x` будет 12.

#flashcard #concept #shadowing #intermediate

---

## Тип переменной

Как явно указать тип переменной при объявлении?
?
```rust
let x: i32 = 5;
let name: String = String::from("Alice");
```

#flashcard #syntax #types #beginner

---

## Множественное присваивание

Как объявить несколько переменных одновременно?
?
```rust
let (x, y, z) = (1, 2, 3);
```
Это называется деструктуризация кортежа.

#flashcard #syntax #destructuring #intermediate

---

## Неиспользуемая переменная

Как избежать предупреждения компилятора о неиспользуемой переменной?
?
Начните имя переменной с подчёркивания:
```rust
let _unused_variable = 42;
```

#flashcard #syntax #compiler-warnings #beginner

---

## Область видимости

В чём разница между этими двумя блоками?
```rust
// Блок 1
{
    let x = 5;
}
println!("{}", x); // Ошибка!

// Блок 2  
let x = 5;
{
    println!("{}", x); // OK
}
```
?
В блоке 1 переменная `x` объявлена внутри области видимости и недоступна снаружи. В блоке 2 переменная доступна внутри вложенной области видимости.

#flashcard #concept #scope #beginner