# 🧵 Многопоточность в Rust

Rust обеспечивает безопасную многопоточность на уровне системы типов, предотвращая data races во время компиляции.

## 📋 Содержание

### Основы многопоточности
- [[Threads-Basics]] - Создание и управление потоками
- [[Thread-Safety]] - Безопасность потоков в Rust
- [[Send-Sync-Traits]] - Трейты Send и Sync
- [[Thread-Communication]] - Обмен данными между потоками

### Примитивы синхронизации
- [[Channels]] - Каналы для передачи сообщений
- [[Mutex-RwLock]] - Мьютексы и блокировки чтения-записи
- [[Arc-Rc]] - Атомарные и обычные умные указатели
- [[Atomic-Types]] - Атомарные типы

### Продвинутые темы
- [[Thread-Pools]] - Пулы потоков
- [[Async-Programming]] - Асинхронное программирование
- [[Lock-Free-Programming]] - Программирование без блокировок
- [[Parallel-Iterators]] - Параллельные итераторы с Rayon

## 🚀 Создание потоков

### Базовое создание потоков
```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap(); // Ждём завершения потока
}
```

### Передача данных в поток
```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
    // v больше недоступен в main потоке
}
```

### Возврат данных из потока
```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        // Вычисляем что-то в фоновом потоке
        let mut sum = 0;
        for i in 1..=1000 {
            sum += i;
        }
        sum // Возвращаем результат
    });

    let result = handle.join().unwrap();
    println!("Sum: {}", result); // 500500
}
```

## 📨 Каналы (Channels)

### Базовое использование
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        // val больше недоступен
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### Множественные сообщения
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

### Множественные производители
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx1 = tx.clone(); // Клонируем отправитель

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

## 🔒 Мьютексы (Mutex)

### Простой мьютекс
```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    } // Блокировка автоматически снимается

    println!("m = {:?}", m);
}
```

### Разделение мьютекса между потоками
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap()); // 10
}
```

### RwLock для чтения-записи
```rust
use std::sync::{Arc, RwLock};
use std::thread;

fn main() {
    let data = Arc::new(RwLock::new(vec![1, 2, 3]));
    let mut handles = vec![];

    // Несколько читающих потоков
    for i in 0..5 {
        let data = Arc::clone(&data);
        let handle = thread::spawn(move || {
            let data = data.read().unwrap();
            println!("Reader {}: {:?}", i, *data);
        });
        handles.push(handle);
    }

    // Один пишущий поток
    let data_write = Arc::clone(&data);
    let write_handle = thread::spawn(move || {
        let mut data = data_write.write().unwrap();
        data.push(4);
        println!("Writer: Added 4");
    });
    handles.push(write_handle);

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final data: {:?}", *data.read().unwrap());
}
```

## ⚡ Атомарные типы

### Атомарные операции
```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::Relaxed);
            }
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final count: {}", counter.load(Ordering::Relaxed)); // 10000
}
```

### Compare-and-swap
```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let value = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];

    for i in 0..10 {
        let value = Arc::clone(&value);
        let handle = thread::spawn(move || {
            // Пытаемся установить значение равное номеру потока
            loop {
                let current = value.load(Ordering::Relaxed);
                if current == 0 {
                    match value.compare_exchange_weak(
                        current, 
                        i, 
                        Ordering::Relaxed, 
                        Ordering::Relaxed
                    ) {
                        Ok(_) => {
                            println!("Thread {} set the value!", i);
                            break;
                        }
                        Err(_) => continue,
                    }
                } else {
                    break;
                }
            }
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final value: {}", value.load(Ordering::Relaxed));
}
```

## 🎯 Практические паттерны

### Producer-Consumer
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    // Producer
    let producer = thread::spawn(move || {
        for i in 0..10 {
            println!("Producing item {}", i);
            tx.send(i).unwrap();
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    // Consumer
    let consumer = thread::spawn(move || {
        for item in rx {
            println!("Consuming item {}", item);
            thread::sleep(Duration::from_millis(150));
        }
    });
    
    producer.join().unwrap();
    consumer.join().unwrap();
}
```

### Worker Pool
```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;
use std::time::Duration;

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {} got a job; executing.", id);
            job();
        });
        
        Worker { id, thread }
    }
}

fn main() {
    let pool = ThreadPool::new(4);
    
    for i in 0..8 {
        pool.execute(move || {
            println!("Job {} is running", i);
            thread::sleep(Duration::from_secs(1));
        });
    }
    
    thread::sleep(Duration::from_secs(5));
}
```

### Parallel computation
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn parallel_sum(numbers: Vec<i32>) -> i32 {
    let numbers = Arc::new(numbers);
    let result = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    let chunk_size = numbers.len() / 4;
    
    for i in 0..4 {
        let numbers = Arc::clone(&numbers);
        let result = Arc::clone(&result);
        
        let handle = thread::spawn(move || {
            let start = i * chunk_size;
            let end = if i == 3 { numbers.len() } else { (i + 1) * chunk_size };
            
            let mut local_sum = 0;
            for j in start..end {
                local_sum += numbers[j];
            }
            
            let mut global_sum = result.lock().unwrap();
            *global_sum += local_sum;
        });
        
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    *result.lock().unwrap()
}

fn main() {
    let numbers: Vec<i32> = (1..=1000).collect();
    let sum = parallel_sum(numbers);
    println!("Sum: {}", sum); // 500500
}
```

## 🚀 Rayon для параллелизма

```rust
// Cargo.toml: rayon = "1.7"
use rayon::prelude::*;

fn main() {
    let numbers: Vec<i32> = (0..1_000_000).collect();
    
    // Последовательная обработка
    let start = std::time::Instant::now();
    let sum1: i32 = numbers.iter().map(|&x| x * x).sum();
    println!("Sequential: {} in {:?}", sum1, start.elapsed());
    
    // Параллельная обработка
    let start = std::time::Instant::now();
    let sum2: i32 = numbers.par_iter().map(|&x| x * x).sum();
    println!("Parallel: {} in {:?}", sum2, start.elapsed());
    
    // Параллельная фильтрация и обработка
    let even_squares: Vec<i32> = numbers
        .par_iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .collect();
    
    println!("Even squares count: {}", even_squares.len());
}
```

## 💡 Лучшие практики

### 1. Избегайте блокировок когда возможно
```rust
// Вместо мьютекса для простых операций:
use std::sync::{Arc, Mutex};

// Лучше атомарные типы:
use std::sync::atomic::{AtomicUsize, Ordering};

let counter = Arc::new(AtomicUsize::new(0));
// counter.fetch_add(1, Ordering::Relaxed); // Быстрее чем Mutex
```

### 2. Используйте каналы для передачи владения
```rust
// Предпочитайте:
let (tx, rx) = mpsc::channel();

// Вместо разделения через Arc<Mutex<T>>
```

### 3. Минимизируйте область действия блокировок
```rust
// Плохо - долгая блокировка:
let data = mutex.lock().unwrap();
expensive_computation();
*data += 1;

// Хорошо - короткая блокировка:
let result = expensive_computation();
let mut data = mutex.lock().unwrap();
*data += result;
```

## 🚨 Частые ошибки

### Deadlock
```rust
// ❌ Deadlock - два мьютекса блокируются в разном порядке
let mutex1 = Arc::new(Mutex::new(0));
let mutex2 = Arc::new(Mutex::new(0));

// Поток 1
let m1 = Arc::clone(&mutex1);
let m2 = Arc::clone(&mutex2);
thread::spawn(move || {
    let _guard1 = m1.lock().unwrap();
    thread::sleep(Duration::from_millis(10));
    let _guard2 = m2.lock().unwrap(); // Deadlock!
});

// Поток 2  
thread::spawn(move || {
    let _guard2 = mutex2.lock().unwrap();
    thread::sleep(Duration::from_millis(10));
    let _guard1 = mutex1.lock().unwrap(); // Deadlock!
});
```

### Race condition с неатомарными операциями
```rust
// ❌ Race condition:
let counter = Arc::new(Mutex::new(0));
// Поток читает, изменяет и записывает - НЕ атомарно!
let value = *counter.lock().unwrap();
let new_value = value + 1;
*counter.lock().unwrap() = new_value;

// ✅ Правильно:
*counter.lock().unwrap() += 1;
```

## 🎯 Упражнения для практики

### Начальный уровень
1. Создайте 5 потоков, каждый выводит числа от 1 до 10
2. Передайте вектор из одного потока в другой
3. Используйте канал для передачи строк между потоками

### Средний уровень
1. Реализуйте параллельный поиск в массиве
2. Создайте thread-safe счётчик с атомарными операциями
3. Реализуйте producer-consumer с ограниченным буфером

### Продвинутый уровень
1. Создайте свой thread pool
2. Реализуйте параллельную сортировку слиянием
3. Напишите lock-free структуру данных

## ✅ Чек-лист изучения
- [ ] Создаю и управляю потоками
- [ ] Использую каналы для обмена данными
- [ ] Работаю с мьютексами и RwLock
- [ ] Применяю атомарные типы
- [ ] Понимаю трейты Send и Sync
- [ ] Избегаю deadlock'ов и race conditions
- [ ] Использую Rayon для параллелизма

## 🔗 Связанные темы
- [[Send-Sync-Traits]] - Трейты для безопасности потоков
- [[Async-Programming]] - Асинхронное программирование
- [[Performance-Optimization]] - Оптимизация многопоточного кода

#concurrency #multithreading #parallelism #thread-safety #advanced