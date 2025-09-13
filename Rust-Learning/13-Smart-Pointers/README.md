# 📦 Умные указатели (Smart Pointers)

Умные указатели обеспечивают дополнительную функциональность поверх обычных ссылок, включая управление памятью и подсчёт ссылок.

## 📋 Содержание

### Основные умные указатели
- [[Box-Pointer]] - Box<T> для кучи
- [[Rc-Pointer]] - Rc<T> для множественного владения
- [[Arc-Pointer]] - Arc<T> для многопоточности
- [[RefCell-Cell]] - RefCell<T> и Cell<T> для interior mutability

### Продвинутые техники
- [[Weak-References]] - Weak<T> для разрыва циклов
- [[Custom-Smart-Pointers]] - Создание собственных умных указателей
- [[Drop-Trait]] - Освобождение ресурсов

## 📦 Box<T>

### Размещение данных в куче
```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
} // b освобождается здесь

// Рекурсивные типы
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

## 🔄 Rc<T> - Reference Counted

### Множественное владение
```rust
use std::rc::Rc;

enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

## 🔒 RefCell<T> - Interior Mutability

### Изменение неизменяемых данных
```rust
use std::cell::RefCell;

fn main() {
    let value = RefCell::new(5);
    
    *value.borrow_mut() += 10;
    
    println!("value: {:?}", value);
}

// Комбинирование Rc и RefCell
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let value = Rc::new(RefCell::new(5));
    
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));
    
    *value.borrow_mut() += 10;
    
    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

## ⚡ Arc<T> - Atomic Reference Counted

### Многопоточное владение
```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            // Используем counter (только для чтения)
            println!("Counter: {}", counter);
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
}
```

## ✅ Чек-лист изучения
- [ ] Понимаю когда использовать Box<T>
- [ ] Работаю с Rc<T> для множественного владения
- [ ] Использую RefCell<T> для interior mutability
- [ ] Применяю Arc<T> в многопоточном коде

#smart-pointers #memory-management #heap #reference-counting #advanced