# 📚 Memory: Stack vs Heap

## 🎯 Основные концепции

В Rust понимание Stack и Heap критически важно для понимания ownership.

### Stack (Стек)
- **LIFO** (Last In, First Out)
- **Быстрый** доступ
- **Фиксированный размер** данных
- **Автоматическое** управление памятью
- Хранит: примитивы, указатели, фиксированные структуры

### Heap (Куча)
- **Произвольный** доступ
- **Медленнее** стека
- **Динамический размер** данных
- **Ручное** управление (через ownership)
- Хранит: String, Vec, Box и другие динамические типы

## 📊 Визуализация памяти

```
STACK                           HEAP
┌─────────────┐                ┌─────────────────┐
│ main()      │                │                 │
├─────────────┤                │  String data    │
│ x = 5       │                │  "Hello"        │
├─────────────┤      ptr ──────>─────────────────┤
│ s = String  │<─────           │                 │
│ (ptr,len,cap)                │  Vec data       │
├─────────────┤      ptr ──────>  [1,2,3,4,5]    │
│ v = Vec     │<─────           │─────────────────┤
│ (ptr,len,cap)                │                 │
└─────────────┘                └─────────────────┘
```

## 💾 Что хранится где?

### На стеке
```rust
// Примитивные типы - известный размер
let x: i32 = 5;              // 4 байта на стеке
let y: f64 = 3.14;           // 8 байт на стеке
let flag: bool = true;       // 1 байт на стеке
let c: char = 'z';           // 4 байта на стеке

// Массивы фиксированного размера
let arr: [i32; 5] = [1, 2, 3, 4, 5]; // 20 байт на стеке

// Кортежи
let tup: (i32, f64) = (500, 6.4);    // 12 байт на стеке

// Структуры с фиксированным размером
struct Point {
    x: i32,
    y: i32,
}
let p = Point { x: 10, y: 20 };      // 8 байт на стеке
```

### На куче
```rust
// String - динамический размер
let s = String::from("hello");       // данные на куче, метаданные на стеке

// Vec - динамический массив
let v = vec![1, 2, 3, 4, 5];        // данные на куче, метаданные на стеке

// Box - явное размещение на куче
let b = Box::new(5);                 // 5 на куче, указатель на стеке

// HashMap и другие коллекции
use std::collections::HashMap;
let mut map = HashMap::new();        // данные на куче
map.insert("key", "value");
```

## 🔄 Copy vs Move семантика

### Copy типы (стек)
```rust
let x = 5;
let y = x;  // Копирование
println!("x = {}, y = {}", x, y); // Оба валидны

// Все целочисленные типы
let a: i32 = 42;
let b = a;  // Copy

// Булевы значения
let flag1 = true;
let flag2 = flag1;  // Copy

// Символы
let c1 = 'A';
let c2 = c1;  // Copy

// Кортежи из Copy типов
let tup1 = (1, 2.0, true);
let tup2 = tup1;  // Copy
```

### Move типы (куча)
```rust
let s1 = String::from("hello");
let s2 = s1;  // Move - s1 больше не валидна
// println!("{}", s1); // ОШИБКА!

let v1 = vec![1, 2, 3];
let v2 = v1;  // Move - v1 больше не валидна

// Для копирования используем clone()
let s3 = String::from("world");
let s4 = s3.clone();  // Глубокое копирование
println!("s3 = {}, s4 = {}", s3, s4); // Оба валидны
```

## 📈 Производительность

### Сравнение операций
| Операция | Stack | Heap |
|----------|-------|------|
| Выделение | O(1) - просто сдвиг указателя | O(n) - поиск свободного места |
| Освобождение | O(1) - автоматическое | O(n) - может требовать дефрагментации |
| Доступ | Прямой, быстрый | Через указатель, медленнее |
| Кэш-эффективность | Высокая | Низкая |

### Оптимизации
```rust
// ❌ Неэффективно - много аллокаций
fn inefficient() {
    for i in 0..1000 {
        let s = String::from("temp"); // Аллокация на каждой итерации
        // использование s
    }
}

// ✅ Эффективно - переиспользование
fn efficient() {
    let mut s = String::with_capacity(100); // Одна аллокация
    for i in 0..1000 {
        s.clear();
        s.push_str("temp");
        // использование s
    }
}
```

## 🎯 Small String Optimization (SSO)

Некоторые типы используют оптимизации для хранения маленьких значений на стеке:

```rust
// SmallVec - вектор с буфером на стеке
use smallvec::SmallVec;
let small: SmallVec<[i32; 4]> = smallvec![1, 2, 3]; // На стеке
let large: SmallVec<[i32; 4]> = smallvec![1, 2, 3, 4, 5]; // Переходит на кучу

// Cow - Copy on Write
use std::borrow::Cow;
fn process(data: Cow<str>) {
    // Избегаем аллокации если не нужно изменение
}
```

## 🔗 Связанные концепции

- [[01_Core/02_Ownership/01_Ownership_Rules|Ownership Rules]] - правила владения
- [[01_Core/02_Ownership/03_Move_Semantics|Move Semantics]] - перемещение данных
- [[01_Core/02_Ownership/04_Copy_Clone|Copy и Clone]] - копирование
- [[02_Advanced/03_Smart_Pointers/01_Box|Box]] - явное размещение на куче

## 💻 Практические примеры

### Пример 1: Анализ размещения
```rust
use std::mem;

fn analyze_memory() {
    // Stack
    let x = 42;
    println!("Size of i32: {} bytes", mem::size_of_val(&x));
    println!("Address of x: {:p}", &x);
    
    // Heap
    let s = String::from("Hello, Rust!");
    println!("Size of String struct: {} bytes", mem::size_of_val(&s));
    println!("String capacity: {} bytes", s.capacity());
    println!("String pointer: {:p}", s.as_ptr());
    
    // Box
    let b = Box::new(100);
    println!("Size of Box<i32>: {} bytes", mem::size_of_val(&b));
    println!("Box points to: {:p}", b.as_ref());
}
```

### Пример 2: Оптимизация аллокаций
```rust
fn string_builder() -> String {
    // Предварительное выделение памяти
    let mut result = String::with_capacity(100);
    
    for i in 0..10 {
        result.push_str(&format!("Item {}, ", i));
    }
    
    result
}

fn vector_operations() {
    // Резервирование места
    let mut vec = Vec::with_capacity(1000);
    
    for i in 0..1000 {
        vec.push(i); // Без реаллокаций
    }
    
    // Проверка использования памяти
    println!("Length: {}, Capacity: {}", vec.len(), vec.capacity());
}
```

### Пример 3: Профилирование памяти
```rust
#[derive(Debug)]
struct MemoryStats {
    stack_size: usize,
    heap_allocations: usize,
}

fn measure_memory<T>(value: &T) -> MemoryStats {
    MemoryStats {
        stack_size: std::mem::size_of_val(value),
        heap_allocations: 0, // Требует специальных инструментов
    }
}
```

## 🎯 Флеш-карточки

#flashcard 
Q: Где хранится String в памяти?
A: Метаданные (ptr, len, capacity) на стеке, сами данные на куче
<!--SR:!2024-01-28,5,280-->

#flashcard 
Q: Какие типы реализуют Copy trait?
A: Все примитивные типы (числа, bool, char) и кортежи из Copy типов
<!--SR:!2024-01-29,4,265-->

#flashcard 
Q: Что быстрее - выделение на стеке или куче?
A: Стек значительно быстрее (O(1) vs O(n))
<!--SR:!2024-01-30,3,250-->

## ⚠️ Частые проблемы

### Проблема 1: Stack Overflow
```rust
// ❌ Слишком большой массив на стеке
fn stack_overflow() {
    let huge_array = [0u8; 10_000_000]; // Stack overflow!
}

// ✅ Используем кучу
fn use_heap() {
    let huge_vec = vec![0u8; 10_000_000]; // На куче
    // или
    let huge_box = Box::new([0u8; 10_000_000]); // На куче
}
```

### Проблема 2: Фрагментация памяти
```rust
// ❌ Много мелких аллокаций
fn fragmented() {
    let mut strings = Vec::new();
    for i in 0..10000 {
        strings.push(format!("{}", i)); // Каждая строка - отдельная аллокация
    }
}

// ✅ Оптимизированная версия
fn optimized() {
    let mut buffer = String::with_capacity(50000);
    let mut positions = Vec::new();
    
    for i in 0..10000 {
        let start = buffer.len();
        buffer.push_str(&i.to_string());
        positions.push(start);
    }
}
```

## 📝 Упражнения

1. **Анализ памяти**:
   - Создайте функцию, показывающую размер различных типов
   - Покажите адреса на стеке и куче

2. **Оптимизация String**:
   - Сравните производительность String::new() vs String::with_capacity()
   - Измерьте количество реаллокаций

3. **Custom allocator**:
   - Реализуйте простой stack allocator
   - Сравните с heap allocation

## 📚 Дополнительные ресурсы

- [Rust Book - Stack and Heap](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap)
- [[01_Core/02_Ownership/00_Index|Ownership Index]]
- [[02_Advanced/06_Performance/03_Memory_Layout|Memory Layout]]

---
#rust #memory #stack #heap #ownership #performance
